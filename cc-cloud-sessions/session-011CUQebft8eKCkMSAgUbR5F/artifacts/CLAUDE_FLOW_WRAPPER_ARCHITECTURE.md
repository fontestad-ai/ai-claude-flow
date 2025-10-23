# How claude-flow Wraps Claude Code: Complete Architectural Analysis
**Critical Question Resolution**: If claude-flow wraps claude-code, how does it handle sessions and context?

---

## EXECUTIVE SUMMARY

Claude-Flow acts as a **sophisticated middleware orchestration layer** that:
1. **Directly wraps the Anthropic SDK** (not Claude Code SDK)
2. **Creates a parallel session management system** outside of Claude Code
3. **Uses MCP (Model Context Protocol) tools** for coordination across multiple Claude Code instances
4. **Implements cross-session memory persistence** through SQLite, JSON, and in-memory stores
5. **Coordinates multi-agent execution** through pre/post-operation hooks

**Key Insight**: Claude-Flow does NOT directly wrap @anthropic-ai/claude-code. Instead, it:
- Wraps `@anthropic-ai/sdk` directly
- Uses claude-code SDK's **hooks system** for coordination
- Maintains its own session/context management in `.claude/` directory
- Acts as an MCP server that Claude Code connects to

---

## 1. INTEGRATION ARCHITECTURE

### 1.1 The Three-Layer Model

```
┌────────────────────────────────────────────────────────┐
│                    CLAUDE CODE SDK                      │
│              (@anthropic-ai/claude-code)               │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Hook System (9 events):                         │  │
│  │  • PreToolUse / PostToolUse                      │  │
│  │  • SessionStart / SessionEnd                     │  │
│  │  • Notification / UserPromptSubmit               │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
                          ↑
            ┌─────────────┴──────────────┐
            │                            │
┌──────────────────────────────────────────────────────────────────┐
│               CLAUDE-FLOW ORCHESTRATION LAYER                     │
│  (Coordinates agents, manages sessions, persists context)        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Hook Manager │  │ Session Mgr  │  │ Memory Store │          │
│  │ (Pre/Post)   │  │ (Sessions)   │  │ (SQLite/JSON)│          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Agent Mgr    │  │ Task Queue   │  │ Memory State │          │
│  │ (Spawning)   │  │ (Execution)  │  │ (Persistence)│          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                          ↑
         ┌────────────────┴────────────────┐
         │                                 │
┌─────────────────────────┐     ┌──────────────────────┐
│  ANTHROPIC SDK WRAPPER  │     │  MCP SERVER TOOLS    │
│  (ClaudeFlowSDKAdapter) │     │  (87+ tools)         │
│                         │     │  (swarm, neural, etc)│
│  • Direct SDK usage     │     │                      │
│  • Message creation     │     │  Coordinates across  │
│  • Streaming support    │     │  multiple agents     │
│  • Error handling       │     │                      │
└─────────────────────────┘     └──────────────────────┘
         ↑                              ↑
         │                              │
         └──────────────┬───────────────┘
                        │
              ┌─────────▼─────────┐
              │  ANTHROPIC API    │
              │  messages/create  │
              └───────────────────┘
```

### 1.2 File Evidence: SDK Integration Points

**Location**: `/src/sdk/sdk-config.ts`

```typescript
import Anthropic from '@anthropic-ai/sdk';

export class ClaudeFlowSDKAdapter {
  private sdk: Anthropic;
  private config: SDKConfiguration;

  constructor(config: SDKConfiguration = {}) {
    // Initialize Anthropic SDK with configuration
    this.sdk = new Anthropic({
      apiKey: this.config.apiKey,
      baseURL: this.config.baseURL,
      maxRetries: this.config.maxRetries,
      timeout: this.config.timeout,
      defaultHeaders: this.config.defaultHeaders
    });
  }

  async createMessage(params: Anthropic.MessageCreateParams): Promise<Anthropic.Message> {
    const message = await this.sdk.messages.create(params);
    
    // Store in swarm metadata if in swarm mode
    if (this.config.swarmMode && message.id) {
      this.swarmMetadata.set(message.id, {
        timestamp: Date.now(),
        model: params.model,
        tokensUsed: message.usage
      });
    }
    return message;
  }
}
```

**Key Finding**: Claude-Flow uses `@anthropic-ai/sdk` directly, NOT `@anthropic-ai/claude-code`.

---

## 2. SESSION BRIDGING MECHANISM

### 2.1 The Session Architecture

Claude-Flow maintains **THREE independent session systems**:

```
CLAUDE CODE SESSIONS          CLAUDE-FLOW SESSIONS        MEMORY PERSISTENCE
(Claude Code SDK)             (Parallel Management)        (Outside SDK)

┌──────────────────┐         ┌──────────────────┐        ┌──────────────────┐
│ Session ID from  │◄────────│ Session Mapper   │────────│  .claude/        │
│ Claude Code      │         │ (Bridges IDs)    │        │  sessions/       │
└──────────────────┘         └──────────────────┘        └──────────────────┘
         │                             │                          │
         ├─Hook: session-start         ├─Store session metadata   ├─session-*.json
         │                             │                          │
         ├─Hook: pre-task              ├─Track task context       ├─Memory DB
         │                             │                          │
         ├─Hook: post-task             ├─Update state             ├─Checkpoints/
         │                             │                          │
         └─Hook: session-end           └─Persist everything       └─Metrics
```

### 2.2 Session Lifecycle: Code Evidence

**Location**: `/bin/hive-mind/session-manager.js`

```javascript
export class HiveMindSessionManager {
  constructor(hiveMindDir = null) {
    this.hiveMindDir = hiveMindDir || path.join(cwd(), '.hive-mind');
    this.sessionsDir = path.join(this.hiveMindDir, 'sessions');
    this.dbPath = path.join(this.hiveMindDir, 'hive.db');
    this.db = null;
    this.initializationPromise = this.initializeDatabase();
  }

  initializeSchema() {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS sessions (
        id TEXT PRIMARY KEY,
        swarm_id TEXT NOT NULL,
        swarm_name TEXT NOT NULL,
        objective TEXT,
        status TEXT DEFAULT 'active',
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        checkpoint_data TEXT,
        metadata TEXT,
        parent_pid INTEGER,
        child_pids TEXT,
        FOREIGN KEY (swarm_id) REFERENCES swarms(id)
      );
    `);
  }
}
```

**Key Sessions Directory Structure** (from bash output):
```
.claude/
├── sessions/
│   ├── 2025-07-05T12-59-38.185Z-metrics.json
│   ├── 2025-07-06T14-02-56.207Z-metrics.json
│   └── ... (multiple session metrics)
├── checkpoints/
│   └── (139,264 checkpoint files!)
├── config.json
├── settings.json
└── agents/
    └── (agent definitions)
```

### 2.3 Session State Transfer Without SDK Wrapping

Claude-Flow implements **external session state management**:

```javascript
// From: /src/cli/simple-commands/hooks.js
export async function sessionRestoreCommand(subArgs, flags) {
  const sessionId = flags['session-id'] || flags.sessionId;
  const store = await getMemoryStore();
  
  // Restore previous session state from SQLite
  const sessionData = await store.retrieve(`session:${sessionId}`);
  
  return {
    agentStates: sessionData.agents,
    taskQueue: sessionData.tasks,
    memory: sessionData.memory,
    configuration: sessionData.config
  };
}
```

**Critical Distinction**:
- Claude Code SDK has its own session ID and message history
- Claude-Flow maintains a PARALLEL session tracking system
- Sessions are NOT directly related to Claude Code's internal sessions
- Memory/context is stored in SQLite, not in Claude Code's session

---

## 3. CONTEXT MANAGEMENT STRATEGY

### 3.1 How Context Is NOT Managed

Claude-Flow does **NOT**:
- Inherit Claude Code's native session context directly
- Use Claude Code's message history as the source of truth
- Rely on Claude Code's built-in context persistence

### 3.2 How Context IS Managed

Claude-Flow **DOES**:
1. **Intercept operations via hooks** to capture context
2. **Store context externally** in SQLite/JSON
3. **Inject context into new operations** via prompt augmentation
4. **Synchronize context** across multiple agents via MCP tools

#### Context Injection Flow

```
OPERATION STARTS
      ↓
┌─────────────────────────────────────┐
│  Hook: pre-task                     │
│  (intercepted by claude-flow)       │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  Retrieve Context from Storage      │
│  • Task history from SQLite         │
│  • Agent state from memory.db       │
│  • Previous decisions from JSON     │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  Augment Prompt with Context        │
│  system_prompt += context_injection │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  Execute via SDK: createMessage()   │
│  (with augmented prompt)            │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  Hook: post-task                    │
│  (capture result)                   │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  Store Result & Update Context      │
│  • Save to SQLite                   │
│  • Update agent memory              │
│  • Record in checkpoints            │
└─────────────────────────────────────┘
```

**Code Evidence**: `/src/cli/simple-commands/hooks.js`

```javascript
async function preTaskCommand(subArgs, flags) {
  const store = await getMemoryStore();
  const taskData = {
    taskId,
    description,
    agentId,
    status: 'started',
    startedAt: new Date().toISOString(),
  };

  // Store context in SQLite for later retrieval
  await store.store(`task:${taskId}`, taskData, {
    namespace: 'hooks:pre-task',
    metadata: { hookType: 'pre-task', agentId },
  });

  // Later, when agent needs context:
  const previousContext = await store.retrieve(`task:${taskId}`);
  // Inject into system prompt for continuity
}
```

---

## 4. MCP SERVER ROLE

### 4.1 MCP Architecture

Claude-Flow functions as an **MCP Server with 87+ tools**:

```typescript
// From: /bin/hive-mind/mcp-wrapper.js
const MCP_TOOLS = {
  swarm: [
    'swarm_init',      // Initialize swarm
    'agent_spawn',     // Spawn agent (for parallel execution)
    'task_orchestrate', // Coordinate tasks
    // ... 9 more tools
  ],
  neural: [
    'neural_train',    // Train patterns from operations
    'neural_patterns', // Query patterns
    // ... 13 more tools
  ],
  memory: [
    'memory_persist',  // Persist state
    'memory_restore',  // Restore context
    // ... 10 more tools
  ],
  // ... and 5 more categories
};
```

### 4.2 MCP Server Configuration (from Claude Code SDK)

**Location**: `/src/.claude/settings.json`

Claude Code's `settings.json` connects to claude-flow MCP server:

```json
{
  "mcpServers": {
    "claude-flow": {
      "type": "stdio",
      "command": "npx",
      "args": ["claude-flow@latest", "mcp"]
    }
  }
}
```

This allows Claude Code to:
- Call `mcp__claude-flow__agent_spawn` to spawn agents
- Call `mcp__claude-flow__task_orchestrate` to coordinate work
- Call `mcp__claude-flow__memory_persist` to store context
- Call 84+ other MCP tools for orchestration

---

## 5. HOOK INTEGRATION MECHANISM

### 5.1 Hook System: How Claude-Flow Intercepts Operations

Claude Code SDK provides **9 hook events**. Claude-Flow hooks into them:

```
CLAUDE CODE HOOK EVENTS         CLAUDE-FLOW HANDLERS
┌────────────────────┐          ┌──────────────────────┐
│ PreToolUse         │◄─────────│ pre-task hook        │
│ (Before tool use)  │          │ (Prepare context)    │
└────────────────────┘          └──────────────────────┘

┌────────────────────┐          ┌──────────────────────┐
│ PostToolUse        │◄─────────│ post-task hook       │
│ (After tool use)   │          │ (Store results)      │
└────────────────────┘          └──────────────────────┘

┌────────────────────┐          ┌──────────────────────┐
│ SessionStart       │◄─────────│ session-restore hook │
│ (Session created)  │          │ (Initialize context) │
└────────────────────┘          └──────────────────────┘

┌────────────────────┐          ┌──────────────────────┐
│ SessionEnd         │◄─────────│ session-end hook     │
│ (Session closes)   │          │ (Persist everything) │
└────────────────────┘          └──────────────────────┘
```

### 5.2 Hook Registration: Code Evidence

**Location**: `/src/cli/simple-commands/hooks.js`

```javascript
export async function hooksAction(subArgs, flags) {
  const subcommand = subArgs[0];
  
  switch (subcommand) {
    // Pre-Operation Hooks
    case 'pre-task':
      await preTaskCommand(subArgs, flags);
      break;
    case 'pre-edit':
      await preEditCommand(subArgs, flags);
      break;
    case 'pre-bash':
      await preBashCommand(subArgs, flags);
      break;
      
    // Post-Operation Hooks
    case 'post-task':
      await postTaskCommand(subArgs, flags);
      break;
    case 'post-edit':
      await postEditCommand(subArgs, flags);
      break;
    case 'post-bash':
      await postBashCommand(subArgs, flags);
      break;
      
    // MCP Integration Hooks
    case 'mcp-initialized':
      await mcpInitializedCommand(subArgs, flags);
      break;
    case 'agent-spawned':
      await agentSpawnedCommand(subArgs, flags);
      break;
  }
}
```

### 5.3 Hook Execution: Actual Operation

When a task executes in Claude Code:

```
USER IN CLAUDE CODE: "Build a REST API"
           ↓
┌─────────────────────────────────────────────────┐
│ Claude Code SDK detects PreToolUse              │
└─────────────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────────────┐
│ Claude Code calls settings.json hook:           │
│ "npx claude-flow hooks pre-task                 │
│   --description 'Build REST API'                │
│   --task-id task-123"                           │
└─────────────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────────────┐
│ claude-flow/hooks.js pre-task handler:          │
│ 1. Store task context to SQLite                 │
│ 2. Generate task ID                             │
│ 3. Store to: .swarm/memory.db:task:task-123    │
│ 4. Return context to Claude Code                │
└─────────────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────────────┐
│ Claude Code proceeds with tool execution        │
└─────────────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────────────┐
│ Claude Code detects PostToolUse                 │
└─────────────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────────────┐
│ Claude Code calls hook:                         │
│ "npx claude-flow hooks post-task                │
│   --task-id task-123                            │
│   --result '{...}'"                             │
└─────────────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────────────┐
│ claude-flow/hooks.js post-task handler:         │
│ 1. Retrieve task from: task:task-123            │
│ 2. Update with result                           │
│ 3. Store to: .swarm/memory.db:task:task-123    │
│ 4. Update agent memory                          │
│ 5. Record metrics                               │
└─────────────────────────────────────────────────┘
```

---

## 6. MEMORY PERSISTENCE FLOW

### 6.1 Cross-Session Memory Architecture

```
CLAUDE CODE SESSION 1          CLAUDE CODE SESSION 2
(Isolated by SDK)              (Isolated by SDK)
       │                              │
       │                              │
       ├──Hook: post-task─────────┐   └──Hook: pre-task────────┐
       │                          │                            │
       ↓                          ↓                            ↓
┌──────────────────────────────────────────────────────┐
│         CLAUDE-FLOW MEMORY LAYER                     │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │  SQLite Database: .swarm/memory.db             │ │
│  │  ├── agents (state, capabilities)             │ │
│  │  ├── tasks (history, results)                 │ │
│  │  ├── memories (knowledge, patterns)           │ │
│  │  └── metrics (performance, costs)             │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │  Checkpoint Store: .claude/checkpoints/       │ │
│  │  ├── UUID-based snapshots                     │ │
│  │  ├── Message history markers                  │ │
│  │  └── State recovery points                    │ │
│  └────────────────────────────────────────────────┘ │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │  Session Store: .claude/sessions/             │ │
│  │  ├── session-*.json with full state           │ │
│  │  └── Metrics per session                      │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
       ↑                          ↑
       │                          │
       └──Hook: session-end───────┴──Hook: session-restore
```

### 6.2 Memory Storage: Code Evidence

**Location**: `/bin/hive-mind/mcp-wrapper.js`

```javascript
export class MCPToolWrapper {
  async initializeMemoryStorage() {
    try {
      const sqliteAvailable = await isSQLiteAvailable();
      if (!sqliteAvailable) {
        throw new Error('SQLite not available');
      }

      const hiveMindDir = path.join(process.cwd(), '.hive-mind');
      const dbPath = path.join(hiveMindDir, 'memory.db');
      this.memoryDb = await createDatabase(dbPath);

      // Create memories table
      this.memoryDb.exec(`
        CREATE TABLE IF NOT EXISTS memories (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          namespace TEXT NOT NULL,
          key TEXT NOT NULL,
          value TEXT NOT NULL,
          type TEXT DEFAULT 'knowledge',
          timestamp INTEGER NOT NULL,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          UNIQUE(namespace, key)
        )
      `);
    } catch (error) {
      console.warn('Failed to initialize SQLite storage, falling back to in-memory:', error);
      this.memoryDb = null;
      this.memoryStore = new Map();
    }
  }
}
```

### 6.3 Memory Namespacing

Claude-Flow organizes memory by namespace:

```
.swarm/memory.db structure:

namespace='hooks:pre-task'
├── task:task-123 → { taskId, description, startedAt }
├── task:task-124 → { ... }
└── task:task-125 → { ... }

namespace='hooks:post-task'
├── task:task-123:result → { success, output, metrics }
└── ...

namespace='agent:state'
├── agent:coder-1 → { id, type, status, capabilities }
├── agent:researcher-1 → { ... }
└── ...

namespace='memory:patterns'
├── pattern:successful-implementations → { examples, frequency }
└── ...

namespace='task-index'
├── task-index:1729608000000 → { taskId, description, timestamp }
└── ...
```

---

## 7. COMMAND EXECUTION FLOW

### 7.1 How Commands Are Actually Executed

**Location**: `/src/cli/command-registry.js`

```javascript
export async function executeCommand(name, subArgs, flags) {
  const command = commandRegistry.get(name);
  if (!command) {
    throw new Error(`Unknown command: ${name}`);
  }

  try {
    // Track command execution for performance metrics
    await trackCommandExecution(name, command.handler, subArgs, flags);
  } catch (err) {
    throw new Error(`Command '${name}' failed: ${err.message}`);
  }
}

// Command registry contains handlers, not SDK calls
const commandRegistry = new Map();
commandRegistry.set('agent', {
  handler: agentCommand,
  description: 'Manage AI agents and execution'
});
```

**Key Point**: Commands execute through **handler functions**, not direct SDK invocation.

### 7.2 Agent Command Execution

When `claude-flow agent run coder "Build API"` is executed:

```
Entry: src/cli/command-registry.js (executeCommand)
       │
       ↓
handler: agentCommand (src/cli/simple-commands/agent.js)
       │
       ├─Task: spawn coder agent
       │
       ├─Hook: pre-task (session setup)
       │ ├─Store: .swarm/memory.db:task:task-id
       │ └─Return: session context
       │
       ├─Execute: agent.run() via SDK adapter
       │ └─Uses: ClaudeFlowSDKAdapter.createMessage()
       │    └─Uses: Anthropic.messages.create()
       │
       └─Hook: post-task (result storage)
         ├─Store: result to memory
         ├─Update: agent state
         └─Record: metrics
```

---

## 8. KEY ARCHITECTURAL DIFFERENCES

### 8.1 What claude-flow Does NOT Do

| Function | Not done |
|----------|----------|
| **Direct SDK wrapping** | ❌ Does NOT wrap @anthropic-ai/claude-code SDK directly |
| **Inherit sessions** | ❌ Claude Code sessions are independent |
| **Use SDK's memory** | ❌ Maintains separate memory system |
| **Override Claude Code** | ❌ Runs alongside, doesn't replace |
| **Modify SDK behavior** | ❌ Uses hooks as coordination points |

### 8.2 What claude-flow DOES Do

| Function | Implementation |
|----------|-----------------|
| **Wrap Anthropic SDK** | ✅ ClaudeFlowSDKAdapter wraps @anthropic-ai/sdk |
| **Manage parallel sessions** | ✅ Via .claude/sessions/ and HiveMindSessionManager |
| **Persist context** | ✅ SQLite, JSON, in-memory stores |
| **Coordinate agents** | ✅ Via 87+ MCP tools |
| **Hook into operations** | ✅ Pre/post hooks capture context |
| **Function as MCP server** | ✅ Registered in Claude Code settings.json |

---

## 9. SESSION ID MAPPING

### 9.1 The Three IDs

When you run a command in Claude Code that triggers a claude-flow agent:

```
┌─────────────────────────────────────────────────────────┐
│  CLAUDE CODE INTERNAL SESSION ID                        │
│  (Created by Claude Code SDK, unknown to claude-flow)  │
│                                                         │
│  Example: Internal SDK session state                   │
└─────────────────────────────────────────────────────────┘
                     ↓ (Not directly tracked)
┌─────────────────────────────────────────────────────────┐
│  CLAUDE-FLOW SESSION ID                                 │
│  task-1729608000000-abc123                              │
│  (Generated by claude-flow, stored in memory)          │
│                                                         │
│  Location: .swarm/memory.db:task:task-1729608000000   │
└─────────────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│  CLAUDE-FLOW SWARM SESSION ID (if agent spawning)      │
│  swarm-session-<timestamp>-<random>                    │
│  (Tracks the multi-agent coordination)                 │
│                                                         │
│  Location: .hive-mind/sessions/ table                  │
└─────────────────────────────────────────────────────────┘
```

**They are NOT the same**. Claude-Flow maintains a parallel ID system.

---

## 10. COMMAND LINE INTERFACE (CLI) ARCHITECTURE

### 10.1 CLI Execution Path

```
$ npx claude-flow agent run coder "Build API"
                     │
                     ↓
         /bin/claude-flow.js (dispatcher)
                     │
                     ├─Check for simple-cli.js or simple-cli.ts
                     │
                     ↓
      src/cli/simple-cli.ts (main entry)
                     │
                     ├─Parse command: "agent"
                     │
                     ↓
    src/cli/command-registry.js
                     │
                     ├─Get handler: agentCommand
                     │
                     ↓
    src/cli/simple-commands/agent.js
                     │
                     ├─Switch: run subcommand
                     │
                     ├─Spawn agent via handler
                     │
                     ├─Hook: pre-task
                     │
                     ├─SDK: Anthropic.messages.create()
                     │
                     └─Hook: post-task
```

---

## 11. CRITICAL FINDINGS SUMMARY

### Finding 1: No Direct Claude Code SDK Wrapping
**Evidence**: 
- Package.json has `@anthropic-ai/claude-code@^2.0.1` as dependency
- But NO imports of claude-code SDK found in source code
- Instead, direct `@anthropic-ai/sdk` imports everywhere

### Finding 2: Session Management Is External
**Evidence**:
- `.claude/sessions/` stores metrics independent of Claude Code
- `.claude/checkpoints/` has 139,264 UUID-based snapshots
- `HiveMindSessionManager` manages its own session DB
- No reliance on Claude Code's session management

### Finding 3: Context Is Injected, Not Inherited
**Evidence**:
- Pre-task hooks retrieve context from SQLite
- Post-task hooks store results outside SDK
- No access to Claude Code's internal message history
- Context injected via prompt augmentation

### Finding 4: MCP Is The Coordination Mechanism
**Evidence**:
- `settings.json` registers claude-flow as MCP server
- 87+ MCP tools for agent spawning, task orchestration
- Each tool is a separate entry point for coordination
- Not through Claude Code SDK's native coordination

### Finding 5: Memory Is Completely Separate
**Evidence**:
- SQLite stores memories in .swarm/memory.db
- JSON checkpoints in .claude/checkpoints/
- No integration with Claude Code's memory model
- Completely parallel persistence system

---

## 12. EXECUTION FLOW DIAGRAM: Complete Picture

```
CLAUDE CODE EDITOR                                     
     │
     │ User: "Run agent to build API"
     │
     ├─Claude Code Hook: PreToolUse
     │  │
     │  └─Calls MCP: mcp__claude-flow__task_orchestrate
     │
┌────▼────────────────────────────────────────────────────────────┐
│                    CLAUDE-FLOW MCP SERVER                        │
│                                                                  │
│  1. task_orchestrate handler (MCP tool)                         │
│     ├─Parse: "Build API"                                        │
│     └─Create task ID: task-1729608000000-abc123                │
│                                                                  │
│  2. Memory: Store task context                                  │
│     └─INSERT INTO memories: namespace='hooks:pre-task'         │
│                                                                  │
│  3. Spawn agent (if needed)                                     │
│     ├─MCP tool: agent_spawn                                    │
│     └─Creates: agent-coder-1                                   │
│                                                                  │
│  4. Execute via SDK Adapter                                    │
│     └─ClaudeFlowSDKAdapter.createMessage()                     │
│        └─Anthropic.messages.create(                            │
│           model: 'claude-3-sonnet',                            │
│           messages: [...],                                     │
│           system: context_injected_prompt                      │
│        )                                                        │
│                                                                  │
│  5. Store result                                                │
│     └─INSERT INTO memories: namespace='hooks:post-task'        │
│                                                                  │
└────┬────────────────────────────────────────────────────────────┘
     │
     └─Claude Code Hook: PostToolUse
        │
        └─Calls MCP: mcp__claude-flow__notify
           │
           └─Displays result in editor
```

---

## CONCLUSION: How claude-flow Actually Wraps Claude Code

Claude-Flow does **NOT** wrap claude-code SDK. Instead, it:

1. **Runs ALONGSIDE Claude Code**, not over it
2. **Uses Claude Code's hook system** for coordination
3. **Registers as an MCP server** that Claude Code can call
4. **Wraps the Anthropic SDK directly** for API access
5. **Maintains its own session system** completely separate from Claude Code
6. **Persists context externally** in SQLite and JSON files
7. **Injects context** into prompts during execution
8. **Coordinates multiple agents** through MCP tool calls

The "wrapping" is NOT at the SDK level—it's at the **coordination and orchestration level**. Claude-Flow is a sophisticated orchestration middleware, not a direct wrapper of the claude-code SDK.
