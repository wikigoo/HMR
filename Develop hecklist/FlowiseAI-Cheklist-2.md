## Agentflow Health Evaluation Checklist

---

## 1. Introduction

This checklist evaluates a multi-agent Agentflow system built on FlowiseAI, deployed on a Hetzner VPS, and serving both a web chatbot and a Flutter Android application. The evaluation is grounded in FlowiseAI's actual execution architecture: agentflows use a **queue-based, dependency-driven execution engine** (`buildAgentflow.ts`) distinct from the simpler BFS-based chatflow engine, with node types including `Agent`, `LLM`, `Tool`, `CustomFunction`, `ConditionAgent`, `Loop`, and `Retriever`. [1](#0-0) [2](#0-1) 

---

## 2. Findings

### 2.1 Node Configuration

| # | Check | What to Verify |
|---|-------|----------------|
| N1 | **Starting node integrity** | Confirm exactly one node has zero incoming dependencies (`getStartingNode` expects a single entry point). Multiple or missing start nodes cause silent execution failures. |
| N2 | **Worker/Supervisor wiring** | In multi-agent (Supervisor/Worker) mode, every `Worker` node must have its `supervisor` input connected. The `Worker.init()` throws `"Worker name is required!"` and `"Worker prompt is required!"` if these are missing. |
| N3 | **Tool-calling model compatibility** | `Worker` nodes that use tools require a model with `bindTools` support (ChatOpenAI, ChatAnthropic, ChatGoogleGenerativeAI, GroqChat, ChatMistral, ChatVertexAI). Incompatible models throw `"This agent only compatible with function calling models."` |
| N4 | **`maxIterations` on Worker nodes** | Unset `maxIterations` means unbounded agent loops. Verify a sensible cap is set per worker. |
| N5 | **Loop node `MAX_LOOP_COUNT`** | Default is 10, configurable via env var. Confirm `MAX_LOOP_COUNT` is set appropriately to prevent runaway loops. |
| N6 | **Condition node output anchors** | `ConditionAgentflow` and `ConditionAgent` nodes dynamically build output anchors based on the number of configured conditions/scenarios. Verify all branches are wired to downstream nodes; unwired branches silently drop execution paths. |
| N7 | **Unused/orphaned nodes** | Nodes not reachable from the start node are never executed but still loaded into memory at initialization. Audit the canvas for disconnected nodes. |
| N8 | **Variable resolution correctness** | All `{{$vars.name}}`, `{{$flow.state.key}}`, and `{{nodeId.output.path}}` references must resolve to values populated by the time the referencing node executes. Verify execution order guarantees this. |
| N9 | **Credential binding** | Each node requiring an API key must have a valid `FLOWISE_CREDENTIAL_ID` bound. Unbound credentials cause runtime errors, not canvas-time errors. | [3](#0-2) [4](#0-3) [5](#0-4) 

---

### 2.2 Knowledge Base Integration

| # | Check | What to Verify |
|---|-------|----------------|
| K1 | **Retriever node connections** | Each `Retriever_Agentflow` node must be connected to a vector store. Confirm the vector store type (Pinecone, Postgres/pgvector, Qdrant, Chroma, etc.) is reachable from the VPS network. |
| K2 | **RecordManager presence** | Without a `RecordManager` node attached to the vector store, re-upserts create duplicate embeddings. Verify `cleanup` mode is set to `incremental` or `full` for managed knowledge bases. |
| K3 | **Embedding model consistency** | The embedding model used during upsert must match the one used at query time. A mismatch produces semantically incorrect similarity scores. |
| K4 | **Document store status** | After each upsert, the `DocumentStore` entity status transitions: `UPSERTING` → `UPSERTED`. Check the Flowise UI or DB for stores stuck in `UPSERTING`, which indicates a failed indexing job. |
| K5 | **Chunk size and overlap** | Verify text splitter `chunkSize` and `chunkOverlap` are tuned for the domain. Overly large chunks reduce retrieval precision; overly small chunks lose context. |
| K6 | **Metadata filtering** | If sub-agents serve different domains, confirm metadata filters (e.g., `pineconeMetadataFilter`, `qdrantFilter`) are applied per retriever to prevent cross-domain contamination. |
| K7 | **Knowledge base refresh schedule** | Confirm a process exists to re-upsert documents when source content changes. The `/api/v1/document-store/refresh/:id` endpoint supports programmatic refresh. | [6](#0-5) [7](#0-6) [8](#0-7) 

---

### 2.3 Technical Debt Review

| # | Check | What to Verify |
|---|-------|----------------|
| T1 | **Agentflow version** | FlowiseAI has two agentflow architectures: the legacy Supervisor/Worker multi-agent system (`packages/components/nodes/multiagents/`) and the newer V2 agentflow (`packages/components/nodes/agentflow/`). Confirm which is in use and whether migration to V2 is warranted for new features. |
| T2 | **`vm2` sandbox deprecation** | Custom Function nodes use `vm2` for sandboxed JS execution. `vm2` is deprecated upstream. Check if the deployed Flowise version has migrated to an alternative sandbox. |
| T3 | **`@langchain/classic` usage** | Worker nodes import from `@langchain/classic/agents`, a compatibility shim for deprecated LangChain v0.x APIs. This is a known technical debt item in the codebase. |
| T4 | **Flutter API integration pattern** | Verify the Flutter app uses the `/api/v1/prediction/{id}` endpoint with proper `sessionId` per user to maintain conversation isolation. Shared or missing `sessionId` values cause cross-user memory contamination. |
| T5 | **Hardcoded API keys in Flutter** | Confirm API keys are not embedded in the Flutter APK. They should be fetched from a secure backend or passed via environment-specific config. |
| T6 | **CORS configuration** | The `CORS_ORIGINS` and `IFRAME_ORIGINS` env vars must explicitly whitelist the website domain and any Flutter deep-link origins. Overly permissive `*` settings are a security risk. | [9](#0-8) [10](#0-9) 

---

### 2.4 Performance and Speed Optimization

| # | Check | What to Verify |
|---|-------|----------------|
| P1 | **Deployment mode** | Default mode is single-process (`main`). Under concurrent load from both web and Android clients, this creates a bottleneck. Evaluate switching to `MODE=queue` with BullMQ + Redis for parallel execution. |
| P2 | **Database backend** | Default is SQLite, which serializes writes. For production multi-client use, migrate to PostgreSQL (`DATABASE_TYPE=postgres`). |
| P3 | **Response streaming** | Confirm the Flutter app and web widget use streaming (`streaming: true` in the prediction request body). Non-streaming responses block until the entire agent graph completes, increasing perceived latency. |
| P4 | **Memory strategy per agent** | `allMessages` memory type loads the entire chat history on every turn. For long conversations, switch to `windowSize` or `summaryBuffer` to reduce token consumption and LLM latency. |
| P5 | **Caching** | Flowise supports `RedisCache` for LLM response caching. For frequently repeated queries (e.g., FAQ-style chatbot), enabling caching can eliminate redundant LLM calls. |
| P6 | **Supervisor recursion limit** | The `recursionLimit` on the Supervisor node caps the total number of agent-to-agent handoffs. Too low causes premature termination; too high allows runaway costs. Tune based on observed workflow depth. |
| P7 | **VPS resource sizing** | Monitor CPU and RAM on the Hetzner VPS. Node.js is single-threaded; LLM inference calls are I/O-bound but embedding and document processing are CPU-intensive. Confirm the VPS has sufficient RAM for the loaded `NodesPool` and active sessions. |
| P8 | **Prometheus metrics** | Flowise exposes `http_requests_total`, `http_request_duration_ms`, and `checkouts_total` at `/api/v1/metrics` when `ENABLE_METRICS=true`. Confirm this is enabled and scraped. | [11](#0-10) [12](#0-11) [13](#0-12) 

---

### 2.5 Update Management

| # | Check | What to Verify |
|---|-------|----------------|
| U1 | **Flowise version** | Run `flowise --version` or check `package.json` on the VPS. Compare against the latest release on the [Flowise GitHub releases page](https://github.com/FlowiseAI/Flowise/releases). |
| U2 | **LangChain core version** | The codebase pins `@langchain/core: 1.1.20`. Verify the deployed version matches and that no breaking changes exist in newer versions affecting your node types. |
| U3 | **Node.js version** | Check `.nvmrc` in the repo for the required Node.js version. Mismatched Node.js versions on the VPS cause subtle runtime errors. |
| U4 | **Flutter SDK and dependencies** | Run `flutter pub outdated` in the Flutter project. Pay particular attention to the HTTP client package used for Flowise API calls and any WebSocket/SSE libraries for streaming. |
| U5 | **Post-update compatibility test** | After any Flowise update, re-export and re-import the agentflow JSON to verify node schemas haven't changed in a breaking way. Use the `/api/v1/chatflows/export` and `/api/v1/chatflows/import` endpoints. |
| U6 | **Security patches** | Check `SECURITY.md` in the Flowise repo for disclosed CVEs. The `HTTP_SECURITY_CHECK`, `PATH_TRAVERSAL_SAFETY`, and `OAUTH2_SECURITY_CHECK` env vars should be enabled. | [14](#0-13) [10](#0-9) 

---

### 2.6 Backup and Recovery

| # | Check | What to Verify |
|---|-------|----------------|
| B1 | **Database backup** | If using SQLite, the database file is at `DATABASE_PATH` (default `~/.flowise`). Schedule daily `cp` or `rsync` to off-VPS storage. If using PostgreSQL, use `pg_dump` with a cron job. |
| B2 | **Agentflow export** | Export all agentflows via the Flowise UI or `GET /api/v1/chatflows/export`. Store the JSON in version control (Git). This is the only portable representation of the flow graph. |
| B3 | **Vector store backup** | Vector store data lives outside Flowise (Pinecone, Postgres, Qdrant, etc.). Confirm the vector store provider has its own backup/snapshot policy. For self-hosted Postgres pgvector, include the vector table in `pg_dump`. |
| B4 | **Blob storage backup** | Uploaded files are stored at `BLOB_STORAGE_PATH` (default `~/.flowise`). Include this directory in backups. |
| B5 | **Credential store backup** | Credentials are encrypted and stored in the Flowise database. The encryption key is `FLOWISE_SECRETKEY_OVERWRITE`. Back up both the DB and the secret key separately — losing the key makes credentials unrecoverable. |
| B6 | **Restore test** | Periodically restore from backup to a staging environment and verify the agentflow executes correctly end-to-end. | [15](#0-14) [16](#0-15) 

---

### 2.7 Agentflow Improvements

| # | Opportunity | Rationale |
|---|-------------|-----------|
| A1 | **Migrate to Agentflow V2** | The V2 architecture (`packages/agentflow/`) supports richer node types (`ConditionAgent`, `Loop`, `Retriever`, `CustomFunction`) with explicit dependency resolution and better state management than the legacy Supervisor/Worker model. |
| A2 | **Add `ConditionAgent` for intent routing** | Replace hardcoded routing logic with a `ConditionAgent` node that uses an LLM to classify user intent and route to the appropriate sub-agent. This reduces prompt engineering burden on the Supervisor. |
| A3 | **Enable follow-up prompt suggestions** | Flowise has a built-in `followUpPrompts` generator (`packages/components/src/followUpPrompts.ts`). Enable this in the chatbot config to improve UX on both web and Android. |
| A4 | **Add moderation node** | Insert a moderation check at the `START` node of the sequential graph to filter harmful inputs before they reach any LLM. Flowise supports this natively via the moderation interface. |
| A5 | **Implement Human-in-the-Loop for sensitive actions** | For actions with real-world consequences (e.g., form submissions, bookings), use the `interrupt` mechanism on Agent nodes to pause execution and require explicit user confirmation. |
| A6 | **MCP tool integration** | Flowise supports Model Context Protocol (MCP) tools via `CUSTOM_MCP_PROTOCOL`. This enables connecting external tool servers without writing custom Function nodes. |
| A7 | **Evaluation framework** | Use Flowise's built-in evaluation system (wiki page 10.4) to run regression tests against golden Q&A pairs after any knowledge base or prompt update. | [17](#0-16) [18](#0-17) 

---

### 2.8 General Health Check

| # | Check | Command / Endpoint |
|---|-------|--------------------|
| H1 | **Flowise process liveness** | `GET /api/v1/ping` → expects `pong` with HTTP 200. This is also the Docker healthcheck target. |
| H2 | **Worker health (queue mode)** | `GET http://localhost:5566/healthz` → worker health endpoint. |
| H3 | **API key authentication** | Test a prediction request with a valid API key and one without. Confirm unauthenticated requests are rejected with 401. |
| H4 | **Streaming endpoint** | Send a prediction with `"streaming": true` and verify SSE events arrive incrementally. A broken SSE connection causes the Flutter app to time out. |
| H5 | **Session isolation** | Send two concurrent requests with different `sessionId` values. Verify chat histories do not bleed between sessions. |
| H6 | **Log level and sanitization** | Confirm `LOG_LEVEL` is set to `info` or `warn` in production (not `debug`). Verify `LOG_SANITIZE_BODY_FIELDS` includes `apikey`, `password`, `token`. |
| H7 | **Error propagation to clients** | Trigger a deliberate node failure (e.g., invalid tool input) and verify the error is returned to the client in a structured format, not as an unhandled 500 with a stack trace. |
| H8 | **CORS headers** | From the website origin and from the Flutter app's domain, confirm `Access-Control-Allow-Origin` is correctly scoped. |
| H9 | **Disk space** | Check `~/.flowise` disk usage. Uploaded files, SQLite WAL files, and logs accumulate over time. Set `LOG_PATH` and implement log rotation. |
| H10 | **OpenTelemetry / observability** | If `ENABLE_METRICS=true` and `METRICS_PROVIDER=opentelemetry`, confirm the OTLP endpoint (`METRICS_OPEN_TELEMETRY_METRIC_ENDPOINT`) is reachable and metrics are being received. | [19](#0-18) [20](#0-19) [21](#0-20) 

---

## 3. Recommendations

### Critical (address immediately)

1. **Switch database from SQLite to PostgreSQL.** SQLite is not safe for concurrent writes from multiple simultaneous chatbot sessions. Use `DATABASE_TYPE=postgres` with a dedicated Postgres instance on the VPS or a managed service.

2. **Verify `sessionId` handling in the Flutter app.** Each user session must pass a unique, stable `sessionId` in the prediction request body. Without it, all Android users share a single conversation memory context.

3. **Enable API key authentication.** Confirm `FLOWISE_USERNAME` / `FLOWISE_PASSWORD` or JWT auth is active. The prediction endpoint must not be publicly accessible without authentication.

4. **Back up the `FLOWISE_SECRETKEY_OVERWRITE` value.** This key encrypts all stored credentials. Loss of this key requires re-entering all API keys manually.

### High Priority

5. **Enable `MODE=queue` with Redis** if concurrent load from web + Android is observed. The single-process mode serializes all prediction requests.

6. **Attach `RecordManager` nodes to all vector stores** with `cleanup: incremental` to prevent duplicate embeddings on re-upsert.

7. **Set `maxIterations` on all Worker/Agent nodes** to prevent runaway LLM loops that exhaust API quotas.

8. **Enable Prometheus metrics** (`ENABLE_METRICS=true`) and connect to a Grafana dashboard to establish a performance baseline.

### Medium Priority

9. **Implement log rotation** for the Flowise log directory. Without rotation, logs grow unbounded and can fill the VPS disk.

10. **Export agentflow JSON to Git** after every structural change. This provides version history and a recovery path without relying solely on the database.

11. **Tune memory strategy** from `allMessages` to `windowSize` (e.g., last 10 messages) or `summaryBuffer` for long-running conversations to reduce per-request token costs.

---

## 4. Next Steps (Prioritized)

```
Priority 1 — Stability & Security
  [ ] Migrate database to PostgreSQL
  [ ] Verify and enforce sessionId per Flutter user
  [ ] Enable and test API key authentication on all endpoints
  [ ] Back up FLOWISE_SECRETKEY_OVERWRITE to a secrets manager
  [ ] Audit CORS_ORIGINS and IFRAME_ORIGINS

Priority 2 — Observability
  [ ] Enable ENABLE_METRICS=true, configure Prometheus scrape
  [ ] Set up Grafana dashboard for http_request_duration_ms and checkouts_total
  [ ] Configure log rotation (logrotate on LOG_PATH)
  [ ] Verify /api/v1/ping is monitored by an uptime tool (e.g., UptimeRobot)

Priority 3 — Knowledge Base Integrity
  [ ] Attach RecordManager to all vector store nodes (cleanup: incremental)
  [ ] Verify embedding model consistency between upsert and query
  [ ] Establish a document refresh schedule using /api/v1/document-store/refresh/:id

Priority 4 — Performance
  [ ] Evaluate MODE=queue deployment with Redis
  [ ] Switch memory strategy from allMessages to windowSize or summaryBuffer
  [ ] Enable response streaming in Flutter HTTP client
  [ ] Enable RedisCache for repeated LLM queries

Priority 5 — Agentflow Quality
  [ ] Export all agentflows to JSON and commit to Git
  [ ] Set maxIterations on all Worker/Agent nodes
  [ ] Add moderation node at graph entry point
  [ ] Build evaluation dataset and run via Flowise evaluation framework
  [ ] Assess migration from legacy Supervisor/Worker to Agentflow V2
``` [22](#0-21) [23](#0-22) [24](#0-23)

### Citations

**File:** packages/server/src/utils/buildAgentflow.ts (L33-78)
```typescript
    INodeDirectedGraph,
    StartInputType
} from '../Interface'
import {
    RUNTIME_MESSAGES_LENGTH_VAR_PREFIX,
    CHAT_HISTORY_VAR_PREFIX,
    databaseEntities,
    FILE_ATTACHMENT_PREFIX,
    getAppVersion,
    getGlobalVariable,
    getStartingNode,
    getTelemetryFlowObj,
    QUESTION_VAR_PREFIX,
    CURRENT_DATE_TIME_VAR_PREFIX,
    _removeCredentialId,
    validateHistorySchema,
    LOOP_COUNT_VAR_PREFIX
} from '.'
import { ChatFlow } from '../database/entities/ChatFlow'
import { Variable } from '../database/entities/Variable'
import { replaceInputsWithConfig, constructGraphs, getAPIOverrideConfig } from '../utils'
import logger from './logger'
import { getErrorMessage } from '../errors/utils'
import { Execution } from '../database/entities/Execution'
import { utilAddChatMessage } from './addChatMesage'
import { CachePool } from '../CachePool'
import { ChatMessage } from '../database/entities/ChatMessage'
import { Telemetry } from './telemetry'
import { getWorkspaceSearchOptions } from '../enterprise/utils/ControllerServiceUtils'
import { UsageCacheManager } from '../UsageCacheManager'
import { generateTTSForResponseStream, shouldAutoPlayTTS } from './buildChatflow'
import { InternalFlowiseError } from '../errors/internalFlowiseError'
import { StatusCodes } from 'http-status-codes'

interface IWaitingNode {
    nodeId: string
    receivedInputs: Map<string, any>
    expectedInputs: Set<string>
    isConditional: boolean
    conditionalGroups: Map<string, string[]>
}

interface INodeQueue {
    nodeId: string
    data: any
    inputs: Record<string, any>
```

**File:** packages/server/src/utils/buildAgentflow.ts (L97-102)
```typescript
interface IAgentFlowRuntime {
    state?: ICommonObject
    chatHistory?: IMessage[]
    form?: Record<string, any>
    webhook?: Record<string, any>
}
```

**File:** packages/server/src/utils/buildAgentflow.ts (L174-174)
```typescript
const MAX_LOOP_COUNT = process.env.MAX_LOOP_COUNT ? parseInt(process.env.MAX_LOOP_COUNT) : 10
```

**File:** packages/server/src/utils/buildAgentflow.ts (L275-388)
```typescript
        if (typeof value !== 'string') return value

        // Convert legacy HTML content to markdown, preserving any markdown syntax within.
        // Legacy content from old getHTML() starts with a TipTap block tag (e.g. <p>text</p>).
        // Anchor with ^ to avoid matching intentional HTML/XML tags in user prompts
        // (e.g. <instruction><div>...</div></instruction>).
        if (/^\s*<(?:p|div|h[1-6]|ul|ol|blockquote|pre|table)\b/i.test(value)) {
            const turndownService = new TurndownService()
            // Disable escaping so markdown characters (e.g. ###, -, *) inside HTML are preserved as-is
            turndownService.escape = (str: string) => str
            value = turndownService.turndown(value)
        }

        const matches = value.match(/{{(.*?)}}/g)

        if (!matches) return value

        let resolvedValue = value
        for (const match of matches) {
            // Remove {{ }} and trim whitespace
            const reference = match.replace(/[{}]/g, '').trim()
            const variableFullPath = reference

            if (variableFullPath === QUESTION_VAR_PREFIX) {
                resolvedValue = resolvedValue.replace(match, question)
                resolvedValue = uploadedFilesContent ? `${uploadedFilesContent}\n\n${resolvedValue}` : resolvedValue
            }

            if (variableFullPath.startsWith('$form.')) {
                const variableValue = get(form, variableFullPath.replace('$form.', ''))
                if (variableValue != null) {
                    // For arrays and objects, stringify them to prevent toString() conversion issues
                    const formattedValue =
                        Array.isArray(variableValue) || (typeof variableValue === 'object' && variableValue !== null)
                            ? JSON.stringify(variableValue)
                            : variableValue
                    resolvedValue = resolvedValue.replace(match, formattedValue)
                }
            }

            if (variableFullPath.startsWith('$webhook.')) {
                const variableValue = get(webhook, variableFullPath.replace('$webhook.', ''))
                if (variableValue != null) {
                    const formattedValue =
                        Array.isArray(variableValue) || (typeof variableValue === 'object' && variableValue !== null)
                            ? JSON.stringify(variableValue)
                            : variableValue
                    resolvedValue = resolvedValue.replace(match, formattedValue)
                }
            }

            if (variableFullPath === FILE_ATTACHMENT_PREFIX) {
                resolvedValue = resolvedValue.replace(match, uploadedFilesContent)
            }

            if (variableFullPath === CHAT_HISTORY_VAR_PREFIX) {
                resolvedValue = resolvedValue.replace(match, convertChatHistoryToText(chatHistory))
            }

            if (variableFullPath === RUNTIME_MESSAGES_LENGTH_VAR_PREFIX) {
                resolvedValue = resolvedValue.replace(match, flowConfig?.runtimeChatHistoryLength ?? 0)
            }

            if (variableFullPath === LOOP_COUNT_VAR_PREFIX) {
                // Get the current loop count from the most recent loopAgentflow node execution
                let currentLoopCount = 0
                if (loopCounts && agentFlowExecutedData) {
                    // Find the most recent loopAgentflow node execution to get its loop count
                    const loopNodes = [...agentFlowExecutedData].reverse().filter((data) => data.data?.name === 'loopAgentflow')
                    if (loopNodes.length > 0) {
                        const latestLoopNode = loopNodes[0]
                        currentLoopCount = loopCounts.get(latestLoopNode.nodeId) || 0
                    }
                }
                resolvedValue = resolvedValue.replace(match, currentLoopCount.toString())
            }

            if (variableFullPath === CURRENT_DATE_TIME_VAR_PREFIX) {
                resolvedValue = resolvedValue.replace(match, new Date().toISOString())
            }

            if (variableFullPath.startsWith('$iteration')) {
                if (iterationContext && iterationContext.value) {
                    if (variableFullPath === '$iteration') {
                        // If it's exactly $iteration, stringify the entire value
                        const formattedValue =
                            typeof iterationContext.value === 'object' ? JSON.stringify(iterationContext.value) : iterationContext.value
                        resolvedValue = resolvedValue.replace(match, formattedValue)
                    } else if (typeof iterationContext.value === 'string') {
                        resolvedValue = resolvedValue.replace(match, iterationContext?.value)
                    } else if (typeof iterationContext.value === 'object') {
                        const iterationValue = get(iterationContext.value, variableFullPath.replace('$iteration.', ''))
                        // For arrays and objects, stringify them to prevent toString() conversion issues
                        const formattedValue =
                            Array.isArray(iterationValue) || (typeof iterationValue === 'object' && iterationValue !== null)
                                ? JSON.stringify(iterationValue)
                                : iterationValue
                        resolvedValue = resolvedValue.replace(match, formattedValue)
                    }
                }
            }

            if (variableFullPath.startsWith('$vars.')) {
                const vars = await getGlobalVariable(flowConfig, availableVariables, variableOverrides)
                const variableValue = get(vars, variableFullPath.replace('$vars.', ''))
                if (variableValue != null) {
                    // For arrays and objects, stringify them to prevent toString() conversion issues
                    const formattedValue =
                        Array.isArray(variableValue) || (typeof variableValue === 'object' && variableValue !== null)
                            ? JSON.stringify(variableValue)
                            : variableValue
                    resolvedValue = resolvedValue.replace(match, formattedValue)
                }
            }
```

**File:** packages/server/src/utils/buildAgentflow.ts (L855-856)
```typescript
    nodeExecutionQueue,
    waitingNodes,
```

**File:** packages/server/src/utils/buildAgentGraph.ts (L115-118)
```typescript
        const workerNodes = initializedNodes.filter((node) => node.data.name === 'worker')
        const supervisorNodes = initializedNodes.filter((node) => node.data.name === 'supervisor')
        const seqAgentNodes = initializedNodes.filter((node) => node.data.category === 'Sequential Agents')

```

**File:** packages/components/nodes/multiagents/Worker/Worker.ts (L6-7)
```typescript
import { formatToOpenAIToolMessages } from '@langchain/classic/agents/format_scratchpad/openai_tools'
import { type ToolsAgentStep } from '@langchain/classic/agents/openai/output_parser'
```

**File:** packages/components/nodes/multiagents/Worker/Worker.ts (L96-99)
```typescript
        if (!workerLabel) throw new Error('Worker name is required!')
        const workerName = workerLabel.toLowerCase().replace(/\s/g, '_').trim()

        if (!workerPrompt) throw new Error('Worker prompt is required!')
```

**File:** packages/server/src/services/documentstore/index.ts (L1280-1290)
```typescript
        // Step 1: Save configuration based on isStrictSave mode
        const entity = await saveVectorStoreConfig(appDataSource, data, isStrictSave, workspaceId)

        // Step 2: Mark as UPSERTING before starting the operation
        entity.status = DocumentStoreStatus.UPSERTING
        await appDataSource.getRepository(DocumentStore).save(entity)

        // Step 3: Perform the actual vector store upsert
        // Note: Configuration already saved above, worker thread just retrieves and uses it
        const indexResult = await _insertIntoVectorStoreWorkerThread(appDataSource, componentNodes, telemetry, data, orgId, workspaceId)
        return indexResult
```

**File:** packages/components/nodes/vectorstores/Pinecone/Pinecone.ts (L164-179)
```typescript
            try {
                if (recordManager) {
                    const vectorStore = (await PineconeStore.fromExistingIndex(embeddings, obj)) as unknown as VectorStore
                    await recordManager.createSchema()
                    const res = await index({
                        docsSource: finalDocs,
                        recordManager,
                        vectorStore,
                        options: {
                            cleanup: recordManager?.cleanup,
                            sourceIdKey: recordManager?.sourceIdKey ?? 'source',
                            vectorStoreName: pineconeNamespace
                        }
                    })

                    return res
```

**File:** packages/server/src/routes/documentstore/index.ts (L10-10)
```typescript
router.post(['/refresh/', '/refresh/:id'], documentStoreController.refreshDocStoreMiddleware)
```

**File:** docker/docker-compose-queue-prebuilt.yml (L26-34)
```yaml
            - DATABASE_PATH=${DATABASE_PATH:-/root/.flowise}
            - DATABASE_TYPE=${DATABASE_TYPE}
            - DATABASE_PORT=${DATABASE_PORT}
            - DATABASE_HOST=${DATABASE_HOST}
            - DATABASE_NAME=${DATABASE_NAME}
            - DATABASE_USER=${DATABASE_USER}
            - DATABASE_PASSWORD=${DATABASE_PASSWORD}
            - DATABASE_SSL=${DATABASE_SSL}
            - DATABASE_SSL_KEY_BASE64=${DATABASE_SSL_KEY_BASE64}
```

**File:** docker/docker-compose-queue-prebuilt.yml (L37-43)
```yaml
            - SECRETKEY_STORAGE_TYPE=${SECRETKEY_STORAGE_TYPE}
            - SECRETKEY_PATH=${SECRETKEY_PATH}
            - FLOWISE_SECRETKEY_OVERWRITE=${FLOWISE_SECRETKEY_OVERWRITE}
            - SECRETKEY_AWS_ACCESS_KEY=${SECRETKEY_AWS_ACCESS_KEY}
            - SECRETKEY_AWS_SECRET_KEY=${SECRETKEY_AWS_SECRET_KEY}
            - SECRETKEY_AWS_REGION=${SECRETKEY_AWS_REGION}
            - SECRETKEY_AWS_NAME=${SECRETKEY_AWS_NAME}
```

**File:** docker/docker-compose-queue-prebuilt.yml (L135-142)
```yaml
            # --- Queue Configuration (Main Instance) ---
            - MODE=${MODE:-queue}
            - QUEUE_NAME=${QUEUE_NAME:-flowise-queue}
            - QUEUE_REDIS_EVENT_STREAM_MAX_LEN=${QUEUE_REDIS_EVENT_STREAM_MAX_LEN}
            - WORKER_CONCURRENCY=${WORKER_CONCURRENCY}
            - REMOVE_ON_AGE=${REMOVE_ON_AGE}
            - REMOVE_ON_COUNT=${REMOVE_ON_COUNT}
            - REDIS_URL=${REDIS_URL:-redis://redis:6379}
```

**File:** docker/docker-compose-queue-prebuilt.yml (L154-163)
```yaml
            # SECURITY
            - CUSTOM_MCP_SECURITY_CHECK=${CUSTOM_MCP_SECURITY_CHECK}
            - CUSTOM_MCP_PROTOCOL=${CUSTOM_MCP_PROTOCOL}
            - CUSTOM_MCP_ALLOWED_ENV_VARS=${CUSTOM_MCP_ALLOWED_ENV_VARS}
            - HTTP_DENY_LIST=${HTTP_DENY_LIST}
            - HTTP_SECURITY_CHECK=${HTTP_SECURITY_CHECK}
            - PATH_TRAVERSAL_SAFETY=${PATH_TRAVERSAL_SAFETY}
            - TRUST_PROXY=${TRUST_PROXY}
            - OAUTH2_SECURITY_CHECK=${OAUTH2_SECURITY_CHECK}
            - OAUTH2_ALLOWED_TOKEN_DOMAINS=${OAUTH2_ALLOWED_TOKEN_DOMAINS}
```

**File:** docker/docker-compose-queue-prebuilt.yml (L167-172)
```yaml
        healthcheck:
            test: ['CMD', 'curl', '-f', 'http://localhost:${PORT:-3000}/api/v1/ping']
            interval: 10s
            timeout: 5s
            retries: 5
            start_period: 30s
```

**File:** docker/docker-compose-queue-prebuilt.yml (L320-325)
```yaml
        healthcheck:
            test: ['CMD', 'curl', '-f', 'http://localhost:${WORKER_PORT:-5566}/healthz']
            interval: 10s
            timeout: 5s
            retries: 5
            start_period: 30s
```

**File:** packages/server/src/enterprise/middleware/prometheus/index.ts (L10-47)
```typescript
    const predictionsTotal = new promClient.Counter({
        name: 'checkouts_total',
        help: 'Total number of checkouts',
        labelNames: ['payment_method']
    })

    const requestCounter = new Counter({
        name: 'http_requests_total',
        help: 'Total number of HTTP requests',
        labelNames: ['method', 'path', 'status']
    })

    app.use('/api/v1/prediction', async (req, res) => {
        res.on('finish', async () => {
            requestCounter.labels(req?.method, req?.path, res.statusCode.toString()).inc()
            predictionsTotal.labels('success').inc()
        })
    })

    // enable default metrics like CPU usage, memory usage, etc.
    promClient.collectDefaultMetrics({ register })
    // Add our custom metric to the registry
    register.registerMetric(requestCounter)
    register.registerMetric(predictionsTotal)

    // Add Prometheus middleware to the app
    app.use('/api/v1/metrics', async (req, res) => {
        res.set('Content-Type', register.contentType)
        const currentMetrics = await register.metrics()
        res.send(currentMetrics)
    })

    const httpRequestDurationMicroseconds = new promClient.Histogram({
        name: 'http_request_duration_ms',
        help: 'Duration of HTTP requests in ms',
        labelNames: ['method', 'route', 'code'],
        buckets: [1, 5, 15, 50, 100, 200, 300, 400, 500] // buckets for response time from 0.1ms to 500ms
    })
```

**File:** packages/components/nodes/memory/BufferWindowMemory/BufferWindowMemory.ts (L1-1)
```typescript
import {
```

**File:** packages/components/package.json (L69-69)
```json
        "@langchain/core": "1.1.20",
```

**File:** packages/components/nodes/agentflow/ConditionAgent/ConditionAgent.ts (L1-1)
```typescript
import { AnalyticHandler } from '../../../src/handler'
```

**File:** packages/api-documentation/src/yml/swagger.yml (L1194-1210)
```yaml
    /ping:
        get:
            tags:
                - ping
            summary: Ping the server
            description: Ping the server to check if it is running
            operationId: pingServer
            responses:
                '200':
                    description: Server is running
                    content:
                        text/plain:
                            schema:
                                type: string
                                example: pong
                '500':
                    description: Internal server error
```

**File:** packages/components/nodes/memory/ConversationSummaryBufferMemory/ConversationSummaryBufferMemory.ts (L1-1)
```typescript
import {
```
