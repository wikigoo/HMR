**Important caveat before proceeding:** I only have access to the `FlowiseAI/FlowiseDocs` repository — the official documentation. I cannot access your Hetzner VPS, your actual workflow JSON, your Flutter app code, your knowledge bases, or any live system metrics. What follows is a **documentation-grounded evaluation framework** — a structured checklist of what to inspect and how, based on FlowiseAI's own documented best practices. You must execute each check against your live system.

---

# Agentflow Evaluation Report

## 1. Overview of the Evaluation Process

This evaluation covers nine domains: node integrity, knowledge base connections, technical health, software currency, backup/resilience, integration audit, security, performance, and monitoring. Each section identifies what to check, the risk if neglected, and the corrective action grounded in FlowiseAI documentation.

---

## 2. Findings by Category

### 2.1 Node Evaluation

Agentflow V2 uses a node-dependency and execution queue system where each node is an independent unit. The node types available are: `Start`, `LLM`, `Agent`, `Tool`, `Retriever`, `HTTP`, `Custom Function`, `Condition`, `Loop`, `Human Input`, and `Direct Reply`. [1](#0-0) 

**What to check:**

| Check | Risk if Skipped | Action |
|---|---|---|
| Every flow has exactly one `Start` node | Flow will not execute | Verify in canvas |
| Every branch terminates at a `Direct Reply` or loops back correctly | Infinite loops, hung sessions | Trace all condition branches |
| `Condition` nodes have all output paths wired | Silent failures on unmatched conditions | Audit each Condition node's "Else" branch |
| `Agent` nodes have a clear, non-ambiguous system prompt | Hallucination, scope creep | Review each system message |
| `Loop` nodes have a defined exit condition | Infinite loops consuming tokens | Confirm loop termination logic |
| No orphaned nodes exist on the canvas | Dead code, confusion | Remove any disconnected nodes |

**Known architectural constraint:** The V1 Multi-Agent system (Supervisor/Worker) only supports one task at a time and one Supervisor per flow. Parallel delegation is not supported and will cause errors. [2](#0-1) 

If your system uses V1 Multi-Agents, evaluate whether migrating to **Agentflow V2** is appropriate — V1 is documented as deprecating. [3](#0-2) 

---

### 2.2 Knowledge Base Connection

Each `Agent` node can connect to knowledge via two mechanisms: **Document Stores** (managed indexing pipeline) and **Vector Embeddings** (direct vector store connection). [4](#0-3) 

**What to check:**

- **Upsert status:** Only upserted Document Stores can be queried. Navigate to each Document Store and confirm status is not "pending" or "error." [5](#0-4) 

- **Knowledge descriptions:** Each Agent node's `Describe Knowledge` field must be accurate and specific. This is the text the LLM uses to decide *when* to query that store. Vague descriptions cause missed retrievals. [6](#0-5) 

- **Embedding/vector store dimension alignment:** The embedding model used during upsert must match the one configured in the Agent node. A mismatch causes upsertion errors. [7](#0-6) 

- **Chunk size and top-K tuning:** Default top-K is 4. If responses are incomplete, increase top-K or chunk overlap. If responses are slow/expensive, reduce them. [8](#0-7) 

- **Record Manager:** If knowledge bases are updated periodically, confirm a Record Manager is configured to prevent duplicate vector embeddings on re-upsert. [9](#0-8) 

- **Retrieval Query test:** Use the built-in "Retrieval Query" button in each Document Store to verify retrieval accuracy before relying on it in production. [10](#0-9) 

---

### 2.3 Technical Health and Performance

**Database:** By default, Flowise uses SQLite. For a production VPS serving a website chatbot and a Flutter app simultaneously, **PostgreSQL is strongly recommended**. [11](#0-10) 

**Queue mode:** For production, Flowise recommends running in **Queue mode** with Redis, using separate main server and worker processes. Without this, all prediction requests are handled synchronously, creating a bottleneck under concurrent load. [12](#0-11) 

Minimum recommended spec for queue mode: 2 main servers (1 vCPU / 2 GB RAM each), 4 workers (2 vCPU / 4 GB RAM each). [12](#0-11) 

**Storage:** If using local file storage (`STORAGE_TYPE=local`), uploaded files and logs are stored on the VPS disk. This is a single point of failure. Consider migrating to S3-compatible storage. [13](#0-12) 

**Rate limiting:** If Flowise is behind a reverse proxy (e.g., Nginx), set `NUMBER_OF_PROXIES` correctly or the rate limiter will treat all requests as coming from the same IP and block legitimate traffic. [14](#0-13) 

---

### 2.4 Update Needs

**Check the installed Flowise version** against the latest release on [npmjs.com/package/flowise](https://www.npmjs.com/package/flowise).

**Critical version note:** Flowise v2.1.0 changed how streaming works. If your embedded chatbot widget (`flowise-embed`) is pinned to a version below v2.1.0 of Flowise, or uses a `web.js` version mismatch, the embedded chatbot will fail to receive streamed messages. [15](#0-14) 

**Agentflow V1 deprecation:** If any flows use the V1 Multi-Agent or Sequential Agent architecture, plan migration to Agentflow V2, which is the current supported engine. [16](#0-15) 

**Node.js version:** Flowise requires Node.js v18.15.0 or v20+. Verify the VPS runtime version. [17](#0-16) 

---

### 2.5 Backup and Resilience

| Asset | Backup Action |
|---|---|
| **SQLite database** (if used) | Schedule daily `cp ~/.flowise/database.sqlite /backup/` or use `pg_dump` for PostgreSQL |
| **Encryption key** | Set `FLOWISE_SECRETKEY_OVERWRITE` to a fixed value and store it in a secrets manager. Without this, a restart can regenerate the key and make all stored credentials unreadable. |
| **Flow JSON exports** | Export each Agentflow as JSON from the UI regularly and commit to a Git repository |
| **Document Store files** | Back up `BLOB_STORAGE_PATH` (default: `~/.flowise/storage`) |
| **Environment variables** | Store `.env` in a secrets manager or encrypted vault, not in plaintext on disk |

The encryption key risk is critical: [18](#0-17) 

---

### 2.6 Integration Audit

**Website chatbot (embedded widget):**

- Confirm `CORS_ORIGINS` and `IFRAME_ORIGINS` are set to your website's domain (not `*` in production, which is overly permissive). [19](#0-18) 

- Confirm the `chatflowid` and `apiHost` in the embed script point to the correct flow and VPS endpoint. [20](#0-19) 

- If the flow is protected by an API key, the embed script must pass the `Authorization: Bearer <key>` header. [21](#0-20) 

**Flutter Android application:**

- The Flutter app must call the Prediction API at `/api/v1/prediction/{chatflowId}` with the correct `Authorization` header if the flow is key-protected. [22](#0-21) 

- For streaming responses (SSE), verify the Flutter HTTP client supports Server-Sent Events. Flowise supports SSE streaming natively. [23](#0-22) 

- Test for timeout handling: multi-agent flows with multiple LLM calls can take 10–60+ seconds. The Flutter client must not time out prematurely.

---

### 2.7 Security Check

| Area | Check | Reference |
|---|---|---|
| **App-level auth** | Confirm JWT auth is enabled (`JWT_AUTH_TOKEN_SECRET`, `JWT_REFRESH_TOKEN_SECRET` are set). The legacy `FLOWISE_USERNAME`/`FLOWISE_PASSWORD` method is deprecated. | [24](#0-23)  |
| **Chatflow-level auth** | Confirm each public-facing flow has an API key assigned | [22](#0-21)  |
| **Credential encryption** | Set `FLOWISE_SECRETKEY_OVERWRITE` to a fixed, strong key | [25](#0-24)  |
| **API key storage** | Set `APIKEY_STORAGE_TYPE=db` to store API keys in the database rather than a local JSON file (safer for multi-instance setups) | [26](#0-25)  |
| **Custom JS sandbox** | If using Custom Function nodes, restrict `TOOL_FUNCTION_BUILTIN_DEP` and `TOOL_FUNCTION_EXTERNAL_DEP` to only what is needed. Avoid `*`. | [27](#0-26)  |
| **VPS access** | Flowise admin UI should not be exposed directly on port 3000 to the internet. Place behind Nginx/Caddy with TLS and restrict admin access by IP. | General VPS hardening |
| **Password hashing** | Use `PASSWORD_SALT_HASH_ROUNDS` of 12–15 in production | [24](#0-23)  |

---

### 2.8 Monitoring Strategy

Flowise has native support for **Prometheus + Grafana** and **OpenTelemetry** for infrastructure-level metrics (API request counts, prediction counts, heap/CPU/RAM). [28](#0-27) 

Enable with:
```properties
ENABLE_METRICS=true
METRICS_PROVIDER=prometheus
METRICS_INCLUDE_NODE_METRICS=true
```

For **node-by-node LLM tracing** (latency per node, token usage, error traces), connect one of the supported analytics providers:

- **Langfuse** — recommended for self-hosted, open-source tracing
- **LangSmith** — strong for LangChain-based flows
- **Lunary** — includes conversation replay and user tracking
- **Phoenix/Arize** — strong for evaluation and drift detection [29](#0-28) 

Flowise also provides two Grafana dashboard templates:
- `grafana.dashboard.app.json.txt` — API-level metrics
- `grafana.dashboard.server.json.txt` — Node.js heap, CPU, RAM [30](#0-29) 

---

## 3. Specific Improvement Recommendations

### Priority 1 — Critical (address immediately)

1. **Set `FLOWISE_SECRETKEY_OVERWRITE`** to a fixed value. Without it, a VPS restart or update can regenerate the encryption key and permanently break all stored credentials (OpenAI keys, vector DB keys, etc.). [18](#0-17) 

2. **Switch to PostgreSQL** if currently on SQLite. SQLite has no concurrent write support and will degrade under simultaneous website + Flutter app load. [11](#0-10) 

3. **Enable Queue mode with Redis** to handle concurrent prediction requests without blocking. [31](#0-30) 

### Priority 2 — High (address within days)

4. **Verify flowise-embed version compatibility** with your installed Flowise version. Pin `web.js` to a specific version in your website embed script. [32](#0-31) 

5. **Restrict `CORS_ORIGINS`** to your specific website domain. Do not use `*` in production. [33](#0-32) 

6. **Assign API keys to all public-facing flows** and ensure the Flutter app and website embed pass them correctly. [34](#0-33) 

7. **Enable an analytics provider** (Langfuse recommended for self-hosted) to gain per-node latency and error visibility. [35](#0-34) 

### Priority 3 — Medium (address within weeks)

8. **Audit all knowledge base descriptions** in Agent nodes. Use the "Generate" button to auto-generate optimized descriptions if they are currently vague. [36](#0-35) 

9. **Add Record Managers** to all Document Stores that are updated periodically to prevent duplicate embeddings. [9](#0-8) 

10. **Export all flow JSONs** to a Git repository as a backup and version control mechanism. Do this after every significant workflow change.

11. **Set up Prometheus + Grafana** on the VPS for infrastructure-level alerting (CPU, RAM, request rate). [37](#0-36) 

12. **Migrate V1 flows to Agentflow V2** if any exist, as V1 is being deprecated. [3](#0-2) 

---

## 4. Conclusion and Prioritized Next Steps

```
IMMEDIATE (this week)
├── 1. Set FLOWISE_SECRETKEY_OVERWRITE (prevents credential loss on restart)
├── 2. Migrate database to PostgreSQL
└── 3. Enable Queue mode + Redis

SHORT-TERM (next 2 weeks)
├── 4. Pin flowise-embed web.js version; verify streaming compatibility
├── 5. Restrict CORS_ORIGINS to specific domains
├── 6. Assign API keys to all flows; verify Flutter + website pass them
└── 7. Connect Langfuse or equivalent analytics provider

MEDIUM-TERM (next month)
├── 8. Audit and regenerate all knowledge base descriptions
├── 9. Add Record Managers to periodically-updated Document Stores
├── 10. Export all flows to Git (establish backup routine)
├── 11. Deploy Prometheus + Grafana dashboards
└── 12. Plan V1 → V2 migration if applicable
```

The most operationally dangerous gap is the encryption key configuration — a VPS reboot without `FLOWISE_SECRETKEY_OVERWRITE` set can silently invalidate all stored API credentials, taking the entire system offline. Address that first, before anything else.

### Citations

**File:** en/using-flowise/agentflowv2.md (L15-21)
```markdown
## Core Concept

AgentFlow V2 represents a significant architectural evolution, introducing a new paradigm in Flowise that focuses on explicit workflow orchestration and enhanced flexibility. Unlike V1's primary reliance on external frameworks for its core agent graph logic, V2 shifts the focus towards designing the entire workflow using a granular set of specialized, standalone nodes developed natively as core Flowise components.

In this V2 architecture, each node functions as an independent unit, executing a discrete operation based on its specific design and configuration. The visual connections between nodes on the canvas explicitly define the workflow's path and control sequence, data can be passed between nodes by referencing the outputs of any previously executed node in the current flow, and the Flow State provides an explicit mechanism for managing and sharing data throughout the workflow.

V2 architecture implements a comprehensive node-dependency and execution queue system that precisely respects these defined pathways while maintaining clear separation between components, allowing workflows to become both more sophisticated and easier to design. This allow complex patterns like loops, conditional branching, human-in-the-loop interactions and others to be achievable. This makes it more adaptable to diverse use cases while remaining more maintainable and extensible.
```

**File:** en/using-flowise/agentflowv2.md (L53-57)
```markdown
### ⚡ Streaming

Supports Server-Sent Events (SSE) for real-time streaming of LLM or agent responses. Streaming also enables subscription to execution updates as the workflow progresses.

<figure><img src="../.gitbook/assets/longGIF.gif" alt=""><figcaption></figcaption></figure>
```

**File:** en/using-flowise/agentflowv2.md (L123-131)
```markdown
  * **Knowledge / Document Stores**: Configure access to information within Flowise-managed Document Stores.
    * **Document Store**: Choose a pre-configured Document Store from which the agent can retrieve information. These stores must be set up and populated in advance.
    * **Describe Knowledge**: Provide a natural language description of the content and purpose of this Document Store. This description guides the agent in understanding what kind of information the store contains and when it would be appropriate to query it.
  * **Knowledge / Vector Embeddings**: Configure access to external, pre-existing vector stores as additional knowledge sources for the agent.
    * **Vector Store**: Selects the specific, pre-configured vector database the agent can query.
    * **Embedding Model**: Specifies the embedding model associated with the selected vector store, ensuring compatibility for queries.
    * **Knowledge Name**: Assigns a short, descriptive name to this vector-based knowledge source, which the agent can use for reference.
    * **Describe Knowledge**: Provide a natural language description of the content and purpose of this vector store, guiding the agent on when and how to utilize this specific knowledge source.
    * **Return Source Documents**: If enabled, instructs the agent to include source document information with the data retrieved from the vector store.
```

**File:** esp/usar-flowise/agentflows/multi-agents.md (L29-36)
```markdown
Para mantener el orden y la simplicidad, este sistema multi-agente opera bajo dos restricciones importantes:

* **Una tarea a la vez:** El Supervisor está intencionalmente diseñado para enfocarse en una sola tarea a la vez. Espera a que el Worker activo complete su tarea y devuelva los resultados antes de analizar el siguiente paso y delegar la tarea subsiguiente. Esto asegura que cada paso se complete exitosamente antes de continuar, previniendo la sobrecomplejidad.
* **Un Supervisor por flujo:** Si bien es teóricamente posible implementar un conjunto de sistemas multi-agente anidados para formar una estructura jerárquica más sofisticada para flujos de trabajo altamente complejos, lo que LangChain define como "[Hierarchical Agent Teams](https://github.com/langchain-ai/langgraph/blob/main/examples/multi_agent/hierarchical_agent_teams.ipynb)", con un supervisor de nivel superior y supervisores de nivel medio gestionando equipos de workers, los sistemas multi-agente de Flowise actualmente operan con un solo Supervisor.

{% hint style="info" %}
Estas dos restricciones son importantes cuando **planificas el flujo de trabajo de tu aplicación**. Si intentas diseñar un flujo de trabajo donde el Supervisor necesita delegar múltiples tareas simultáneamente, en paralelo, el sistema no podrá manejarlo y encontrarás un error.
{% endhint %}
```

**File:** en/SUMMARY.md (L25-28)
```markdown
  * [Agentflow V1 (Deprecating)](using-flowise/agentflowv1/README.md)
    * [Multi-Agents](using-flowise/agentflowv1/multi-agents.md)
    * [Sequential Agents](using-flowise/agentflowv1/sequential-agents/README.md)
      * [Video Tutorials](using-flowise/agentflowv1/sequential-agents/video-tutorials.md)
```

**File:** en/tutorials/rag.md (L30-32)
```markdown
{% hint style="success" %}
Only upserted document store can be used
{% endhint %}
```

**File:** en/using-flowise/document-stores.md (L159-161)
```markdown
{% hint style="warning" %}
To ensure compatibility between an embedding model and a Vector Store index, dimensional alignment is essential. Both **the embedding model and the vector store index must have the same number of dimensions**. Dimensionality mismatch will result in upsertion errors, as the Vector Store is designed to handle vectors of a specific size determined by the chosen embedding model.
{% endhint %}
```

**File:** en/using-flowise/document-stores.md (L181-185)
```markdown
Record Manager is an optional but incredibly useful addition to our upserting flow. It allows us to maintain records of all the chunks that have been upserted to our Vector Store, enabling us to efficiently add or delete chunks as needed.&#x20;

In other words, any changes to your documents during a new upsert will not result in duplicate vector embeddings being stored in the vector store.

Detailed instructions on how to set up and utilize this feature can be found in the dedicated [guide](../integrations/langchain/record-managers.md).
```

**File:** en/using-flowise/document-stores.md (L199-207)
```markdown
## 8. Test Your Dataset

To quickly test the functionality of your dataset without navigating away from the Document Store, simply utilize the "Retrieval Query" button. This initiates a test query, allowing you to verify the accuracy and effectiveness of your data retrieval process.

<figure><img src="../.gitbook/assets/dastore010.png" alt=""><figcaption></figcaption></figure>

In our case, we see that when querying for information about kitchen flooring coverage in our insurance policy, we retrieve 4 relevant chunks from Upstash, our designated Vector Store. This retrieval is limited to 4 chunks as per the defined "top k" parameter, ensuring we receive the most pertinent information without unnecessary redundancy.

<figure><img src="../.gitbook/assets/dastore009.png" alt=""><figcaption></figcaption></figure>
```

**File:** en/use-cases/upserting-data.md (L70-78)
```markdown
In the context of vector-based retrieval and LLM querying, chunk overlap plays an **important role in maintaining contextual continuity** and **improving response accuracy**, especially when dealing with limited retrieval depth or **top K**, which is the parameter that determines the maximum number of most similar chunks that are retrieved from the [Vector Store](../integrations/langchain/vector-stores/) in response to a query.

During query processing, the LLM executes a similarity search against the Vector Store to retrieve the most semantically relevant chunks to the given query. If the retrieval depth, represented by the top K parameter, is set to a small value, 4 for default, the LLM initially uses information only from these 4 chunks to generate its response.

This scenario presents us with a problem, since relying solely on a limited number of chunks without overlap can lead to incomplete or inaccurate answers, particularly when dealing with queries that require information spanning multiple chunks.

Chunk overlap helps with this issue by ensuring that a portion of the textual context is shared across consecutive chunks, **increasing the likelihood that all relevant information for a given query is contained within the retrieved chunks**.

In other words, this overlap serves as a bridge between chunks, enabling the LLM to access a wider contextual window even when limited to a small set of retrieved chunks (top K). If a query relates to a concept or piece of information that extends beyond a single chunk, the overlapping regions increase the likelihood of capturing all the necessary context.
```

**File:** en/configuration/running-in-production.md (L5-10)
```markdown
When running in production, we highly recommend using [Queue](running-flowise-using-queue.md) mode with the following settings:

* 2 main servers with load balancing, each starting from 1 vCPU 2GB RAM
* 4 workers, each starting from 2 vCPU 4GB RAM

You can configure auto scaling depending on the traffic and volume.
```

**File:** en/configuration/running-in-production.md (L12-14)
```markdown
## Database

By default, Flowise will use SQLite as the database. However when running at scale, its recommended to use PostgresQL.
```

**File:** en/configuration/running-in-production.md (L28-31)
```markdown
## Rate Limit

When deployed to cloud/on-prem, most likely the instances are behind a proxy/load balancer. The IP address of the request might be the IP of the load balancer/reverse proxy, making the rate limiter effectively a global one and blocking all requests once the limit is reached or `undefined`. Setting the correct `NUMBER_OF_PROXIES` can resolve the issue. Refer [#rate-limit-setup](rate-limit.md#rate-limit-setup "mention")

```

**File:** en/configuration/environment-variables.md (L33-48)
```markdown
User can specify `STORAGE_TYPE` to use AWS S3, Google Cloud Storage or local path

| Variable                               | Description                                                                      | Type                              | Default                          |
| -------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------- | -------------------------------- |
| STORAGE\_TYPE                          | Type of storage for uploaded files. default is `local`                           | Enum String: `s3`, `gcs`, `local` | `local`                          |
| BLOB\_STORAGE\_PATH                    | Local folder path where uploaded files are stored when `STORAGE_TYPE` is `local` | String                            | `your-home-dir/.flowise/storage` |
| S3\_STORAGE\_BUCKET\_NAME              | Bucket name to hold the uploaded files when `STORAGE_TYPE` is `s3`               | String                            |                                  |
| S3\_STORAGE\_ACCESS\_KEY\_ID           | AWS Access Key                                                                   | String                            |                                  |
| S3\_STORAGE\_SECRET\_ACCESS\_KEY       | AWS Secret Key                                                                   | String                            |                                  |
| S3\_STORAGE\_REGION                    | Region for S3 bucket                                                             | String                            |                                  |
| S3\_ENDPOINT\_URL                      | Custom S3 endpoint (optional)                                                    | String                            |                                  |
| S3\_FORCE\_PATH\_STYLE                 | Force S3 path style (optional)                                                   | Boolean                           | false                            |
| GOOGLE\_CLOUD\_STORAGE\_CREDENTIAL     | Google Cloud Service Account Key                                                 | String                            |                                  |
| GOOGLE\_CLOUD\_STORAGE\_PROJ\_ID       | Google Cloud Project ID                                                          | String                            |                                  |
| GOOGLE\_CLOUD\_STORAGE\_BUCKET\_NAME   | Google Cloud Storage Bucket Name                                                 | String                            |                                  |
| GOOGLE\_CLOUD\_UNIFORM\_BUCKET\_ACCESS | Type of Access                                                                   | Boolean                           | true                             |
```

**File:** en/configuration/environment-variables.md (L90-97)
```markdown
| FLOWISE\_SECRETKEY\_OVERWRITE | Encryption key to be used instead of the existing key | String                      |                           |
| SECRETKEY\_AWS\_ACCESS\_KEY   |                                                       | String                      |                           |
| SECRETKEY\_AWS\_SECRET\_KEY   |                                                       | String                      |                           |
| SECRETKEY\_AWS\_REGION        |                                                       | String                      |                           |

For some reasons, sometimes encryption key might be re-generated or the stored path was changed, this will cause errors like - <mark style="color:red;">Credentials could not be decrypted.</mark>

To avoid this, you can set your own encryption key as `FLOWISE_SECRETKEY_OVERWRITE`, so that the same encryption key will be used everytime. There is no restriction on the format, you can set it as any text that you want, or the same as your `FLOWISE_PASSWORD`.
```

**File:** en/configuration/environment-variables.md (L117-122)
```markdown
| Variable              | Description                                                                                | Type                      | Default                   |
| --------------------- | ------------------------------------------------------------------------------------------ | ------------------------- | ------------------------- |
| APIKEY\_STORAGE\_TYPE | Method to store API keys                                                                   | Enum string: `json`, `db` | `json`                    |
| APIKEY\_PATH          | Location where the API keys are stored when `APIKEY_STORAGE_TYPE` is unspecified or `json` | String                    | `Flowise/packages/server` |

Using `db` as storage type will store the API keys to database instead of a local JSON file.
```

**File:** en/configuration/environment-variables.md (L126-148)
```markdown
## For Built-In and External Dependencies

There are certain nodes/features within Flowise that allow user to run Javascript code. For security reasons, by default it only allow certain dependencies. It's possible to lift that restriction for built-in and external modules by setting the following environment variables:

| Variable                      | Description                                          |        |
| ----------------------------- | ---------------------------------------------------- | ------ |
| TOOL\_FUNCTION\_BUILTIN\_DEP  | NodeJS built-in modules to be used for Tool Function | String |
| TOOL\_FUNCTION\_EXTERNAL\_DEP | External modules to be used for Tool Function        | String |

{% code title=".env" %}
```bash
# Allows usage of all builtin modules
TOOL_FUNCTION_BUILTIN_DEP=*

# Allows usage of only fs
TOOL_FUNCTION_BUILTIN_DEP=fs

# Allows usage of only crypto and fs
TOOL_FUNCTION_BUILTIN_DEP=crypto,fs

# Allow usage of external npm modules.
TOOL_FUNCTION_EXTERNAL_DEP=axios,moment
```
```

**File:** en/using-flowise/embed.md (L19-31)
```markdown
## Using Specific Version

You can specify the version of flowise-embed's `web.js` to use. For full list of versions: [https://www.npmjs.com/package/flowise-embed](https://www.npmjs.com/package/flowise-embed)

```html
<script type="module">
  import Chatbot from 'https://cdn.jsdelivr.net/npm/flowise-embed@<some-version>/dist/web.js';
  Chatbot.init({
    chatflowid: 'your-chatflowid-here',
    apiHost: 'your-apihost-here',
  })
</script>
```
```

**File:** en/using-flowise/embed.md (L33-43)
```markdown
{% hint style="warning" %}
In Flowise **v2.1.0**, we have modified the way streaming works. If your Flowise version is lower than that, you might find your embedded chatbot not able to receive messages.

You can either update Flowise to **v2.1.0** and above

Or, if for some reason you prefer not to update Flowise, you can specify the latest **v1.x.x** version of [Flowise-Embed](https://www.npmjs.com/package/flowise-embed?activeTab=versions). Last maintained `web.js` version is **v1.3.14.**

For instance:

`https://cdn.jsdelivr.net/npm/flowise-embed@1.3.14/dist/web.js`
{% endhint %}
```

**File:** en/using-flowise/embed.md (L271-294)
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

For example, if you are using `npx flowise start`

```
npx flowise start --CORS_ORIGINS=* --IFRAME_ORIGINS=*
```

If using Docker, place the env variables inside `Flowise/docker/.env`

If using local Git clone, place the env variables inside `Flowise/packages/server/.env`
```

**File:** en/getting-started/README.md (L11-13)
```markdown
{% hint style="info" %}
Pre-requisite: ensure [NodeJS](https://nodejs.org/en/download) is installed on machine. Node `v18.15.0` or `v20` and above is supported.
{% endhint %}
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

**File:** en/configuration/authorization/app-level.md (L61-62)
```markdown
* Use higher `PASSWORD_SALT_HASH_ROUNDS` values (12-15) for better security in production

```

**File:** esp/usar-flowise/monitoring.md (L1-4)
```markdown
# Monitoreo

Flowise tiene soporte nativo para Prometheus con Grafana y OpenTelemetry. Sin embargo, solo se rastrean métricas de alto nivel como solicitudes de API, conteos de flujos/predicciones. Consulta [aquí](https://github.com/FlowiseAI/Flowise/blob/main/packages/server/src/Interface.Metrics.ts#L13) para ver la lista de métricas de contador. Para observabilidad detallada nodo por nodo, recomendamos usar [Analítica](analytic.md).

```

**File:** esp/usar-flowise/monitoring.md (L5-19)
```markdown
## Prometheus

[Prometheus](https://prometheus.io/) es una solución de monitoreo y alertas de código abierto.

Antes de configurar Prometheus, configura las siguientes variables de entorno en Flowise:

```properties
ENABLE_METRICS=true
METRICS_PROVIDER=prometheus
METRICS_INCLUDE_NODE_METRICS=true
```

Después de instalar Prometheus, ejecútalo usando un archivo de configuración. Flowise proporciona un archivo de configuración predeterminado que se puede encontrar [aquí](https://github.com/FlowiseAI/Flowise/blob/main/metrics/prometheus/prometheus.config.yml).

Recuerda tener la instancia de Flowise también en ejecución. Puedes abrir el navegador y navegar al puerto 9090. Desde el panel de control, deberías poder ver que el punto final de métricas - `/api/v1/metrics` está activo.
```

**File:** esp/usar-flowise/monitoring.md (L55-59)
```markdown
Flowise proporciona 2 plantillas de paneles:

* [grafana.dashboard.app.json.txt](https://github.com/FlowiseAI/Flowise/blob/main/metrics/grafana/grafana.dashboard.app.json.txt): métricas de API como número de flujos de chat/agentes, conteo de predicciones, herramientas, asistentes, vectores insertados, etc.
* [grafana.dashboard.server.json.txt](https://github.com/FlowiseAI/Flowise/blob/main/metrics/grafana/grafana.dashboard.server.json.txt): métricas de la instancia node.js de Flowise como uso de heap, CPU, RAM

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

**File:** en/configuration/running-flowise-using-queue.md (L55-100)
```markdown
## Docker Setup

### Method 1: Pre-built Images (Recommended)

This method uses pre-built Docker images from Docker Hub, making it the fastest and most reliable deployment option.

**Step 1: Setup Environment**

Create a `.env` file in the `docker` directory:

```bash
# Basic Configuration
PORT=3000
WORKER_PORT=5566

# Queue Configuration (Required)
MODE=queue
QUEUE_NAME=flowise-queue
REDIS_URL=redis://redis:6379

# Optional Queue Settings
WORKER_CONCURRENCY=5
REMOVE_ON_AGE=24
REMOVE_ON_COUNT=1000
QUEUE_REDIS_EVENT_STREAM_MAX_LEN=1000
ENABLE_BULLMQ_DASHBOARD=false

# Database (Optional - defaults to SQLite)
DATABASE_PATH=/root/.flowise

# Storage
BLOB_STORAGE_PATH=/root/.flowise/storage

# Secret Keys
SECRETKEY_PATH=/root/.flowise

# Logging
LOG_PATH=/root/.flowise/logs
```

**Step 2: Deploy**

```bash
cd docker
docker compose -f docker-compose-queue-prebuilt.yml up -d
```
```

**File:** en/tutorials/customer-support.md (L161-165)
```markdown
#### Step 6: Tools and Knowledge naming and description

Most prebuilt tools come with clear names and descriptions, so users typically don’t need to modify them. However, for custom tools and knowledge bases, providing a clear and descriptive name is essential to ensure the LLM knows when and how to use the appropriate tool. Refer to [best practices for defining functions](https://platform.openai.com/docs/guides/function-calling?api-mode=chat#best-practices-for-defining-functions). You can also use the "**Generate**" button to help with knowledge description:

<figure><img src="../.gitbook/assets/image (330).png" alt="" width="397"><figcaption></figcaption></figure>
```
