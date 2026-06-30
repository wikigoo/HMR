---

# Agentflow Operational Audit — Initial Report

---

## 1. EVALUATION OVERVIEW

**Scope:** All seven audit categories as defined in the audit methodology.

**Inputs received:** None. No `<flow_export>`, `<node_configs>`, `<knowledge_bases>`, `<vps_specs>`, `<metrics_logs>`, `<dependency_versions>`, `<backup_config>`, `<integration_details>`, or `<security_config>` artifacts were supplied.

**Inputs missing:** All nine input categories.

**Consequence:** Per the CORE RULE, no findings can be stated. This report delivers the complete evidence-collection methodology — the exact artifacts and commands required to enable a full audit pass. Resubmit with those artifacts to receive findings, severities, and ranked recommendations.

---

## 2. FINDINGS BY CATEGORY

---

### Category 1 — Nodes & Workflow Logic

**INSUFFICIENT DATA — required:**

Export the Agentflow JSON from the Flowise UI:

```
Flowise UI → Agentflow → top-right menu → Export
```

This produces a `.json` file containing the full node graph (`nodes[]`, `edges[]`, and per-node `inputs`/`outputs`). Supply this as `<flow_export>`.

Additionally supply `<node_configs>`: for each node, the rendered parameter values (system prompts, model names, temperature, max tokens, tool lists, memory type, condition logic). These are visible in the node's side-panel; screenshot or copy-paste each.

**What will be evaluated once received:**
- Dead-end or disconnected nodes with no outgoing edges
- Condition nodes whose branches are never reachable
- Redundant LLM calls that could be collapsed
- Missing Start or Direct Reply termination nodes
- Loop nodes without a defined exit condition (infinite-loop risk)
- `$flow.state` usage correctness across shared memory [1](#0-0) 

---

### Category 2 — Knowledge Bases

**INSUFFICIENT DATA — required:**

From the Flowise UI → Document Stores, export or screenshot for each KB:

1. Store name and which sub-agent/node it is attached to
2. Document loader type (PDF, Web, CSV, etc.)
3. Text splitter type, `chunk_size`, `chunk_overlap`
4. Embedding model name and dimension count
5. Vector store type (Chroma, Pinecone, Qdrant, etc.) and index name
6. Last upsert date (visible in Upsert History: `GET /api/v1/upsert-history`)
7. Record Manager presence (yes/no)

Supply this as `<knowledge_bases>`.

**What will be evaluated once received:**
- KB attached to wrong agent (routing mismatch)
- Embedding model / vector store dimension mismatch (causes upsert errors)
- Stale KBs (last-updated date older than content refresh cycle)
- Missing Record Manager (causes duplicate vectors on re-upsert)
- `top_k` values that are too low (default 4) for complex multi-document queries [2](#0-1) [3](#0-2) 

---

### Category 3 — System & Dependency Health

**INSUFFICIENT DATA — required:**

Run the following on the Hetzner VPS and supply output as `<vps_specs>` and `<dependency_versions>`:

```bash
# VPS specs
nproc && free -h && df -h

# OS and kernel
uname -a && cat /etc/os-release

# Docker runtime
docker version && docker compose version

# Flowise container status and image tag
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

# Flowise version (from running container)
docker exec <flowise-container-name> npx flowise --version 2>/dev/null || \
  docker exec <flowise-container-name> cat /app/package.json | grep '"version"'

# Node.js version inside container
docker exec <flowise-container-name> node --version

# Current MODE (main vs queue)
docker exec <flowise-container-name> printenv MODE

# Database type in use
docker exec <flowise-container-name> printenv DATABASE_TYPE
```

For Flutter app dependency versions, supply `pubspec.lock` content.

**What will be evaluated once received:**
- Whether `MODE=queue` with Redis is configured (Flowise docs recommend this for production)
- Whether PostgreSQL is used instead of SQLite (SQLite is not recommended at scale)
- Whether VPS RAM/CPU meets the production minimum (≥1 vCPU / 2 GB RAM per server instance)
- Whether Flowise is pinned to a specific image tag or using `latest` (unpinned = uncontrolled upgrades) [4](#0-3) 

---

### Category 4 — Integrations

**INSUFFICIENT DATA — required:**

Supply `<integration_details>` containing:

1. **Website embed:** The full `Chatbot.init({...})` script block, including `chatflowid`, `apiHost`, `chatflowConfig`, and any `theme` or `observersConfig` values
2. **Flutter app:** The HTTP call implementation — the `url`, `headers` (especially `Authorization`), request body shape, and timeout configuration
3. **CORS config:** Current values of `CORS_ORIGINS` and `IFRAME_ORIGINS` env vars
4. **Chatflow-level API key:** Whether an API key is assigned to the flow (Flowise UI → flow Settings → API Key)

**What will be evaluated once received:**
- Whether `apiHost` points to HTTPS (not plain HTTP) in production
- Whether `Authorization: Bearer <key>` is present in Flutter HTTP calls
- Whether `CORS_ORIGINS` is locked to specific domains or set to `*` (wildcard = security risk)
- Whether `flowise-embed` version is pinned (unpinned CDN loads can break on Flowise v2.1.0+ streaming changes)
- Flutter timeout handling and error-state UI behavior [5](#0-4) [6](#0-5) 

---

### Category 5 — Security

**INSUFFICIENT DATA — required:**

Supply `<security_config>` containing (redact actual secret values — supply only key names and storage method):

```bash
# Show which env vars are set (names only, not values)
docker exec <flowise-container-name> printenv | grep -E \
  'JWT|SECRET|FLOWISE_USERNAME|FLOWISE_PASSWORD|APIKEY|TOKEN|CORS|IFRAME' | cut -d= -f1

# Confirm FLOWISE_SECRETKEY_OVERWRITE is set (name only)
docker exec <flowise-container-name> printenv FLOWISE_SECRETKEY_OVERWRITE | \
  wc -c   # should be > 1 if set

# VPS SSH access method
cat /etc/ssh/sshd_config | grep -E 'PasswordAuthentication|PubkeyAuthentication|PermitRootLogin'

# Open ports
ss -tlnp
```

**What will be evaluated once received:**
- Whether `FLOWISE_SECRETKEY_OVERWRITE` is set (if not, encryption key can be regenerated on restart, causing "Credentials could not be decrypted" failures — a known Flowise failure mode)
- Whether `JWT_AUTH_TOKEN_SECRET` and `JWT_REFRESH_TOKEN_SECRET` are set to strong unique values
- Whether `CORS_ORIGINS` / `IFRAME_ORIGINS` are domain-restricted
- Whether VPS SSH uses key-based auth only (no password auth)
- Whether API keys are stored in DB (`APIKEY_STORAGE_TYPE=db`) vs. a local JSON file (JSON file is lost on container rebuild)
- Whether `ALLOW_UNAUTHORIZED_CERTS=false` in production [7](#0-6) [8](#0-7) 

---

### Category 6 — Backup & Resilience

**INSUFFICIENT DATA — required:**

Supply `<backup_config>` containing:

```bash
# Database type and path
docker exec <flowise-container-name> printenv DATABASE_TYPE
docker exec <flowise-container-name> printenv DATABASE_PATH

# If SQLite: confirm file exists and size
ls -lh ~/.flowise/database.sqlite 2>/dev/null || \
  docker exec <flowise-container-name> ls -lh /root/.flowise/database.sqlite

# If PostgreSQL: last successful pg_dump
ls -lht /path/to/backups/*.sql | head -5

# Blob storage path (uploaded files, KB source docs)
docker exec <flowise-container-name> printenv BLOB_STORAGE_PATH

# Cron jobs for backup
crontab -l

# Hetzner snapshot schedule (from Hetzner Cloud Console or hcloud CLI)
hcloud server list && hcloud snapshot list
```

Also confirm: date of last backup restore test.

**What will be evaluated once received:**
- Whether automated backups exist for the SQLite/PostgreSQL database
- Whether `BLOB_STORAGE_PATH` (uploaded KB source files) is included in backups
- Whether `SECRETKEY_PATH` (encryption key file) is backed up — loss of this file makes all stored credentials permanently unrecoverable
- Whether Hetzner volume snapshots cover all persistent data
- Whether a restore has ever been tested [9](#0-8) [10](#0-9) 

---

### Category 7 — Improvements & Monitoring

**INSUFFICIENT DATA — required:**

Supply `<metrics_logs>` containing:

```bash
# Last 200 lines of Flowise server error log
docker exec <flowise-container-name> tail -200 /root/.flowise/logs/server-error.log

# Last 200 lines of request log
docker exec <flowise-container-name> tail -200 /root/.flowise/logs/server-requests.log.jsonl

# Container resource usage
docker stats --no-stream

# Check if Prometheus metrics endpoint is enabled
curl -s http://localhost:3000/api/v1/metrics | head -20
```

Also confirm: whether any analytics provider (LangFuse, Lunary, Arize, Phoenix, Opik) is currently enabled in the flow's Settings → Configuration → Analyse Chatflow.

**What will be evaluated once received:**
- Error rate and error types from `server-error.log`
- Whether `ENABLE_METRICS=true` and `METRICS_PROVIDER=prometheus` are set (required for Grafana dashboards)
- Whether any LLM-level tracing (LangFuse / Phoenix) is active for per-node latency visibility
- Memory/CPU headroom on the VPS container
- Whether Agentflow V1 (deprecated) nodes are still in use — migration to V2 is recommended [11](#0-10) [12](#0-11) 

---

## 3. PRIORITIZED ACTION LIST

No findings can be ranked until inputs are received. The collection actions are ordered by the risk category they unlock:

| Priority | Action | Risk Category Unlocked |
|---|---|---|
| 1 | Run security env-var audit commands (Category 5) | CRITICAL candidates: unset `FLOWISE_SECRETKEY_OVERWRITE`, open CORS, no SSH key auth |
| 2 | Run backup audit commands (Category 6) | CRITICAL candidates: no DB backup, no encryption key backup |
| 3 | Export flow JSON + node configs (Category 1) | HIGH candidates: broken routing, infinite loops |
| 4 | Run VPS/dependency commands (Category 3) | HIGH candidates: SQLite in production, `MODE=main`, outdated Flowise |
| 5 | Collect KB metadata (Category 2) | MEDIUM candidates: stale KBs, missing Record Manager |
| 6 | Collect integration details (Category 4) | MEDIUM candidates: HTTP not HTTPS, wildcard CORS, unpinned embed |
| 7 | Collect metrics/logs (Category 7) | MEDIUM candidates: no monitoring, no tracing |

---

## 4. CONCLUSION & NEXT STEPS

**No findings can be concluded without evidence.** The audit framework is fully defined. To proceed:

**Top 5 evidence items to gather first** (highest risk-to-effort ratio):

1. **Run the Category 5 security commands** — the `FLOWISE_SECRETKEY_OVERWRITE` gap is a known Flowise production failure mode that causes permanent credential loss on container restart; this must be ruled out first.
2. **Run the Category 6 backup commands** — confirm whether the SQLite file and `SECRETKEY_PATH` are included in any automated backup; these are the two unrecoverable-loss scenarios.
3. **Export the flow JSON** — provides the complete node graph for logic and routing analysis.
4. **Run the Category 3 VPS/dependency commands** — confirms whether the deployment is running in `main` mode (not production-recommended) and whether the database is SQLite (not recommended at scale).
5. **Collect integration details** — confirms whether the Flutter app and website embed are using authenticated HTTPS endpoints.

Resubmit any subset of the above and this audit will produce findings, severities, and ranked runnable recommendations for those categories immediately.

### Citations

**File:** en/using-flowise/agentflowv2.md (L24-24)
```markdown

```

**File:** en/using-flowise/document-stores.md (L159-161)
```markdown
{% hint style="warning" %}
To ensure compatibility between an embedding model and a Vector Store index, dimensional alignment is essential. Both **the embedding model and the vector store index must have the same number of dimensions**. Dimensionality mismatch will result in upsertion errors, as the Vector Store is designed to handle vectors of a specific size determined by the chosen embedding model.
{% endhint %}
```

**File:** en/use-cases/upserting-data.md (L130-134)
```markdown
## 5. Record Manager

The [Record Manager node](../integrations/langchain/record-managers.md) is an optional but incredibly useful addition to our upserting flow. It allows us to maintain records of all the chunks that have been upserted to our Vector Store, enabling us to efficiently add or delete chunks as needed.

For a more in-depth guide, we refer you to [this guide](../integrations/langchain/record-managers.md).
```

**File:** en/configuration/running-in-production.md (L5-14)
```markdown
When running in production, we highly recommend using [Queue](running-flowise-using-queue.md) mode with the following settings:

* 2 main servers with load balancing, each starting from 1 vCPU 2GB RAM
* 4 workers, each starting from 2 vCPU 4GB RAM

You can configure auto scaling depending on the traffic and volume.

## Database

By default, Flowise will use SQLite as the database. However when running at scale, its recommended to use PostgresQL.
```

**File:** en/using-flowise/embed.md (L271-284)
```markdown
## CORS

When using embedded chat widget, there's chance that you might face CORS issue like:

{% hint style="danger" %}
Access to fetch at 'https://\<your-flowise.com>/api/v1/prediction/' from origin 'https://\<your-flowise.com>' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
{% endhint %}

To fix it, specify the following environment variables:

```
CORS_ORIGINS=*
IFRAME_ORIGINS=*
```
```

**File:** en/configuration/authorization/chatflow-level.md (L9-29)
```markdown
After you have a chatflow / agentflow constructed, by default, your flow is available to public. Anyone that has access to the Chatflow ID is able to run prediction through Embed or API.

In cases where you might want to allow certain people to be able to access and interact with it, you can do so by assigning an API key for that specific chatflow.

## API Key

In dashboard, navigate to API Keys section, and you should be able to see a DefaultKey created. You can also add or delete any keys.

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Chatflow

Navigate to the chatflow, and now you can select the API Key you want to use to protect the chatflow.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After assigning an API key, one can only access the chatflow API when the Authorization header is provided with the correct API key specified during a HTTP call.

```json
"Authorization": "Bearer <your-api-key>"
```
```

**File:** en/configuration/environment-variables.md (L50-68)
```markdown
## For Debugging and Logs

| Variable   | Description                         | Type                                             |                                |
| ---------- | ----------------------------------- | ------------------------------------------------ | ------------------------------ |
| DEBUG      | Print logs from components          | Boolean                                          |                                |
| LOG\_PATH  | Location where log files are stored | String                                           | `Flowise/packages/server/logs` |
| LOG\_LEVEL | Different levels of logs            | Enum String: `error`, `info`, `verbose`, `debug` | `info`                         |

`DEBUG`: if set to true, will print logs to terminal/console:

<figure><img src="../.gitbook/assets/image (3) (3) (1).png" alt=""><figcaption></figcaption></figure>

`LOG_LEVEL`: Different log levels for loggers to be saved. Can be `error`, `info`, `verbose`, or `debug.` By default it is set to `info,` only `logger.info` will be saved to the log files. If you want to have complete details, set to `debug`.

<figure><img src="../.gitbook/assets/image (2) (4).png" alt=""><figcaption><p><strong>server-requests.log.jsonl - logs every request sent to Flowise</strong></p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><strong>server.log - logs general actions on Flowise</strong></p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5) (4).png" alt=""><figcaption><p><strong>server-error.log - logs error with stack trace</strong></p></figcaption></figure>
```

**File:** en/configuration/environment-variables.md (L80-97)
```markdown
Flowise store your third party API keys as encrypted credentials using an encryption key.

By default, a random encryption key will be generated when starting up the application and stored under a file path. This encryption key is then retrieved everytime to decrypt the credentials used within a chatflow. For example, your OpenAI API key, Pinecone API key, etc.

You can configure to use AWS Secret Manager to store the encryption key instead.

| Variable                      | Description                                           | Type                        | Default                   |
| ----------------------------- | ----------------------------------------------------- | --------------------------- | ------------------------- |
| SECRETKEY\_STORAGE\_TYPE      | How to store the encryption key                       | Enum String: `local`, `aws` | `local`                   |
| SECRETKEY\_PATH               | Local file path where encryption key is saved         | String                      | `Flowise/packages/server` |
| FLOWISE\_SECRETKEY\_OVERWRITE | Encryption key to be used instead of the existing key | String                      |                           |
| SECRETKEY\_AWS\_ACCESS\_KEY   |                                                       | String                      |                           |
| SECRETKEY\_AWS\_SECRET\_KEY   |                                                       | String                      |                           |
| SECRETKEY\_AWS\_REGION        |                                                       | String                      |                           |

For some reasons, sometimes encryption key might be re-generated or the stored path was changed, this will cause errors like - <mark style="color:red;">Credentials could not be decrypted.</mark>

To avoid this, you can set your own encryption key as `FLOWISE_SECRETKEY_OVERWRITE`, so that the same encryption key will be used everytime. There is no restriction on the format, you can set it as any text that you want, or the same as your `FLOWISE_PASSWORD`.
```

**File:** en/configuration/authorization/app-level.md (L56-61)
```markdown
### Security Best Practices

* Use strong, unique values for `TOKEN_HASH_SECRET` and store them securely
* For production, use `SMTP_SECURE=true` and `ALLOW_UNAUTHORIZED_CERTS=false`
* Set appropriate token expiry times based on your security requirements
* Use higher `PASSWORD_SALT_HASH_ROUNDS` values (12-15) for better security in production
```

**File:** en/configuration/databases.md (L69-106)
```markdown
## Backup

1. Shut down FlowiseAI application.
2. Ensure that the database connection to other applications is turned off.
3. Backup your database.
4. Test backup database.

### SQLite

1. Rename file name.

   Windows:

   ```bash
   rename "DATABASE_PATH\database.sqlite" "DATABASE_PATH\BACKUP_FILE_NAME.sqlite"
   ```

   Linux:

   ```bash
   mv DATABASE_PATH/database.sqlite DATABASE_PATH/BACKUP_FILE_NAME.sqlite
   ```

2. Backup database.

   Windows:

   ```bash
   copy DATABASE_PATH\BACKUP_FILE_NAME.sqlite DATABASE_PATH\database.sqlite
   ```

   Linux:

   ```bash
   cp DATABASE_PATH/BACKUP_FILE_NAME.sqlite DATABASE_PATH/database.sqlite
   ```

3. Test backup database by running Flowise.
```

**File:** en/using-flowise/analytics/README.md (L9-22)
```markdown
Flowise provides step by step tracing for [Agentflow V2](../agentflowv2.md):

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

Besides, there are also several analytic providers Flowise integrates with:

* [LunaryAI](https://lunary.ai/)
* [Langsmith](https://smith.langchain.com/)
* [Langfuse](https://langfuse.com/)
* [LangWatch](https://langwatch.ai/)
* [Arize](https://arize.com/)
* [Phoenix](https://phoenix.arize.com/)
* [Opik](https://www.comet.com/site/products/opik/)

```
