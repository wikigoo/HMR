# FlowiseAI Agentflow VPS Deployment Checklist

---

## 1. VPS Prerequisites & Setup

- [ ] Provision a VPS with sufficient resources — for production, the docs recommend at minimum **1 vCPU / 2 GB RAM** per main server instance
- [ ] Install **Node.js** v18.15.0 or v20+ on the VPS (required for npm-based installs)
- [ ] Install **Docker** and **Docker Compose** if using the containerized deployment path
- [ ] Install **Git** if cloning the repository directly
- [ ] Open the required firewall port (default: **3000**) on the VPS
- [ ] Point a domain name at the VPS IP and configure a reverse proxy (e.g., Nginx) with TLS/SSL termination
- [ ] Set `APP_URL` environment variable to your public-facing URL (e.g., `https://your-domain.com`) [1](#0-0) [2](#0-1) [3](#0-2) 

---

## 2. Installation

- [ ] **Option A — NPM:** `npm install -g flowise`, then `npx flowise start`
- [ ] **Option B — Docker Compose:** Clone the repo, copy `docker/.env.example` to `docker/.env`, then run `docker compose up -d`
- [ ] **Option C — Docker Image:** Build with `docker build --no-cache -t flowise .` and run with `docker run -d --name flowise -p 3000:3000 flowise`
- [ ] Verify the instance is reachable at `http://<your-vps-ip>:3000` (or your configured domain)
- [ ] Confirm the health endpoint responds: `GET /api/v1/ping` [4](#0-3) [5](#0-4) 

---

## 3. Database Configuration

- [ ] For development/small deployments: use the default **SQLite** (`DATABASE_TYPE=sqlite`, `DATABASE_PATH=/root/.flowise`)
- [ ] For production at scale: switch to **PostgreSQL** (`DATABASE_TYPE=postgres`) — set `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USER`, `DATABASE_PASSWORD`, `DATABASE_NAME`
- [ ] Enable SSL for the database connection in production: `DATABASE_SSL=true`
- [ ] Set `APIKEY_STORAGE_TYPE=db` when running multiple instances to avoid per-instance JSON file conflicts [6](#0-5) [7](#0-6) 

---

## 4. Environment Variables Configuration

- [ ] Create a `.env` file (based on `.env.example`) in `packages/server` (npm) or `docker/` (Docker)
- [ ] Set `PORT` (default: `3000`)
- [ ] Set `FLOWISE_SECRETKEY_OVERWRITE` to a fixed, strong encryption key — prevents credential decryption errors across restarts
- [ ] Set `LOG_LEVEL` (`error`, `info`, `verbose`, or `debug`) and `LOG_PATH`
- [ ] Set `CORS_ORIGINS` to restrict which domains can make cross-origin requests
- [ ] Set `IFRAME_ORIGINS` to restrict which domains can embed the chat widget
- [ ] *(Optional)* Configure `STORAGE_TYPE=s3` or `gcs` for cloud-based file and log storage
- [ ] *(Optional)* Set `TOOL_FUNCTION_BUILTIN_DEP` and `TOOL_FUNCTION_EXTERNAL_DEP` only if custom tool nodes require specific Node.js modules [8](#0-7) 

---

## 5. Authentication & Access Control

### App-Level Authentication
- [ ] On first launch, complete the **admin account setup** (email + password)
- [ ] Set strong JWT secrets: `JWT_AUTH_TOKEN_SECRET`, `JWT_REFRESH_TOKEN_SECRET`, `TOKEN_HASH_SECRET`
- [ ] Configure token expiry: `JWT_TOKEN_EXPIRY_IN_MINUTES` (default: 60), `JWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES` (default: 90 days)
- [ ] Set `PASSWORD_SALT_HASH_ROUNDS` to 12–15 for production
- [ ] Configure SMTP variables (`SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_SECURE=true`) to enable password reset emails
- [ ] Set `ALLOW_UNAUTHORIZED_CERTS=false` in production

### Flow-Level Authentication
- [ ] In the Flowise dashboard, navigate to **API Keys** and create keys for each Agentflow
- [ ] Assign an API key to each Agentflow via **Settings > Configuration** to restrict public access
- [ ] Require `Authorization: Bearer <api-key>` header on all API calls to protected flows

### Optional: SSO (Enterprise only)
- [ ] Configure OIDC SSO via Microsoft Entra, Google, or Auth0 from the SSO Config panel
- [ ] Invite users before enabling SSO; set `INVITE_TOKEN_EXPIRY_IN_HOURS` and SMTP variables [9](#0-8) [10](#0-9) [11](#0-10) 

---

## 6. Agentflow Deployment & Configuration

- [ ] Build your Agentflow in the **Agentflow Builder** using the visual canvas (Start → LLM/Agent/Tool nodes → Direct Reply)
- [ ] Declare all `$flow.state` keys in the **Start Node** before using them in downstream nodes
- [ ] Verify all **Agent Nodes** have their model, system messages, tools, and document stores configured
- [ ] For any **Tool Node**, confirm input argument mappings use correct `{{ variable }}` syntax
- [ ] For **Condition Nodes**, validate all routing branches are connected and terminate correctly
- [ ] For **Loop Nodes**, ensure a termination condition exists to prevent infinite loops
- [ ] If using **Human-in-the-Loop** nodes, confirm checkpointing is working (execution resumes after restart)
- [ ] Create Flowise **Variables** (static or runtime) for any values that should be centrally managed or pulled from `.env`
- [ ] Export/save the Agentflow JSON as a backup after finalizing the design [12](#0-11) [13](#0-12) [14](#0-13) 

---

## 7. Document Stores & RAG (if applicable)

- [ ] Create and populate **Document Stores** with your data sources before connecting them to Agent nodes
- [ ] Confirm the embedding model and vector store are configured and accessible from the VPS
- [ ] Run an initial upsert to index documents; verify via **Upsert History** (`/api/v1/upsert-history`)
- [ ] *(Optional)* Configure `STORAGE_TYPE` for uploaded files if using file-based document loaders [15](#0-14) [16](#0-15) 

---

## 8. Production Scaling (Queue Mode)

- [ ] For high-traffic deployments, switch to **Queue mode**: set `MODE=queue`
- [ ] Deploy and start a **Redis** instance accessible by both the main server and workers
- [ ] Set `REDIS_URL` pointing to your Redis instance
- [ ] Start one or more **worker processes** (`pnpm run start-worker` or via Docker Compose queue config)
- [ ] Ensure main server and all workers share the **same `FLOWISE_SECRETKEY_OVERWRITE`** value
- [ ] *(Optional)* Enable the BullMQ dashboard: `ENABLE_BULLMQ_DASHBOARD=true`, accessible at `<your-url>/admin/queues`
- [ ] *(Optional)* Configure `WORKER_CONCURRENCY` to tune parallel job processing per worker [17](#0-16) [18](#0-17) [19](#0-18) 

---

## 9. Rate Limiting

- [ ] For publicly accessible Agentflows, configure rate limiting per flow: **Settings > Configuration > Rate Limit** (e.g., 20 messages per 60 seconds)
- [ ] Set `NUMBER_OF_PROXIES` correctly if the VPS is behind a load balancer or reverse proxy
- [ ] Verify the correct client IP is detected via `GET /api/v1/ip` and compare against your actual IP [20](#0-19) [21](#0-20) 

---

## 10. Testing

- [ ] Send a test prediction request to `POST /api/v1/prediction/<agentflow-id>` and verify the response
- [ ] Test each node type in the flow (LLM, Agent, Tool, Condition, Loop) with representative inputs
- [ ] Test Human-in-the-Loop checkpoints: confirm execution pauses and resumes correctly
- [ ] Test API key enforcement: confirm unauthenticated requests to protected flows return `401`
- [ ] Test rate limiting: confirm requests exceeding the limit return the configured limit message
- [ ] *(Cloud/Enterprise)* Run **Evaluations** using Datasets and Evaluators to benchmark output quality, latency, and token usage
- [ ] *(Optional)* Run load tests using the Artillery script provided in the Flowise repo (`artillery-load-test.yml`) [22](#0-21) [23](#0-22) 

---

## 11. Monitoring & Observability

- [ ] Enable step-by-step execution tracing in the Agentflow UI (built-in for Agentflow V2)
- [ ] Set `ENABLE_METRICS=true` and `METRICS_PROVIDER=prometheus` to expose `/api/v1/metrics`
- [ ] Deploy **Prometheus** using the provided config (`metrics/prometheus/prometheus.config.yml`)
- [ ] Connect **Grafana** to Prometheus and import the two provided dashboard templates:
  - `grafana.dashboard.app.json.txt` — flow/prediction counts
  - `grafana.dashboard.server.json.txt` — CPU, RAM, heap usage
- [ ] *(Optional)* Enable **OpenTelemetry** (`METRICS_PROVIDER=open_telemetry`) for APM tool integration (Datadog, Zipkin, New Relic, etc.)
- [ ] *(Optional)* Connect an external analytics provider (LangFuse, Arize, Phoenix, Opik, Lunary) via **Settings > Configuration > Analyse Chatflow** [24](#0-23) [25](#0-24) [26](#0-25) 

---

## 12. Backup & Recovery

- [ ] **SQLite:** Stop Flowise, copy `database.sqlite` to a backup location, restart and verify
- [ ] **PostgreSQL:** Use `pg_dump` to export; restore to a test DB and verify before relying on it
- [ ] **MySQL/MariaDB:** Use `mysqldump`; restore and test similarly
- [ ] Back up the encryption key file (or document the `FLOWISE_SECRETKEY_OVERWRITE` value) — losing it makes stored credentials unrecoverable
- [ ] Back up the `BLOB_STORAGE_PATH` directory (uploaded files, images, audio) if using local storage
- [ ] Schedule automated backups (cron job or cloud snapshot) on a regular cadence [27](#0-26) [28](#0-27) 

---

## 13. Maintenance & Updates

- [ ] Subscribe to the [FlowiseAI GitHub releases](https://github.com/FlowiseAI/Flowise) to track new versions
- [ ] Before updating: back up the database and `.env` file
- [ ] Update via npm: `npm install -g flowise@x.x.x` or pull the latest Docker image and restart containers
- [ ] After updating: re-verify the health endpoint (`/api/v1/ping`) and run a test prediction
- [ ] Review release notes for breaking changes to node configurations or environment variables
- [ ] Monitor logs (`server.log`, `server-requests.log.jsonl`, `server-error.log`) regularly for errors [29](#0-28) [30](#0-29) 

---

## 14. Security Hardening

- [ ] Never expose port 3000 directly to the internet — always use a reverse proxy with HTTPS
- [ ] Set `CORS_ORIGINS` and `IFRAME_ORIGINS` to explicit allowlists, not wildcards
- [ ] Use `SECRETKEY_STORAGE_TYPE=aws` (AWS Secrets Manager) in production for encryption key management
- [ ] Rotate API keys periodically and revoke unused ones from the dashboard
- [ ] Set `EXPIRE_AUTH_TOKENS_ON_RESTART=true` in sensitive environments to invalidate sessions on restart
- [ ] Do not set `TOOL_FUNCTION_BUILTIN_DEP=*` in production — only allow the specific modules your tools require
- [ ] *(Enterprise)* Use **Workspaces** with RBAC to isolate Agentflows between teams; assign least-privilege roles
- [ ] *(Enterprise)* Enable SSO and disable direct password login where possible
- [ ] *(Optional)* Configure `GLOBAL_AGENT_HTTP_PROXY` / `GLOBAL_AGENT_HTTPS_PROXY` if the VPS must route outbound traffic through a corporate proxy [31](#0-30) [32](#0-31) [33](#0-32) [34](#0-33) [35](#0-34)

### Citations

**File:** en/getting-started/README.md (L11-13)
```markdown
{% hint style="info" %}
Pre-requisite: ensure [NodeJS](https://nodejs.org/en/download) is installed on machine. Node `v18.15.0` or `v20` and above is supported.
{% endhint %}
```

**File:** en/getting-started/README.md (L17-78)
```markdown
1. Install Flowise:

```bash
npm install -g flowise
```

You can also install a specific version. Refer to available [versions](https://www.npmjs.com/package/flowise?activeTab=versions).

```
npm install -g flowise@x.x.x
```

2. Start Flowise:

```bash
npx flowise start
```

3. Open: [http://localhost:3000](http://localhost:3000)

***

## Docker

There are two ways to deploy Flowise with Docker. First, git clone the project: [https://github.com/FlowiseAI/Flowise](https://github.com/FlowiseAI/Flowise)

### Docker Compose

1. Go to `docker folder` at the root of the project
2. Copy the `.env.example` file and paste it as another file named `.env`
3. Run:

```bash
docker compose up -d
```

4. Open: [http://localhost:3000](http://localhost:3000)
5. You can bring the containers down by running:

```bash
docker compose stop
```

### Docker Image

1. Build the image:

```bash
docker build --no-cache -t flowise .
```

2. Run image:

```bash
docker run -d --name flowise -p 3000:3000 flowise
```

3. Stop image:

```bash
docker stop flowise
```
```

**File:** en/configuration/running-in-production.md (L1-10)
```markdown
# Running in Production

## Mode

When running in production, we highly recommend using [Queue](running-flowise-using-queue.md) mode with the following settings:

* 2 main servers with load balancing, each starting from 1 vCPU 2GB RAM
* 4 workers, each starting from 2 vCPU 4GB RAM

You can configure auto scaling depending on the traffic and volume.
```

**File:** en/configuration/running-in-production.md (L12-26)
```markdown
## Database

By default, Flowise will use SQLite as the database. However when running at scale, its recommended to use PostgresQL.

## Storage

Currently Flowise only supports [AWS S3](https://aws.amazon.com/s3/) with plan to support more blob storage providers. This will allow files and logs to be stored on S3, instead of local file path. Refer [#for-storage](environment-variables.md#for-storage "mention")

## Encryption

Flowise uses an encryption key to encrypt/decrypt credentials you use such as OpenAI API keys. [AWS Secret Manager](https://aws.amazon.com/secrets-manager/) is recommended to be used in production for better security control and key rotation. Refer [#for-credentials](environment-variables.md#for-credentials "mention")

## API Key Storage

Users can create multiple API keys within Flowise in order to authenticate with the [APIs](broken-reference). By default, keys get stored as a JSON file to your local file path. However when you have multiple instances, each instance will create a new JSON file, causing confusion. You can change the behaviour to store into database instead. Refer [#for-flowise-api-keys](environment-variables.md#for-flowise-api-keys "mention")
```

**File:** en/configuration/running-in-production.md (L28-31)
```markdown
## Rate Limit

When deployed to cloud/on-prem, most likely the instances are behind a proxy/load balancer. The IP address of the request might be the IP of the load balancer/reverse proxy, making the rate limiter effectively a global one and blocking all requests once the limit is reached or `undefined`. Setting the correct `NUMBER_OF_PROXIES` can resolve the issue. Refer [#rate-limit-setup](rate-limit.md#rate-limit-setup "mention")

```

**File:** en/configuration/running-in-production.md (L32-34)
```markdown
## Load Testing

Artillery can be used to load testing your deployed Flowise application. Example script can be found [here](https://github.com/FlowiseAI/Flowise/blob/main/artillery-load-test.yml).
```

**File:** en/configuration/authorization/app-level.md (L21-21)
```markdown
* `APP_URL` - Your hosted Flowise appication URL. Default to `http://localhost:3000`
```

**File:** en/configuration/authorization/app-level.md (L26-61)
```markdown

* `JWT_AUTH_TOKEN_SECRET` - The secret key for signing access tokens
* `JWT_REFRESH_TOKEN_SECRET` - Secret for refresh tokens (defaults to auth token secret if not set)
* `JWT_TOKEN_EXPIRY_IN_MINUTES` - Access token lifetime (default: 60 minutes)
* `JWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES` - Refresh token lifetime (default: 129,600 minutes or 90 days)
* `JWT_AUDIENCE` - Token validation audience claim (default: 'AUDIENCE')
* `JWT_ISSUER` - Token validation issuer claim (default: 'ISSUER')
* `EXPRESS_SESSION_SECRET` - Session encryption secret (default: 'flowise')
* `EXPIRE_AUTH_TOKENS_ON_RESTART` - Set to 'true' to invalidate all tokens on server restart (useful for development)

### SMTP Email Configuration

Configure these variables to enable email functionality for password resets, and notifications:

* `SMTP_HOST` - The hostname of your SMTP server (e.g., `smtp.gmail.com`, `smtp.host.com`)
* `SMTP_PORT` - The port number for SMTP connection (common values: `587` for TLS, `465` for SSL, `25` for unencrypted)
* `SMTP_USER` - Username for SMTP authentication (usually your email address)
* `SMTP_PASSWORD` - Password or app-specific password for SMTP authentication
* `SMTP_SECURE` - Set to `true` for SSL/TLS encryption, `false` for unencrypted connections
* `ALLOW_UNAUTHORIZED_CERTS` - Set to `true` to allow self-signed certificates (not recommended for production)
* `SENDER_EMAIL` - The "from" email address that will appear on outgoing emails

### Security and Token Configuration

These variables control authentication security, token expiration, and password hashing:

* `PASSWORD_RESET_TOKEN_EXPIRY_IN_MINS` - Expiration time for password reset tokens (default: 15 minutes)
* `PASSWORD_SALT_HASH_ROUNDS` - Number of bcrypt salt rounds for password hashing (default: 10, higher = more secure but slower)
* `TOKEN_HASH_SECRET` - Secret key used for hashing tokens and sensitive data (use a strong, random string)

### Security Best Practices

* Use strong, unique values for `TOKEN_HASH_SECRET` and store them securely
* For production, use `SMTP_SECURE=true` and `ALLOW_UNAUTHORIZED_CERTS=false`
* Set appropriate token expiry times based on your security requirements
* Use higher `PASSWORD_SALT_HASH_ROUNDS` values (12-15) for better security in production
```

**File:** en/configuration/running-flowise-using-queue.md (L1-13)
```markdown
# Running Flowise using Queue

By default, Flowise runs in a NodeJS main thread. However, with large number of predictions, this does not scale well. Therefore there are 2 modes you can configure: `main` (default) and `queue`.

## Queue Mode

With the following environment variables, you can run Flowise in `queue` mode.

<table><thead><tr><th width="263">Variable</th><th>Description</th><th>Type</th><th>Default</th></tr></thead><tbody><tr><td>MODE</td><td>Mode to run Flowise</td><td>Enum String: <code>main</code>, <code>queue</code></td><td><code>main</code></td></tr><tr><td>WORKER_CONCURRENCY</td><td>How many jobs are allowed to be processed in parallel for a worker. If you have 1 worker, that means how many concurrent prediction tasks it can handle. More <a href="https://docs.bullmq.io/guide/workers/concurrency">info</a></td><td>Number</td><td>10000</td></tr><tr><td>QUEUE_NAME</td><td>The name of the message queue</td><td>String</td><td>flowise-queue</td></tr><tr><td>QUEUE_REDIS_EVENT_STREAM_MAX_LEN</td><td>Event stream is auto-trimmed so that its size does not grow too much. More <a href="https://docs.b ... (truncated)

In `queue` mode, the main server will be responsible for processing requests, sending jobs to message queue. Main server will not execute the job. One or multiple workers receive jobs from the queue, execute them and send the results back.

This allows for dynamic scaling: you can add workers to handle increased workloads or remove them during lighter periods.
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

**File:** en/configuration/running-flowise-using-queue.md (L150-156)
```markdown
```bash
# Check main instance health
curl http://localhost:3000/api/v1/ping

# Check worker health
curl http://localhost:5566/healthz
```
```

**File:** en/configuration/databases.md (L13-63)
```markdown
- SQLite
- MySQL
- PostgreSQL
- MariaDB

### SQLite (Default)

SQLite will be the default database. These databases can be configured with following env variables:

```sh
DATABASE_TYPE=sqlite
DATABASE_PATH=/root/.flowise #your preferred location
```

A `database.sqlite` file will be created and saved in the path specified by `DATABASE_PATH`. If not specified, the default store path will be in your home directory -> .flowise

**Note:** If none of the env variables is specified, SQLite will be the fallback database choice.

### MySQL

```sh
DATABASE_TYPE=mysql
DATABASE_PORT=3306
DATABASE_HOST=localhost
DATABASE_NAME=flowise
DATABASE_USER=user
DATABASE_PASSWORD=123
```

### PostgreSQL

```sh
DATABASE_TYPE=postgres
DATABASE_PORT=5432
DATABASE_HOST=localhost
DATABASE_NAME=flowise
DATABASE_USER=user
DATABASE_PASSWORD=123
PGSSLMODE=require
```

### MariaDB

```bash
DATABASE_TYPE="mariadb"
DATABASE_PORT="3306"
DATABASE_HOST="localhost"
DATABASE_NAME="flowise"
DATABASE_USER="flowise"
DATABASE_PASSWORD="mypassword"
```
```

**File:** en/configuration/databases.md (L69-136)
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

### PostgreSQL

1. Backup database.

   ```bash
   pg_dump -U USERNAME -h HOST -p PORT -d DATABASE_NAME -f /PATH/TO/BACKUP_FILE_NAME.sql
   ```

2. Enter database password.
3. Create test database.
   ```bash
   psql -U USERNAME -h HOST -p PORT -d TEST_DATABASE_NAME -f /PATH/TO/BACKUP_FILE_NAME.sql
   ```
4. Test the backup database by running Flowise with the `.env` file modified to point to the backup database.

### MySQL & MariaDB

1. Backup database.

   ```bash
   mysqldump -u USERNAME -p DATABASE_NAME > BACKUP_FILE_NAME.sql
   ```

2. Enter database password.
3. Create test database.
   ```bash
   mysql -u USERNAME -p TEST_DATABASE_NAME < BACKUP_FILE_NAME.sql
   ```
4. Test the backup database by running Flowise with the `.env` file modified to point to the backup database.
```

**File:** en/configuration/environment-variables.md (L7-97)
```markdown
Flowise support different environment variables to configure your instance. You can specify the following variables in the `.env` file inside `packages/server` folder. Refer to [.env.example](https://github.com/FlowiseAI/Flowise/blob/main/packages/server/.env.example) file.

<table><thead><tr><th width="233">Variable</th><th width="219">Description</th><th width="104">Type</th><th>Default</th></tr></thead><tbody><tr><td>PORT</td><td>The HTTP port Flowise runs on</td><td>Number</td><td>3000</td></tr><tr><td>FLOWISE_FILE_SIZE_LIMIT</td><td>Maximum file size when uploading</td><td>String</td><td><code>50mb</code></td></tr><tr><td>NUMBER_OF_PROXIES</td><td>Rate Limit Proxy</td><td>Number</td><td></td></tr><tr><td>CORS_ORIGINS</td><td>The allowed origins for all cross-origin HTTP calls</td><td>String</td><td></td></tr><tr><td>IFRAME_ORIGINS</td><td>The allowed origins for iframe src embedding</td><td>String</td><td></td></tr><tr><td>SHOW_COMMUNITY_NODES</td><td>Display nodes that are created by community</td><td>Boolean: <code>true</code> or <code>false</code></td> ... (truncated)

## For Database

| Variable           | Description                                                      | Type                                       | Default                  |
| ------------------ | ---------------------------------------------------------------- | ------------------------------------------ | ------------------------ |
| DATABASE\_TYPE     | Type of database to store the flowise data                       | Enum String: `sqlite`, `mysql`, `postgres` | `sqlite`                 |
| DATABASE\_PATH     | Location where database is saved (When DATABASE\_TYPE is sqlite) | String                                     | `your-home-dir/.flowise` |
| DATABASE\_HOST     | Host URL or IP address (When DATABASE\_TYPE is not sqlite)       | String                                     |                          |
| DATABASE\_PORT     | Database port (When DATABASE\_TYPE is not sqlite)                | String                                     |                          |
| DATABASE\_USER     | Database username (When DATABASE\_TYPE is not sqlite)            | String                                     |                          |
| DATABASE\_PASSWORD | Database password (When DATABASE\_TYPE is not sqlite)            | String                                     |                          |
| DATABASE\_NAME     | Database name (When DATABASE\_TYPE is not sqlite)                | String                                     |                          |
| DATABASE\_SSL      | Database SSL is required (When DATABASE\_TYPE is not sqlite)     | Boolean: `true` or `false`                 | `false`                  |

## For Storage

Flowise store the following files under a local path folder by default.

* Files uploaded on [Document Loaders](../integrations/langchain/document-loaders/)/Document Store
* Image/Audio uploads from chat
* Images/Files from Assistant
* Files from [Vector Upsert API](broken-reference)

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

### Logs Streaming S3

When `STORAGE_TYPE` env variable is set to `s3` , logs will be automatically streamed and stored to S3. New log file will be created hourly, enabling easier debugging.

### Logs Streaming GCS

When `STORAGE_TYPE` env variable is set to `gcs` , logs will be automatically streamed to Google [Cloud Logging](https://cloud.google.com/logging?hl=en).

## For Credentials

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

**File:** en/configuration/environment-variables.md (L126-149)
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
{% endcode %}
```

**File:** en/configuration/authorization/chatflow-level.md (L9-33)
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

An example of calling the API using POSTMAN

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
```

**File:** en/configuration/sso.md (L1-7)
```markdown
# SSO

{% hint style="info" %}
SSO is only available for Enterprise plan
{% endhint %}

Flowise supports [OIDC](https://openid.net/) that allows users to use _single sign_-on (_SSO_) to access application. Currently only the [Organization Admin](../using-flowise/workspaces.md#setting-up-admin-account) can configure the SSO configurations.
```

**File:** en/using-flowise/agentflowv2.md (L39-46)
```markdown
### 🙋‍♂ Human-in-the-loop

Execution is paused while awaiting human input, without blocking the running thread. Each checkpoint is saved, allowing the workflow to resume from the same point even after an application restart.

The use of checkpoints enables **long-running, stateful agents**.

Agents can also be configured to **request permission before executing tools**, similar to how Claude asks for user approval before using MCP tools. This helps prevent the autonomous execution of sensitive actions without explicit user approval.

```

**File:** en/using-flowise/agentflowv2.md (L75-87)
```markdown
### **1. Start Node**

The designated entry point for initiating any AgentFlow V2 workflow execution. Every flow must begin with this node.

* **Functionality:** Defines how the workflow is triggered and sets up the initial conditions. It can accept input either directly from the chat interface or through a customizable form presented to the user. It also allows for the initialization of `Flow State` variables at the beginning of the execution and can manage how conversation memory is handled for the run.
* **Configuration Parameters**
  * **Input Type**: Determines how the workflow execution is initiated, either by `Chat Input` from the user or via a submitted `Form Input`.
    * **Form Title, Form Description, Form Input Types**: If `Form Input` is selected, these fields configure the appearance of the form presented to the user, allowing for various input field types with defined labels and variable names.
  * **Ephemeral Memory**: If enabled, instructs the workflow to begin the execution without considering any past messages from the conversation thread, effectively starting with a clean memory slate.
  * **Flow State**: Defines the complete set of initial key-value pairs for the workflow's runtime state `$flow.state`. All state keys that will be used or updated by subsequent nodes must be declared and initialized here.
* **Inputs:** Receives the initial data that triggers the workflow, which will be either a chat message or the data submitted through a form.
* **Outputs:** Provides a single output anchor to connect to the first operational node, passing along the initial input data and the initialized Flow State.

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

**File:** en/using-flowise/variables.md (L9-21)
```markdown
Flowise allow users to create variables that can be used in the nodes. Variables can be Static or Runtime.

### Static

Static variable will be saved with the value specified, and retrieved as it is.

<figure><img src="../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png" alt="" width="542"><figcaption></figcaption></figure>

### Runtime

Value of the variable will be fetched from **.env** file using `process.env`

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="537"><figcaption></figcaption></figure>
```

**File:** en/configuration/rate-limit.md (L9-31)
```markdown
When you share your chatflow to public with no API authorization through API or embedded chat, anybody can access the flow. To prevent spamming, you can set the rate limit on your chatflow.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="462"><figcaption></figcaption></figure>

* **Message Limit per Duration**: How many messages can be received in a specific duration. Ex: 20
* **Duration in Seconds**: The specified duration. Ex: 60
* **Limit Message**: What message to return when the limit is exceeded. Ex: Quota Exceeded

Using the example above, that means only 20 messages are allowed to be received in 60 seconds. The rate limitation is tracked by IP-address. If you have deployed Flowise on cloud service, you'll have to set `NUMBER_OF_PROXIES` env variable.

## Rate Limit Setup

When you are hosting Flowise on cloud such as AWS, GCP, Azure, etc, most likely there you are behind a proxy/load balancer. Therefore, the rate limit might not be able to work. More info can be found [here](https://github.com/express-rate-limit/express-rate-limit/wiki/Troubleshooting-Proxy-Issues).

To fix the issue:

1. **Set Environment Variable:** Create an environment variable named `NUMBER_OF_PROXIES` and set its value to `0` in your hosting environment.
2. **Restart your hosted Flowise instance:** This enables Flowise to apply changes of environment variables.
3. **Check IP Address:** To verify the IP address, access the following URL: `{{hosted_url}}/api/v1/ip`. You can do this either by entering the URL into your web browser or by making an API request.
4. **Compare IP Address** After making the request, compare the IP address returned to your current IP address. You can find your current IP address by visiting either of these websites:
   * [http://ip.nfriedly.com/](http://ip.nfriedly.com/)
   * [https://api.ipify.org/](https://api.ipify.org/)
5. **Incorrect IP Address:** If the returned IP address does not match your current IP address, increase `NUMBER_OF_PROXIES` by 1 and restart your Flowise instance. Repeat this process until the IP address matches your own.
```

**File:** en/using-flowise/evaluations.md (L7-11)
```markdown
Evaluations help you monitor and understand the performance of your Chatflow/Agentflow application. On the high level, an evaluation is a process that takes a set of inputs and corresponding outputs from your Chatflow/Agentflow, and generates scores. These scores can be derived by comparing outputs to reference results, such as through string matching, numeric comparison, or even leveraging an LLM as a judge. These evaluations are conducted using Datasets and Evaluators.

## Datasets

Datasets are the inputs that will be used to run your Chatflow/Agentflow, along with the corresponding outputs for comparison. User can add the input and anticipated output manually, or upload a CSV file with 2 columns: Input and Output.
```

**File:** en/using-flowise/monitoring.md (L1-19)
```markdown
# Monitoring

Flowise has native support for Prometheus with Grafana and OpenTelemetry. However, only high-level metrics such as API requests, counts of flows/predictions are tracked. Refer [here](https://github.com/FlowiseAI/Flowise/blob/main/packages/server/src/Interface.Metrics.ts#L13) for the lists of counter metrics. For details node by node observability, we recommend using [Analytic](broken-reference).

## Prometheus

[Prometheus](https://prometheus.io/) is an open-source monitoring and alerting solution.

Before setting up Prometheus, configure the following env variables in Flowise:

```properties
ENABLE_METRICS=true
METRICS_PROVIDER=prometheus
METRICS_INCLUDE_NODE_METRICS=true
```

After Prometheus is installed, run it using a configuration file. Flowise provides a default configuration file that can be found [here](https://github.com/FlowiseAI/Flowise/blob/main/metrics/prometheus/prometheus.config.yml).

Remember to have Flowise instance also running. You can open browser and navigate to port 9090. From the dashboard, you should be able to see the metric endpoint - `/api/v1/metrics` is now live.
```

**File:** en/using-flowise/monitoring.md (L55-95)
```markdown
Flowise provides 2 template dashboards:

* [grafana.dashboard.app.json.txt](https://github.com/FlowiseAI/Flowise/blob/main/metrics/grafana/grafana.dashboard.app.json.txt): API metrics such as number of chatflows/agentflows, predictions count, tools, assistant, upserted vectors, etc.
* [grafana.dashboard.server.json.txt](https://github.com/FlowiseAI/Flowise/blob/main/metrics/grafana/grafana.dashboard.server.json.txt): metrics of the Flowise node.js instance such as heap, CPU, RAM usage

If you are using templates above, find and replace all occurence of `cds4j1ybfuhogb` with the data source ID you created and saved earlier.

<figure><img src="../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

You can also choose to import first then edit the JSON later:

<figure><img src="../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

Now, try to perform some actions on the Flowise, you should be able to see the metrics displayed:

<figure><img src="../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

## OpenTelemetry

[OpenTelemetry](https://opentelemetry.io/) is an open source framework for creating and managing telemetry data. To enable OTel, configure the following env variables in Flowise:

```properties
ENABLE_METRICS=true
METRICS_PROVIDER=open_telemetry
METRICS_INCLUDE_NODE_METRICS=true
METRICS_OPEN_TELEMETRY_METRIC_ENDPOINT=http://localhost:4318/v1/metrics
METRICS_OPEN_TELEMETRY_PROTOCOL=http # http | grpc | proto (default is http)
METRICS_OPEN_TELEMETRY_DEBUG=true
```

Next, we need OpenTelemetry Collector to receive, process and export telemetry data. Flowise provides a [docker compose file](https://github.com/FlowiseAI/Flowise/blob/main/metrics/otel/compose.yaml) which can be used to start the collector container.

```bash
cd Flowise
cd metrics && cd otel
docker compose up -d
```

The collector will be using the [otel.config.yml](https://github.com/FlowiseAI/Flowise/blob/main/metrics/otel/otel.config.yml) file under the same directory for configurations. Currently only [Datadog](https://www.datadoghq.com/) and Prometheus are supported, refer to the [Open Telemetry](https://opentelemetry.io/) documentation to configure different APM tools such as Zipkin, Jeager, New Relic, Splunk and others.
```

**File:** en/using-flowise/analytics/README.md (L9-39)
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

## Setup

1. At the top right corner of your Chatflow or Agentflow, click **Settings** > **Configuration**

<figure><img src="../../.gitbook/assets/analytic-1.webp" alt="Screenshot of user clicking in the configuration menu" width="375"><figcaption></figcaption></figure>

2. Then go to the Analyse Chatflow section

<figure><img src="../../.gitbook/assets/analytic-2.png" alt="Screenshot of the Analyse Chatflow section with the different Analytics providers"><figcaption></figcaption></figure>

3. You will see a list of providers, along with their configuration fields

<figure><img src="../../.gitbook/assets/image (82).png" alt="Screenshot of an analytics provider with credentials fields expanded"><figcaption></figcaption></figure>

4. Fill in the credentials and other configuration details, then turn the provider **ON**. Click Save.

<figure><img src="../../.gitbook/assets/image (83).png" alt="Screenshot of analytics providers enabled"><figcaption></figcaption></figure>
```

**File:** en/configuration/running-flowise-behind-company-proxy.md (L9-15)
```markdown
There are 2 environment variables you will need to run Flowise behind a company proxy:

| Variable                   | Purpose                                                                          | Required |
| -------------------------- | -------------------------------------------------------------------------------- | -------- |
| `GLOBAL_AGENT_HTTP_PROXY`  | Where to proxy all server HTTP requests through                                  | Yes      |
| `GLOBAL_AGENT_HTTPS_PROXY` | Where to proxy all server HTTPS requests through                                 | No       |
| `GLOBAL_AGENT_NO_PROXY`    | A pattern of URLs that should be excluded from proxying. Eg. `*.foo.com,baz.com` | No       |
```

**File:** en/using-flowise/workspaces.md (L7-7)
```markdown
Upon your initial login, a default workspace will be automatically generated for you. Workspaces serve to partition resources among various teams or business units. Inside each workspace, Role-Based Access Control (RBAC) is used to manage permissions and access, ensuring users have access only to the resources and settings required for their role.
```
