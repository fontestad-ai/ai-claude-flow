# Complete Cross-Session Memory Persistence Architecture
## Claude-Flow v2.5-alpha.130+ Analysis

---

## EXECUTIVE SUMMARY

The Claude-Flow cross-session persistence system provides sophisticated state management across development sessions through:
1. **Multi-backend persistence** (SQLite, JSON, distributed memory)
2. **Session forking & checkpointing** with message UUIDs
3. **Memory hooks** for automatic state synchronization
4. **Serialization/deserialization** with type preservation
5. **Metrics export & analysis** for session optimization

---

## 1. SESSION MANAGEMENT ARCHITECTURE

### 1.1 Session Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│                    SESSION LIFECYCLE                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CREATE          INITIALIZE         ACTIVE        END       │
│    │                 │                │            │        │
│    ↓                 ↓                ↓            ↓        │
│  ┌──┐  pre-task  ┌────────┐  hooks  ┌──┐  session ┌──┐    │
│  │ID│◄──────────┤SESSION │◄───────┤WK│◄─ end   │X │    │
│  └──┘           └────────┘         └──┘         └──┘    │
│    │                 │                │            │        │
│    └─────STORE──────┘────CHECKPOINT──┴─PERSIST───┘        │
│         (Memory)         (UUID)      (Files/DB)            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Core Session Components

```
SESSION OBJECT:
├── id: string (session_<timestamp>_<random>)
├── agentId: string
├── agentType: string
├── transport: 'stdio' | 'http' | 'websocket'
├── clientInfo: { name, version }
├── protocolVersion: { major, minor, patch }
├── capabilities: Record<string, any>
├── isInitialized: boolean
├── authenticated: boolean
├── createdAt: Date
├── lastActivity: Date
├── lastActivityTime?: Date
└── metadata:
    ├── version: '1.0.0' | '2.0.0'
    ├── platform: string
    ├── checksum: string
    ├── active: boolean (tracking)
    └── orphaned: boolean
```

### 1.3 Session Manager Implementation

**Location**: `/src/mcp/session-manager.ts`

```typescript
interface ISessionManager {
  createSession(transport): MCPSession
  getSession(id): MCPSession | undefined
  initializeSession(sessionId, params): void
  authenticateSession(sessionId, credentials): boolean
  updateActivity(sessionId): void
  removeSession(sessionId): void
  getActiveSessions(): MCPSession[]
  cleanupExpiredSessions(): void
  getSessionMetrics(): {
    total: number
    active: number
    authenticated: number
    expired: number
  }
}
```

**Key Features**:
- Session timeout: 3600000ms (1 hour default)
- Max sessions: 100 concurrent
- Cleanup interval: 60000ms (1 minute)
- Automatic expiration detection
- Auth method support: token, basic, oauth

---

## 2. PERSISTENCE BACKENDS

### 2.1 Multi-Backend Architecture

```
PERSISTENCE LAYER:
┌─────────────────────────────────────────────────┐
│                  PERSISTENCE                     │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────┐│
│  │   SQLite DB  │  │ JSON Files   │  │Memory  ││
│  │ (Agents,     │  │ (Sessions,   │  │(Cache) ││
│  │  Tasks)      │  │  Checkpts)   │  │        ││
│  └──────────────┘  └──────────────┘  └────────┘│
│        │                 │               │      │
│        └────────┬────────┴───────────────┘      │
│                 │                               │
│        ┌────────▼────────┐                      │
│        │ Session Manager │                      │
│        └────────┬────────┘                      │
│                 │                               │
└─────────────────┼───────────────────────────────┘
                  │
            ┌─────▼─────┐
            │ CLI / SDK │
            └───────────┘
```

### 2.2 SQLite Persistence (Database)

**Location**: `/src/core/persistence.ts`

```sql
-- AGENTS TABLE
CREATE TABLE agents (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  name TEXT NOT NULL,
  status TEXT NOT NULL,
  capabilities TEXT NOT NULL,
  system_prompt TEXT NOT NULL,
  max_concurrent_tasks INTEGER,
  priority INTEGER,
  created_at INTEGER
);

-- TASKS TABLE
CREATE TABLE tasks (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  description TEXT NOT NULL,
  status TEXT NOT NULL,
  priority INTEGER,
  dependencies TEXT,
  metadata TEXT,
  assigned_agent TEXT,
  progress INTEGER DEFAULT 0,
  error TEXT,
  created_at INTEGER,
  completed_at INTEGER
);

-- SESSIONS TABLE
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  agent_id TEXT NOT NULL,
  terminal_id TEXT NOT NULL,
  status TEXT NOT NULL,
  created_at INTEGER,
  FOREIGN KEY (agent_id) REFERENCES agents(id)
);
```

### 2.3 JSON Persistence (File-based)

**Location**: `/src/core/json-persistence.ts`

```json
{
  "agents": [
    {
      "id": "agent-001",
      "type": "researcher",
      "name": "Lead Researcher",
      "status": "active",
      "capabilities": ["analysis", "research"],
      "createdAt": 1729608000000
    }
  ],
  "tasks": [
    {
      "id": "task-001",
      "type": "research",
      "description": "Research topic",
      "status": "in_progress",
      "progress": 50,
      "createdAt": 1729608000000
    }
  ],
  "lastUpdated": 1729608600000
}
```

### 2.4 Session File Storage

**Location**: `.claude-flow/sessions/`

```
SESSION STRUCTURE:
{
  id: "session-1729608000000-abc123",
  name: "dev-session-2024",
  description: "Authentication system development",
  tags: ["auth", "security", "api"],
  createdAt: "2025-10-23T10:00:00Z",
  updatedAt: "2025-10-23T11:30:00Z",
  
  state: {
    agents: [
      {
        id: "agent-001",
        type: "coder",
        status: "active"
      }
    ],
    tasks: [
      {
        id: "task-001",
        type: "implementation",
        status: "in_progress"
      }
    ],
    memory: [
      {
        id: "memory-001",
        type: "conversation",
        agentId: "agent-001"
      }
    ],
    configuration: {
      orchestrator: { maxAgents: 10 },
      memory: { backend: "hybrid" }
    }
  },
  
  metadata: {
    version: "1.0.0" | "2.0.0",
    platform: "linux|darwin|win32",
    checksum: "sha256hash"
  }
}
```

---

## 3. CHECKPOINT & VERSION CONTROL

### 3.1 Real Checkpoint Manager

**Location**: `/src/sdk/checkpoint-manager.ts`

```typescript
interface Checkpoint {
  id: string;              // Message UUID (unique identifier)
  sessionId: string;       // Parent session
  description: string;     // User description
  timestamp: number;       // Creation time
  messageCount: number;    // Messages up to checkpoint
  totalTokens: number;     // Token usage
  filesModified: string[]; // Modified files
}

class RealCheckpointManager {
  async createCheckpoint(sessionId, description): Promise<string>
  async rollbackToCheckpoint(checkpointId, continuePrompt?): Promise<Query>
  listCheckpoints(sessionId): Checkpoint[]
  getCheckpoint(checkpointId): Checkpoint | undefined
  async deleteCheckpoint(checkpointId): Promise<void>
  getCheckpointDiff(fromId, toId): { messagesDiff, tokensDiff, filesAdded, filesRemoved }
}
```

### 3.2 Checkpoint Data Flow

```
CHECKPOINT CREATION:
┌─────────────────────────────────────┐
│  Session Message Stream             │
│  ┌─────────────────────────────┐   │
│  │ Message 1 (user input)      │   │
│  │ Message 2 (system)          │   │
│  │ Message 3 (assistant)       │ ──┼──> CHECKPOINT
│  │ ...                         │   │    (UUID = Message.uuid)
│  │ Message N (last message)    │   │
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
        │
        ↓
    ┌──────────────┐
    │ PERSIST TO   │
    │ DISK: .json  │
    │ checkpoints/ │
    └──────────────┘
        │
        ↓
┌──────────────────────┐
│ ROLLBACK MECHANISM   │
│ resumeSessionAt:UUID │
│ Rewind to checkpoint │
└──────────────────────┘

KEY: Checkpoint ID = Message UUID
     Enables git-like time travel to exact message
```

### 3.3 Session Serialization

**Location**: `/src/memory/enhanced-session-serializer.js`

```typescript
class SessionSerializer {
  // Serialize with metadata
  serializeSessionData(sessionData): string
  
  // Deserialize with validation and migration
  deserializeSessionData(serializedData, options?): any
  
  // Checkpoint serialization
  serializeCheckpointData(checkpointData): string
  deserializeCheckpointData(serializedData): any
  
  // Version migration (v1.0.0 → v2.0.0)
  _migrateSessionData(data, fromVersion): void
}

// Serialization flow:
preprocessSessionData() 
  → enhancedData with __session_meta__
    → serializer.serialize()
      → stored/transmitted
        → deserializer.deserialize()
          → validation & migration
            → postprocessSessionData()
```

---

## 4. SESSION HOOKS & RESTORATION

### 4.1 Hook Integration

**Location**: `/src/services/agentic-flow-hooks/`

```
HOOK EXECUTION TIMELINE:
┌──────────────────────────────────────────────────┐
│                  SESSION LIFECYCLE                │
├──────────────────────────────────────────────────┤
│                                                   │
│  BEFORE SESSION START:                           │
│  ┌──────────────────────────────────┐           │
│  │ npx claude-flow hooks             │           │
│  │   session-restore                 │           │
│  │   --session-id "prev-session"     │           │
│  │   --load-memory true              │           │
│  │   --restore-files true            │           │
│  └──────────────────────────────────┘           │
│       ↓                                          │
│   Load previous:                                │
│   • Agent configurations                        │
│   • Task progress                               │
│   • Memory entries                              │
│   • File contexts                               │
│                                                   │
│  DURING SESSION:                                │
│  ┌──────────────────────────────────┐           │
│  │ npx claude-flow hooks             │           │
│  │   post-edit --file "file.ts"      │           │
│  │   --memory-key "path/to/state"    │           │
│  │   --telemetry true                │           │
│  └──────────────────────────────────┘           │
│       ↓                                          │
│   After each step:                              │
│   • Store decisions                             │
│   • Update progress                             │
│   • Track dependencies                          │
│                                                   │
│  AFTER SESSION END:                             │
│  ┌──────────────────────────────────┐           │
│  │ npx claude-flow hooks             │           │
│  │   session-end                     │           │
│  │   --session-id "current-session"  │           │
│  │   --save-state true               │           │
│  │   --export-metrics true           │           │
│  │   --generate-summary true         │           │
│  └──────────────────────────────────┘           │
│       ↓                                          │
│   Persist:                                      │
│   • Session state                               │
│   • Performance metrics                         │
│   • Session summary                             │
│   • Task completion status                      │
│                                                   │
└──────────────────────────────────────────────────┘
```

### 4.2 Memory Hooks Implementation

**Location**: `/src/services/agentic-flow-hooks/memory-hooks.ts`

```typescript
// PRE-MEMORY STORE HOOK
preMemoryStoreHook: {
  // Validation: size limits, namespace quotas
  // Compression: auto-compress values > 1KB
  // Enrichment: add metadata (timestamp, provider, sessionId)
  // Tracking: memory usage metrics
}

// POST-MEMORY STORE HOOK
postMemoryStoreHook: {
  // Cross-provider sync
  // Memory indexing
  // Pattern detection (neural)
  // Change notification
}

// PRE-MEMORY RETRIEVE HOOK
preMemoryRetrieveHook: {
  // Cache checking
  // Prefetching related keys
  // Access pattern tracking
}

// POST-MEMORY RETRIEVE HOOK
postMemoryRetrieveHook: {
  // Decompression
  // Access pattern updates
  // Local caching
  // Latency tracking
}

// MEMORY SYNC HOOK
memorySyncHook: {
  // Bidirectional synchronization
  // Change detection
  // Memory snapshots
  // Expiration cleanup
}

// MEMORY PERSIST HOOK
memoryPersistHook: {
  // Full backup creation
  // Metadata storage
  // Checksum calculation
  // TTL management (7 days default)
}
```

---

## 5. SESSION FORKING & PARALLEL EXECUTION

### 5.1 Session Forking Architecture

**Location**: `/src/sdk/session-forking.ts`

```
PARALLEL AGENT SPAWNING WITH SESSION FORKING:

BASE SESSION
    │
    ├─────fork──────> [FORKED SESSION 1]
    │                 Agent: researcher-1
    │                 Messages: [...]
    │                 Status: active
    │
    ├─────fork──────> [FORKED SESSION 2]
    │                 Agent: coder-1
    │                 Messages: [...]
    │                 Status: active
    │
    └─────fork──────> [FORKED SESSION 3]
                      Agent: tester-1
                      Messages: [...]
                      Status: active

PERFORMANCE GAIN:
┌─────────────────────────────────────┐
│ Sequential: 3 agents × 750ms = 2250ms
│ Forked:     3 agents parallel = 750ms
│ Gain:       10-20x faster            │
└─────────────────────────────────────┘

KEY SDK OPTIONS:
{
  forkSession: true,           // Enable session forking
  resume: baseSessionId,       // Resume from base session
  resumeSessionAt: messageId,  // Resume from specific message
  model: 'claude-sonnet-4',    // Model to use
  maxTurns: 50,                // Max turns per agent
  timeout: 60000,              // Timeout per agent
  mcpServers: {...}            // MCP configuration
}
```

### 5.2 ParallelSwarmExecutor Implementation

```typescript
class ParallelSwarmExecutor {
  async spawnParallelAgents(
    configs: ParallelAgentConfig[],
    options: SessionForkOptions = {}
  ): Promise<ParallelExecutionResult>
  
  // Execution metrics
  executionMetrics: {
    totalAgentsSpawned: number
    parallelExecutions: number
    avgSpawnTime: number
    performanceGain: number  // Est. sequential / actual
  }
  
  // Session tracking
  activeSessions: Map<string, ForkedSession>
  sessionHistory: Map<string, SDKMessage[]>
}

// Execution flow:
1. Sort by priority (critical → high → medium → low)
2. Create batches (maxParallel agents per batch)
3. Spawn all agents in batch concurrently
4. Collect messages from each forked session
5. Track session history for checkpointing
6. Calculate performance metrics
```

---

## 6. MEMORY STORAGE & MANAGEMENT

### 6.1 Advanced Memory Manager

**Location**: `/src/memory/advanced-memory-manager.ts`

```
MEMORY ENTRY STRUCTURE:
{
  id: string                    // Unique ID
  key: string                   // Access key
  value: any                    // Stored data
  type: string                  // Entry type
  namespace: string             // Logical grouping
  tags: string[]                // Search tags
  metadata: Record<string, any> // Custom metadata
  owner: string                 // Agent ID
  accessLevel: 'private' | 'shared' | 'public'
  createdAt: Date
  updatedAt: Date
  lastAccessedAt: Date
  expiresAt?: Date              // Optional TTL
  version: number               // Entry version
  size: number                  // Size in bytes
  compressed: boolean
  checksum: string              // Integrity check
  references: string[]          // Entry dependencies
  dependencies: string[]        // What it depends on
}

MEMORY INDEX:
┌──────────────────────────────────┐
│          INDEX STRUCTURES         │
├──────────────────────────────────┤
│ keys:       Map<key → entryIds>  │
│ tags:       Map<tag → entryIds>  │
│ types:      Map<type → entryIds> │
│ namespaces: Map<ns → entryIds>   │
│ owners:     Map<agent → ids>     │
│ fullText:   Map<word → entryIds> │
└──────────────────────────────────┘
```

### 6.2 Memory Operations

```typescript
// QUERY OPTIONS
interface QueryOptions {
  namespace?: string
  type?: string
  tags?: string[]
  owner?: string
  accessLevel?: 'private' | 'shared' | 'public'
  keyPattern?: string
  valueSearch?: string
  fullTextSearch?: string
  createdAfter?: Date
  createdBefore?: Date
  updatedAfter?: Date
  updatedBefore?: Date
  sizeGreaterThan?: number
  sizeLessThan?: number
  limit?: number
  offset?: number
  sortBy?: 'key' | 'createdAt' | 'updatedAt' | 'lastAccessedAt' | 'size'
  sortOrder?: 'asc' | 'desc'
}

// EXPORT/IMPORT
export async function exportMemory(
  format: 'json' | 'csv' | 'xml' | 'yaml',
  options: ExportOptions
): Promise<string>

async function importMemory(
  data: string,
  options: ImportOptions
): Promise<{ imported: number; skipped: number; errors: string[] }>

// CLEANUP
async function cleanup(options: CleanupOptions): Promise<{
  removed: number
  archived: number
  compressed: number
  reclaimed: number  // bytes freed
}>
```

### 6.3 Memory Statistics & Health

```typescript
interface MemoryStatistics {
  overview: {
    totalEntries: number
    totalSize: number
    compressedEntries: number
    compressionRatio: number
    indexSize: number
    memoryUsage: number
    diskUsage: number
  }
  
  distribution: {
    byNamespace: { count, size }
    byType: { count, size }
    byOwner: { count, size }
    byAccessLevel: { count, size }
  }
  
  temporal: {
    entriesCreatedLast24h: number
    entriesUpdatedLast24h: number
    entriesAccessedLast24h: number
    oldestEntry?: Date
    newestEntry?: Date
  }
  
  performance: {
    averageQueryTime: number
    averageWriteTime: number
    cacheHitRatio: number
    indexEfficiency: number
  }
  
  health: {
    expiredEntries: number
    orphanedReferences: number
    duplicateKeys: number
    corruptedEntries: number
    recommendedCleanup: boolean
  }
  
  optimization: {
    suggestions: string[]
    potentialSavings: {
      compression: number
      cleanup: number
      deduplication: number
    }
  }
}
```

---

## 7. DATA FLOW DIAGRAMS

### 7.1 Complete Session Persistence Flow

```
SESSION CREATION & PERSISTENCE:

┌──────────────────┐
│  User Initiates  │
│  New Session     │
└────────┬─────────┘
         │
         ↓
┌──────────────────────────────┐
│ SessionManager.createSession() │
│  • Generate session ID        │
│  • Set timestamps             │
│  • Initialize state           │
└────────┬─────────────────────┘
         │
         ↓
┌──────────────────────────────┐
│ MCP Session Object Created    │
│ {                            │
│   id, agent, transport,      │
│   createdAt, lastActivity    │
│ }                            │
└────────┬─────────────────────┘
         │
         ↓
┌──────────────────────────────┐
│ Session Stored in Memory     │
│ (sessions Map)               │
└────────┬─────────────────────┘
         │
    PARALLEL: Storage Paths
    │
    ├──────────────────────────────────┐
    │                                  │
    ↓                                  ↓
┌─────────────────────┐      ┌──────────────────┐
│ SQLite Database     │      │ JSON Session File│
│ (.claude-flow.db)   │      │ (sessions/id.json)
│ • agents table      │      │                  │
│ • tasks table       │      │ __session_meta__ │
│ • sessions table    │      │ version: 2.0.0   │
│                     │      │ platform: linux  │
│                     │      │ checksum: sha256 │
└─────────────────────┘      └──────────────────┘
    │                                  │
    └──────────────────┬───────────────┘
                       │
                       ↓
            ┌────────────────────┐
            │ Cleanup Timer      │
            │ (60s intervals)    │
            │ Remove expired     │
            └────────┬───────────┘
                     │
                     ↓
          ┌──────────────────────┐
          │ Session Cleanup      │
          │ isSessionExpired()?  │
          │ Remove if > timeout  │
          └──────────────────────┘
```

### 7.2 Session Restoration Flow

```
SESSION RESTORATION & RECOVERY:

┌────────────────────────────┐
│ User Requests              │
│ Session Restore            │
│ session-restore            │
│ --session-id "prev-id"     │
└────────┬───────────────────┘
         │
         ↓
┌────────────────────────────────┐
│ Load Session Metadata           │
│ • Check file exists             │
│ • Verify checksum              │
│ • Load __session_meta__        │
└────────┬───────────────────────┘
         │
         ↓
    IS CHECKSUM VALID?
    /                    \
   YES                    NO
   │                      │
   ↓                      ↓
┌──────────────────┐  ┌──────────────────┐
│ Proceed with     │  │ Warn user        │
│ deserialization  │  │ Data may be      │
│                  │  │ corrupted        │
└────────┬─────────┘  │ Continue anyway? │
         │            └────────┬─────────┘
         │                     │
         ↓                     ↓
┌────────────────────────────────┐
│ Deserialize Session Data       │
│ • Parse JSON                   │
│ • Handle migration (v1→v2)     │
│ • Restore Date objects         │
│ • Decompress if needed         │
└────────┬───────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│ Restore Components in Parallel     │
│                                    │
│ ├─ Load agent configs              │
│ ├─ Load task progress              │
│ ├─ Load memory entries             │
│ ├─ Restore file contexts           │
│ └─ Recover dependencies            │
└────────┬───────────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│ Update SessionManager              │
│ • Add to active sessions           │
│ • Set lastActivity timestamp       │
│ • Restore authentication state     │
└────────┬───────────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│ Session Ready                      │
│ • Continue work                    │
│ • Access all recovered state       │
│ • Use shared memory                │
└────────────────────────────────────┘
```

### 7.3 Checkpoint & Rollback Flow

```
CHECKPOINT CREATION & ROLLBACK:

CREATE CHECKPOINT:
┌─────────────────────────────┐
│ Session Message Stream      │
│ message[0] → message[N]     │
│ Each with UUID              │
└────────┬────────────────────┘
         │
         ↓
┌──────────────────────────────┐
│ Detect Checkpoint Trigger    │
│ • User requested             │
│ • Auto-checkpoint at N msgs  │
│ • Milestone reached          │
└────────┬─────────────────────┘
         │
         ↓
┌──────────────────────────────────┐
│ Extract Session State             │
│ • Message count                  │
│ • Token usage                    │
│ • Files modified                 │
│ • Checkpoint ID = Last Message   │
│   UUID                           │
└────────┬───────────────────────────┘
         │
         ↓
┌──────────────────────────────────┐
│ Create Checkpoint Object          │
│ {                                │
│   id: messageUUID,               │
│   sessionId: "xxx",              │
│   messageCount: N,               │
│   totalTokens: T,                │
│   filesModified: [...],          │
│   timestamp: now()               │
│ }                                │
└────────┬───────────────────────────┘
         │
         ↓
┌──────────────────────────────────┐
│ Persist Checkpoint                │
│ .claude-flow/checkpoints/         │
│ {uuid}.json                      │
└────────┬───────────────────────────┘
         │
         ↓
    ┌────────────────┐
    │ CHECKPOINT OK  │
    └────────────────┘

---

ROLLBACK TO CHECKPOINT:
┌──────────────────────────────┐
│ User Requests Rollback       │
│ checkpoint rollback <id>     │
└────────┬─────────────────────┘
         │
         ↓
┌──────────────────────────────┐
│ Find Checkpoint              │
│ Load from disk or memory     │
│ Verify checkpoint exists     │
└────────┬─────────────────────┘
         │
         ↓
┌──────────────────────────────┐
│ Confirm with User            │
│ Show:                        │
│ • Description                │
│ • Created timestamp          │
│ • Messages count             │
│ • Files that will change     │
└────────┬─────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│ Use SDK resumeSessionAt              │
│ query({                              │
│   prompt: "continue prompt",         │
│   options: {                         │
│     resume: sessionId,               │
│     resumeSessionAt: checkpointId    │
│   }                                  │
│ })                                   │
└────────┬───────────────────────────────┘
         │
         ↓
┌──────────────────────────────────────┐
│ SDK Rewinds to Exact Message UUID    │
│ • Loads session history up to UUID   │
│ • Discards all messages after        │
│ • Maintains file state up to point   │
│ • Continues from exact point         │
└────────┬───────────────────────────────┘
         │
         ↓
    ┌────────────────┐
    │ ROLLBACK OK    │
    │ Continue work  │
    └────────────────┘
```

### 7.4 Memory Hooks Coordination Flow

```
MEMORY HOOKS EXECUTION PIPELINE:

STORE OPERATION:
┌──────────────────────────────┐
│ Agent Stores Value           │
│ namespace: "task"            │
│ key: "task-001"              │
│ value: { ... }               │
└────────┬─────────────────────┘
         │
         ↓
    ┌─────────────────────────┐
    │ PRE-STORE HOOK          │
    ├─────────────────────────┤
    │ • Validate size limits  │
    │ • Check quotas          │
    │ • Compress if > 1KB     │
    │ • Add metadata          │
    │ • Track usage metrics   │
    └────────┬────────────────┘
             │
             ↓
    ┌─────────────────────────┐
    │ Store in Backend        │
    │ (SQLite/Memory/File)    │
    └────────┬────────────────┘
             │
             ↓
    ┌────────────────────────────┐
    │ POST-STORE HOOK            │
    ├────────────────────────────┤
    │ • Cross-provider sync      │
    │ • Update memory index      │
    │ • Detect patterns (neural) │
    │ • Emit notification event  │
    └────────┬───────────────────┘
             │
             ↓
    ┌────────────────────────────┐
    │ MEMORY SYNC HOOK           │
    ├────────────────────────────┤
    │ • Bidirectional sync       │
    │ • Create snapshots         │
    │ • Detect changes           │
    │ • Manage expiration        │
    └────────┬───────────────────┘
             │
             ↓
    ┌────────────────────────────┐
    │ MEMORY PERSIST HOOK        │
    ├────────────────────────────┤
    │ • Create full backup       │
    │ • Store with checksum      │
    │ • Set TTL (7 days)         │
    │ • Track metrics            │
    └────────┬───────────────────┘
             │
             ↓
         ┌────────┐
         │ STORED │
         └────────┘

RETRIEVE OPERATION:
┌──────────────────────────────┐
│ Agent Retrieves Value        │
│ namespace: "task"            │
│ key: "task-001"              │
└────────┬─────────────────────┘
         │
         ↓
    ┌─────────────────────────┐
    │ PRE-RETRIEVE HOOK       │
    ├─────────────────────────┤
    │ • Check local cache     │
    │ • If hit: return (fast) │
    │ • Prefetch related keys │
    │ • Track access patterns │
    └────────┬────────────────┘
             │
             ↓
    ┌──────────────────────────┐
    │ Retrieve from Backend    │
    │ (SQLite/Memory/File)     │
    └────────┬─────────────────┘
             │
             ↓
    ┌────────────────────────────┐
    │ POST-RETRIEVE HOOK         │
    ├────────────────────────────┤
    │ • Decompress if needed     │
    │ • Update access patterns   │
    │ • Cache locally            │
    │ • Track retrieval latency  │
    └────────┬───────────────────┘
             │
             ↓
         ┌──────────┐
         │ RETURNED │
         └──────────┘
```

---

## 8. WHAT PERSISTS ACROSS SESSIONS

### 8.1 Automatic Persistence

```
✅ PERSISTS:
├── Session metadata
│   ├── Session ID
│   ├── Creation/update timestamps
│   ├── Session metadata (version, platform)
│   └── Checksum for integrity
│
├── Agent state
│   ├── Agent IDs and types
│   ├── Agent status (active, idle, complete)
│   ├── Agent capabilities
│   └── Agent configurations
│
├── Task state
│   ├── Task descriptions
│   ├── Task status and progress
│   ├── Task dependencies
│   ├── Task assignments
│   └── Task completion timestamps
│
├── Memory entries
│   ├── All stored values
│   ├── Metadata (timestamps, owner, access level)
│   ├── Tags and search indices
│   ├── Compression status
│   └── TTL information
│
├── File context
│   ├── Files read during session
│   ├── Files modified/created
│   ├── Edit history
│   └── File reference tracking
│
├── Background processes
│   ├── Process IDs
│   ├── Command history
│   ├── Output position markers
│   └── Process state (running/completed/killed)
│
├── Checkpoints
│   ├── Message UUIDs
│   ├── Token usage at checkpoint
│   ├── Files modified up to point
│   ├── Descriptions and timestamps
│   └── Checkpoint history
│
└── Performance metrics
    ├── Session duration
    ├── Commands executed
    ├── Files modified count
    ├── Tokens consumed
    ├── Parallel execution stats
    └── Hook execution times
```

### 8.2 Non-Persisting Data

```
❌ DOES NOT PERSIST:
├── Live process outputs
│   └── Must re-read with BashOutput
│
├── Authentication tokens
│   └── Must re-authenticate
│
├── Temporary variables
│   ├── In-memory caches
│   └── Runtime state
│
├── Permissions
│   └── May need re-approval (security)
│
├── SSH sessions
│   └── Need to reconnect
│
└── Environment variables
    └── Set per-session
```

---

## 9. SESSION LIFECYCLE HOOKS

### 9.1 Hook Commands

```bash
# SESSION-START (Initialization)
npx claude-flow@alpha hooks session-start \
  --session-id "dev-session-2024" \
  --load-memory true \
  --auto-spawn-agents false

# PRE-TASK (Before work)
npx claude-flow@alpha hooks pre-task \
  --description "Research API patterns" \
  --task-id "task-001" \
  --agent-id "agent-001"

# POST-EDIT (After file modification)
npx claude-flow@alpha hooks post-edit \
  --file "src/api/server.js" \
  --memory-key "swarm/coder-1/step-1" \
  --telemetry true

# POST-TASK (After completing task)
npx claude-flow@alpha hooks post-task \
  --task-id "task-001" \
  --analyze-performance true

# SESSION-RESTORE (Resuming work)
npx claude-flow@alpha hooks session-restore \
  --session-id "dev-session-2024" \
  --load-memory true \
  --restore-files true \
  --resume-tasks true

# SESSION-END (Cleanup & persist)
npx claude-flow@alpha hooks session-end \
  --session-id "dev-session-2024" \
  --save-state true \
  --export-metrics true \
  --generate-summary true
```

### 9.2 Hook Output Format

```json
// session-end output
{
  "sessionId": "dev-session-2024",
  "duration": 7200000,
  "saved": true,
  "metrics": {
    "commandsRun": 145,
    "filesModified": 23,
    "tokensUsed": 85000,
    "tasksCompleted": 8
  },
  "summaryPath": "/sessions/dev-session-2024-summary.md",
  "cleanedUp": true,
  "nextSession": "dev-session-2025"
}

// session-restore output
{
  "sessionId": "dev-session-2024",
  "restored": true,
  "memories": 25,
  "files": ["src/auth/login.js", "src/api/users.js"],
  "tasks": {
    "incomplete": 3,
    "completed": 12
  },
  "context": {
    "project": "auth-system",
    "branch": "feature/oauth"
  },
  "warnings": []
}
```

---

## 10. RECOVERY & FAULT TOLERANCE

### 10.1 Corruption Detection & Recovery

```
INTEGRITY CHECKING:

1. CHECKSUM VERIFICATION
   ├── Calculated at save: SHA-256(session data)
   ├── Stored in metadata
   ├── Validated at restore
   └── Warning if mismatch

2. VERSION DETECTION
   ├── Store version in metadata
   ├── Migrate v1.0.0 → v2.0.0 automatically
   └── Handle breaking changes

3. FALLBACK DESERIALIZATION
   ├── Try advanced deserializer
   ├── Fall back to JSON.parse
   ├── Attempt legacy format recovery
   └── Return partial data if possible

4. PARTIAL RECOVERY
   ├── Recover session metadata
   ├── Recover agent configurations
   ├── Recover task progress
   └── Skip corrupted memory entries
```

### 10.2 Auto-Cleanup & Expiration

```
AUTOMATIC CLEANUP POLICY:

SESSION EXPIRATION:
• Timeout: 3600000ms (1 hour default)
• Check interval: 60000ms (1 minute)
• Auto-remove expired sessions

MEMORY ENTRY EXPIRATION:
• Default TTL: 604800s (7 days)
• Cleanup on retrieval
• Expired entry removal
• Auto-archive old entries

CHECKPOINT CLEANUP:
• Max checkpoints per session: 50
• Oldest checkpoints deleted when limit reached
• Manual deletion supported

DISK SPACE MANAGEMENT:
• Monitor total session disk usage
• Archive old sessions
• Compress large entries
• Clean temporary files
```

---

## 11. PERFORMANCE CHARACTERISTICS

### 11.1 Latency Profiles

```
OPERATION LATENCIES (Approximate):

Session Creation:          5-10ms
Session Lookup:           <1ms
Session Initialization:   10-50ms
Session Cleanup:          5-20ms

Memory Store:             10-100ms (compress: 50-200ms)
Memory Retrieve:          10-50ms (decompress: 50-150ms)
Memory Query:             50-500ms (indexed: 10-100ms)
Memory Sync:              100-500ms

Checkpoint Create:        50-200ms
Checkpoint Rollback:      100-500ms (SDK rewind)
Checkpoint Diff:          20-100ms

Session Serialization:    50-1000ms (size-dependent)
Session Deserialization:  50-1000ms (size-dependent)

SQLite Operations:        5-50ms
JSON File I/O:            10-100ms
```

### 11.2 Session Capacity & Limits

```
LIMITS & CONSTRAINTS:

Sessions:
  • Max concurrent: 100
  • Max size (JSON): ~100MB
  • Max message history: ~10,000 messages
  • Timeout: 3600s (configurable)

Agents per session:
  • Default max: 10
  • Configurable up to: 50

Memory entries:
  • Max size per entry: 10MB
  • Default namespace quota: 100MB
  • Max entries per session: 10,000
  • TTL: 7 days (configurable)

Checkpoints:
  • Max per session: 50
  • Auto-cleanup when exceeded
  • Preserves 10 most recent

Database:
  • SQLite file: Can grow to several GB
  • Automatic maintenance recommended
  • Indexes on key columns

File Storage:
  • Session storage: ~/.claude-flow/sessions/
  • Checkpoint storage: ~/.claude-flow/checkpoints/
  • Memory storage: ~/.claude-flow/memory/
```

---

## 12. MIGRATION & UPGRADES

### 12.1 Version Migration

```
MIGRATION PATH:

v1.0.0 → v2.0.0:
├── Add new fields
│   ├── capabilities: []
│   ├── enhanced metadata
│   └── compression support
│
├── Preserve old data
│   ├── All existing entries valid
│   ├── Timestamps converted
│   └── References maintained
│
└── Enable new features
    ├── Compression in new entries
    ├── Advanced serialization
    └── Enhanced hooks

AUTOMATIC MIGRATION:
session.deserializeSessionData()
  ├── Detect version
  ├── If v1: migrate()
  │   ├── Add missing fields
  │   ├── Convert formats
  │   └── Update references
  └── Return v2.0.0 format
```

### 12.2 Upgrade Process

```
UPGRADE STEPS:

1. Backup
   └── Copy ~/.claude-flow/ directory

2. Detect version
   └── Read __session_meta__.version

3. Migrate if needed
   └── Run _migrateSessionData()

4. Validate
   └── Check checksums

5. Test recovery
   └── Try to restore sessions

6. Auto-format new sessions
   └── Use v2.0.0 format
```

---

## 13. SECURITY CONSIDERATIONS

### 13.1 Data Protection

```
SECURITY LAYERS:

ACCESS CONTROL:
• Session ownership (agent-id)
• Memory access levels (private, shared, public)
• Namespace isolation
• Key-based access control

AUTHENTICATION:
• Session token validation
• Auth method support (token, basic, oauth)
• Timing-safe comparison
• Password hashing (SHA-256)

DATA ENCRYPTION:
• Optional encryption for exports
• TLS for network transport
• HTTPS for remote sessions
• At-rest encryption supported

SENSITIVE DATA:
• Redaction hooks available
• Environment variable masking
• Secret detection patterns
• Secure deletion on cleanup
```

### 13.2 Audit Trail

```
AUDIT TRACKING:

Session Events:
├── Creation timestamp
├── Access logs
├── Modification records
└── Deletion confirmation

Memory Operations:
├── Store/retrieve logs
├── Access patterns
├── Size changes
└── User identity

Checkpoint History:
├── Creation events
├── Rollback actions
├── Deletion confirmations
└── Diff audits

Metrics Export:
├── Command execution
├── File modifications
├── Token consumption
└── Performance tracking
```

---

## APPENDIX: FILE LOCATIONS

```
Session & Checkpoint Storage:
.claude-flow/
├── sessions/
│   ├── session-id.json          # Individual session files
│   └── ...
├── checkpoints/
│   ├── message-uuid.json        # Checkpoint files
│   └── ...
├── memory/
│   ├── sqlite-store.db          # In-memory store
│   └── ...
├── swarm-config.json            # Swarm configuration
├── pipeline-config.json         # Pipeline configuration
└── .swarm/
    ├── memory.db                # SQLite memory storage
    └── ...

Database Storage:
~/.claude-flow/
├── claude-flow.db               # SQLite database
    ├── agents table
    ├── tasks table
    └── sessions table
└── claude-flow-data.json        # JSON persistence fallback

Session Metadata:
session-id.json
├── id: string
├── name: string
├── createdAt: ISO timestamp
├── updatedAt: ISO timestamp
├── state: { agents, tasks, memory, config }
└── metadata: { version, platform, checksum }
```

