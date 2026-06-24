# **Complete Guide: Flowise Multi-Agent Architecture (Supervisor \+ Worker)**

---

## **1\. Overview**

This guide summarizes the architecture discussed in our conversation and provides a detailed technical reference for building a **mobile hardware specialist chatbot** using Flowise's Multi-Agent system.

The core idea: instead of one large agent trying to know everything, you split responsibilities across specialized agents coordinated by a central decision-maker.

---

## **2\. Architecture Diagram**

graph TD

    "User Input" \--\> "Supervisor Agent"

    "Supervisor Agent" \-- "Route to Samsung Worker" \--\> "Worker: Samsung"

    "Supervisor Agent" \-- "Route to Xiaomi Worker" \--\> "Worker: Xiaomi"

    "Supervisor Agent" \-- "Route to Apple Worker" \--\> "Worker: Apple"

    "Supervisor Agent" \-- "Route to Sony Worker" \--\> "Worker: Sony"

    "Worker: Samsung" \--\> "KB: Samsung Docs"

    "Worker: Samsung" \--\> "Web Search Tool"

    "Worker: Xiaomi" \--\> "KB: Xiaomi Docs"

    "Worker: Xiaomi" \--\> "Web Search Tool"

    "Worker: Apple" \--\> "KB: Apple Docs"

    "Worker: Apple" \--\> "Web Search Tool"

    "Worker: Sony" \--\> "KB: Sony Docs"

    "Worker: Sony" \--\> "Web Search Tool"

    "Worker: Samsung" \-- "Result \+ Status" \--\> "Supervisor Agent"

    "Worker: Xiaomi" \-- "Result \+ Status" \--\> "Supervisor Agent"

    "Worker: Apple" \-- "Result \+ Status" \--\> "Supervisor Agent"

    "Worker: Sony" \-- "Result \+ Status" \--\> "Supervisor Agent"

    "Supervisor Agent" \-- "FINISH" \--\> "Final Answer"

---

## **3\. The Supervisor Node — Deep Dive**

### **3.1 What the Supervisor Does**

The Supervisor is the **decision-making brain** of the system. It does not answer questions directly. Its only job is:

1. Read the user's input  
2. Decide which Worker should act next  
3. Send specific instructions to that Worker  
4. Receive the Worker's result  
5. Decide if another Worker is needed, or if it's time to `FINISH`  
6. (Optionally) Summarize all Worker results into a final answer

### **3.2 How It Works Internally**

The Supervisor uses a special **routing tool** called `route`. When invoked, the LLM must call this tool with three fields: [1](#12-0)

| Field | Type | Description |
| :---- | :---- | :---- |
| `reasoning` | string | Why the Supervisor chose this Worker |
| `next` | enum | Which Worker to call next, or `FINISH` |
| `instructions` | string | Specific sub-task instructions for the chosen Worker |

After each Worker finishes, the Supervisor is called again with the full conversation history (including the Worker's result), and it decides the next step.

### **3.3 Supervisor Configuration Parameters [2](#12-1)**

| Parameter | Required | Description |
| :---- | :---- | :---- |
| `Supervisor Name` | Yes | Internal name (converted to lowercase, spaces → underscores) |
| `Supervisor Prompt` | Yes | System prompt — **must contain `{team_members}`** |
| `Tool Calling Chat Model` | Yes | LLM for routing decisions — must support function calling |
| `Agent Memory` | No | Checkpoint saver to persist state across sessions |
| `Summarization` | No | If enabled, Supervisor produces a final summary of all Worker outputs |
| `Recursion Limit` | No | Max number of Supervisor↔Worker cycles (default: 100\) |
| `Input Moderation` | No | Filter harmful inputs before they reach the LLM |

### **3.4 The `{team_members}` Placeholder — Critical Rule**

The Supervisor Prompt **must** contain `{team_members}`. Flowise replaces this at runtime with the actual names of all connected Workers: [3](#12-2)

If you have Workers named `samsung_worker`, `xiaomi_worker`, `apple_worker`, the placeholder becomes:

samsung\_worker, xiaomi\_worker, apple\_worker

### **3.5 Execution Flow (Sequential, Not Parallel)**

Workers execute **one at a time**. The Supervisor picks one Worker per cycle: [4](#12-3)

For a comparison question like "Is Samsung S24 camera better than Xiaomi 14?":

Cycle 1: Supervisor → samsung\_worker (search Samsung KB)

Cycle 2: Supervisor → xiaomi\_worker  (search Xiaomi KB)

Cycle 3: Supervisor → FINISH         (both results available, generate comparison)

### **3.6 Summarization Mode**

When `Summarization` is enabled, the routing tool schema gains an extra field: [5](#12-4)

This causes the Supervisor to produce a clean summary of the entire multi-step conversation as the final output, rather than returning the raw last Worker message.

---

## **4\. The Worker Node — Deep Dive**

### **4.1 What a Worker Does**

Each Worker is a **specialized agent** that:

1. Receives specific instructions from the Supervisor  
2. Uses its assigned Tools (Knowledge Base retriever, Web Search, etc.)  
3. Returns its result back to the Supervisor

### **4.2 Worker Configuration Parameters [6](#12-5)**

| Parameter | Required | Description |
| :---- | :---- | :---- |
| `Worker Name` | Yes | Unique name — this is what the Supervisor uses to route |
| `Worker Prompt` | Yes | System prompt defining the Worker's specialty |
| `Tools` | No | List of tools: Retriever Tool, Web Search, Calculator, etc. |
| `Supervisor` | Yes | Must be connected to a Supervisor node |
| `Tool Calling Chat Model` | No | If not set, uses the Supervisor's model |
| `Format Prompt Values` | No | Dynamic variable injection into the prompt |
| `Max Iterations` | No | Limits tool-use loops within a single Worker invocation |

### **4.3 Worker Prompt Auto-Appended Text**

Whatever you write in the Worker Prompt, Flowise **automatically appends** this text: [7](#12-6)

Work autonomously according to your specialty, using the tools available to you.

Do not ask for clarification.

Your other team members (and other teams) will collaborate with you with their own specialties.

You are chosen for a reason\! You are one of the following team members: {team\_members}.

This means Workers are instructed to act independently without asking follow-up questions.

### **4.4 Model Fallback [8](#12-7)**

const llm \= model || (supervisor.llm as BaseChatModel)

If no model is assigned to a Worker, it inherits the Supervisor's model. This means you can use an expensive model only for the Supervisor and a cheaper model for Workers.

---

## **5\. Knowledge Base Setup for Each Brand**

Each Worker should be connected to a **Retriever Tool** that points to its brand-specific Document Store:

Worker: Samsung

  └── Retriever Tool

        └── Document Store: "Samsung Hardware KB"

              └── Vector Store (Pinecone / Chroma / etc.)

                    └── Chunks from Samsung manuals, specs, repair guides

The Document Store must be **upserted** (indexed) before the chatbot can use it.

---

## **6\. Web Search Integration Per Worker**

Each Worker can have both a Retriever Tool and a Web Search Tool. The Worker's LLM decides which to use based on the question:

- "What is the battery capacity of Galaxy S24?" → uses Knowledge Base (static spec)  
- "What is the current price of Galaxy S24 in Iran?" → uses Web Search (real-time data)

Recommended web search tools in Flowise: `TavilyAPI`, `BraveSearchAPI`, `SerpAPI`.

---

## **7\. LLM Strategy: Supervisor vs. Workers**

| Role | Recommended Model | Reason |
| :---- | :---- | :---- |
| **Supervisor** | Claude Sonnet 4.5 / GPT-4o | Strong reasoning for routing decisions and final synthesis |
| **Workers** | DeepSeek V3 / GPT-4o-mini | Cost-effective for retrieval \+ answer generation |

**Requirement**: Both Supervisor and Worker models must support **function calling** (tool use).

---

## **8\. Supervisor Prompt — Full Explanation**

### **8.1 What the Prompt Must Contain**

The Supervisor Prompt is a **system message** that tells the LLM:

- What its role is  
- Who the Workers are (via `{team_members}`)  
- How to decide which Worker to call  
- When to stop (`FINISH`)

### **8.2 Default Prompt (from Flowise source) [9](#12-8)**

You are a supervisor tasked with managing a conversation between the following workers: {team\_members}.

Given the following user request, respond with the worker to act next.

Each worker will perform a task and respond with their results and status.

When finished, respond with FINISH.

Select strategically to minimize the number of steps taken.

This default is minimal. For a production chatbot, you need a much more detailed prompt.

### **8.3 What Makes a Good Supervisor Prompt**

A good Supervisor Prompt should define:

1. **Domain context** — what the overall system is about  
2. **Worker descriptions** — what each Worker specializes in  
3. **Routing rules** — when to call which Worker  
4. **Comparison handling** — what to do when the question involves multiple brands  
5. **Fallback behavior** — what to do when no Worker matches  
6. **Output format** — how to present the final answer

---

## **9\. Supervisor Prompt Template**

You are a supervisor managing a team of mobile hardware specialist agents.

Your team members are: {team\_members}.

\#\# Team Member Specializations

\- samsung\_worker: Expert in Samsung Galaxy smartphones (hardware specs, common issues, repair guides, battery, camera, display, chipset)

\- xiaomi\_worker: Expert in Xiaomi smartphones (Redmi, POCO, Mi series — hardware specs, MIUI hardware behavior, common issues)

\- apple\_worker: Expert in Apple iPhone hardware (all iPhone models, hardware specs, Face ID, cameras, chips, repair)

\- sony\_worker: Expert in Sony Xperia smartphones (hardware specs, display technology, camera system)

\- general\_worker: Handles questions not specific to any single brand, or general mobile hardware concepts

\#\# Routing Rules

1\. If the user asks about a specific brand, route to that brand's worker.

2\. If the user asks a comparison question between two brands (e.g., "Samsung vs Xiaomi"), route to BOTH workers sequentially — first one brand, then the other — before finishing.

3\. If the question is about general mobile hardware concepts (e.g., "how does OLED work?"), route to general\_worker.

4\. If no worker is relevant, respond with FINISH and answer directly from your own knowledge.

\#\# Execution Strategy

\- Minimize the number of steps. Do not call a worker if its result is not needed.

\- After each worker responds, evaluate whether you have enough information to answer the user.

\- If yes, respond with FINISH.

\- If no, route to the next relevant worker with specific instructions.

\#\# Output Quality

\- When finishing, synthesize all worker results into a clear, structured final answer.

\- For comparison questions, present results in a side-by-side format.

\- Always cite which worker provided which information.

Given the conversation above, who should act next? Or should we FINISH?

Select one of: {team\_members}, FINISH.

---

## **10\. Worker Prompt Template (Per Brand)**

You are a mobile hardware specialist for \[BRAND NAME\] smartphones.

Your expertise covers:

- Hardware specifications (chipset, RAM, storage, battery, display, cameras)  
    
- Common hardware issues and their causes  
    
- Repair and troubleshooting guidance  
    
- Comparison of different models within the \[BRAND\] lineup

## **Instructions**

- Search the knowledge base first for technical specifications and known issues.  
    
- If the question requires real-time data (current price, latest news, availability), use the web search tool.  
    
- Provide precise, technical answers. Include model numbers and specifications when relevant.  
    
- If you cannot find the answer in your tools, clearly state that the information is not available.  
    
- Do not speculate. Only provide information you can verify from your tools.

---

## **11\. Complete Node Setup Checklist**

For each brand (repeat for Samsung, Xiaomi, Apple, Sony, etc.):

  \[ \] Create Document Store in Flowise

  \[ \] Upload brand-specific documents (manuals, spec sheets, repair guides)

  \[ \] Configure chunking (recommended: chunk size 1000, overlap 200\)

  \[ \] Upsert documents into Vector Store

  \[ \] Create Worker node

        \[ \] Set Worker Name (e.g., "Samsung Worker")

        \[ \] Write Worker Prompt (use template above)

        \[ \] Connect Retriever Tool → Document Store

        \[ \] Connect Web Search Tool (Tavily or Brave)

        \[ \] Optionally assign a dedicated LLM model

        \[ \] Connect to Supervisor node

For the Supervisor:

  \[ \] Create Supervisor node

  \[ \] Set Supervisor Name

  \[ \] Write Supervisor Prompt (use template above, list all workers)

  \[ \] Connect a strong LLM (Claude Sonnet / GPT-4o)

  \[ \] Enable Summarization (recommended for cleaner final output)

  \[ \] Set Recursion Limit (recommended: 20-30 for 10 workers)

  \[ \] Optionally connect Agent Memory for session persistence

---

## **12\. Key Technical Constraints Summary**

| Constraint | Detail |
| :---- | :---- |
| Supervisor Prompt must contain | `{team_members}` (required, throws error if missing) |
| Worker execution | Sequential only — no parallel execution |
| Worker model | Optional — falls back to Supervisor's model if not set |
| Both models must support | Function calling / tool use |
| Recursion limit default | 100 (lower it to 20-30 for production) |
| Worker name format | Lowercased, spaces replaced with underscores internally |

### **Citations**

**File:** packages/components/nodes/multiagents/Supervisor/Supervisor.ts (L17-21)

const sysPrompt \= \`You are a supervisor tasked with managing a conversation between the following workers: {team\_members}.

Given the following user request, respond with the worker to act next.

Each worker will perform a task and respond with their results and status.

When finished, respond with FINISH.

Select strategically to minimize the number of steps taken.\`

**File:** packages/components/nodes/multiagents/Supervisor/Supervisor.ts (L41-103)

    constructor() {

        this.label \= 'Supervisor'

        this.name \= 'supervisor'

        this.version \= 3.0

        this.type \= 'Supervisor'

        this.icon \= 'supervisor.svg'

        this.category \= 'Multi Agents'

        this.baseClasses \= \[this.type\]

        this.inputs \= \[

            {

                label: 'Supervisor Name',

                name: 'supervisorName',

                type: 'string',

                placeholder: 'Supervisor',

                default: 'Supervisor'

            },

            {

                label: 'Supervisor Prompt',

                name: 'supervisorPrompt',

                type: 'string',

                description: 'Prompt must contains {team\_members}',

                rows: 4,

                default: sysPrompt,

                additionalParams: true

            },

            {

                label: 'Tool Calling Chat Model',

                name: 'model',

                type: 'BaseChatModel',

                description: \`Only compatible with models that are capable of function calling: ChatOpenAI, ChatMistral, ChatAnthropic, ChatGoogleGenerativeAI, GroqChat. Best result with GPT-4 model\`

            },

            {

                label: 'Agent Memory',

                name: 'agentMemory',

                type: 'BaseCheckpointSaver',

                description: 'Save the state of the agent',

                optional: true

            },

            {

                label: 'Summarization',

                name: 'summarization',

                type: 'boolean',

                description: 'Return final output as a summarization of the conversation',

                optional: true,

                additionalParams: true

            },

            {

                label: 'Recursion Limit',

                name: 'recursionLimit',

                type: 'number',

                description: 'Maximum number of times a call can recurse. If not provided, defaults to 100.',

                default: 100,

                additionalParams: true

            },

            {

                label: 'Input Moderation',

                description: 'Detect text that could generate harmful output and prevent it from being sent to the language model',

                name: 'inputModeration',

                type: 'Moderation',

                optional: true,

                list: true

            }

        \]

**File:** packages/components/nodes/multiagents/Supervisor/Supervisor.ts (L130-134)

            systemPrompt \= systemPrompt.replaceAll('{team\_members}', members.join(', '))

            let userPrompt \= \`Given the conversation above, who should act next? Or should we FINISH? Select one of: ${memberOptions.join(

                ', '

            )}\`

**File:** packages/components/nodes/multiagents/Supervisor/Supervisor.ts (L136-142)

            const tool \= new RouteTool({

                schema: z.object({

                    reasoning: z.string(),

                    next: z.enum(\['FINISH', ...members\]),

                    instructions: z.string().describe('The specific instructions of the sub-task the next role should accomplish.')

                })

            })

**File:** packages/components/nodes/multiagents/Supervisor/Supervisor.ts (L385-392)

            const tool \= new RouteTool({

                schema: z.object({

                    reasoning: z.string(),

                    next: z.enum(\['FINISH', ...members\]),

                    instructions: z.string().describe('The specific instructions of the sub-task the next role should accomplish.'),

                    summarization: z.string().optional().describe('Summarization of the conversation')

                })

            })

**File:** packages/server/src/utils/buildAgentGraph.ts (L552-566)

            for (const worker of supervisorResult.workers) {

                //@ts-ignore

                workflowGraph.addEdge(worker, supervisorResult.name)

            }

            let conditionalEdges: { \[key: string\]: string } \= {}

            for (let i \= 0; i \< supervisorResult.workers.length; i++) {

                conditionalEdges\[supervisorResult.workers\[i\]\] \= supervisorResult.workers\[i\]

            }

            //@ts-ignore

            workflowGraph.addConditionalEdges(supervisorResult.name, (x: ITeamState) \=\> x.next, {

                ...conditionalEdges,

                FINISH: END

            })

**File:** packages/components/nodes/multiagents/Worker/Worker.ts (L36-83)

        this.inputs \= \[

            {

                label: 'Worker Name',

                name: 'workerName',

                type: 'string',

                placeholder: 'Worker'

            },

            {

                label: 'Worker Prompt',

                name: 'workerPrompt',

                type: 'string',

                rows: 4,

                default: examplePrompt

            },

            {

                label: 'Tools',

                name: 'tools',

                type: 'Tool',

                list: true,

                optional: true

            },

            {

                label: 'Supervisor',

                name: 'supervisor',

                type: 'Supervisor'

            },

            {

                label: 'Tool Calling Chat Model',

                name: 'model',

                type: 'BaseChatModel',

                optional: true,

                description: \`Only compatible with models that are capable of function calling: ChatOpenAI, ChatMistral, ChatAnthropic, ChatGoogleGenerativeAI, ChatVertexAI, GroqChat. If not specified, supervisor's model will be used\`

            },

            {

                label: 'Format Prompt Values',

                name: 'promptValues',

                type: 'json',

                optional: true,

                acceptVariable: true,

                list: true

            },

            {

                label: 'Max Iterations',

                name: 'maxIterations',

                type: 'number',

                optional: true

            }

        \]

**File:** packages/components/nodes/multiagents/Worker/Worker.ts (L111-111)

        const llm \= model || (supervisor.llm as BaseChatModel)

**File:** packages/components/nodes/multiagents/Worker/Worker.ts (L171-17  
