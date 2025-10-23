# Comprehensive Hooks-Memory Integration Analysis

## Executive Summary

The Claude Flow codebase contains an advanced hooks system that is deeply integrated with memory management. This system provides automatic memory persistence, cross-agent coordination, and pattern learning through a sophisticated hook pipeline architecture.

## System Architecture Overview

### Core Components

1. **Agentic Hook Manager** (`src/services/agentic-flow-hooks/hook-manager.ts`)
   - Central registry and execution engine
   - Pipeline management with error strategies
   - Hook filtering and prioritization
   - Metrics collection and monitoring

2. **Memory Hooks** (`src/services/agentic-flow-hooks/memory-hooks.ts`)
   - Pre/Post store and retrieve hooks
   - Cross-provider synchronization
   - Memory persistence and compression
   - Cache management and pattern detection

3. **Workflow Hooks** (`src/services/agentic-flow-hooks/workflow-hooks.ts`)
   - Workflow lifecycle management
   - Decision tracking and optimization
   - Error analysis and recovery
   - Learning extraction from executions

4. **CLI Hooks Implementation** (`src/cli/simple-commands/hooks.js`)
   - Pre/Post operation hooks
   - Session management hooks
   - MCP integration hooks
   - PreToolUse modification hooks (v2.0.10+)

---

## Memory Hook Integration Patterns

### Memory Hook Types

#### 1. Pre-Memory Store Hook
**Type**: `pre-memory-store`
**Priority**: 100
**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:18-92`

**Function**: Validates and prepares data before storage

**Process Flow**:
```
Input: namespace, key, value, ttl, provider
  ↓
Validation: Check size limits (10MB max), namespace quota
  ↓
Compression: Compress values > 1KB
  ↓
Enrichment: Add metadata (stored, provider, sessionId, size)
  ↓
Metrics: Track memory.usage.{namespace}
  ↓
Output: Modified payload with enriched value
```

**Key Features**:
- Size validation (max 10MB per entry)
- Automatic compression for large values
- Metadata enrichment with timestamps
- Namespace quota checking
- Side effects for metrics tracking

**Example Memory Keys**:
```
pattern: "swarm/[agent]/[step]"
pattern: "workflow:session:{workflowId}:{sessionId}"
pattern: "edit-context:{editId}"
pattern: "learning:{workflowId}:{type}:{timestamp}"
```

#### 2. Post-Memory Store Hook
**Type**: `post-memory-store`
**Priority**: 100
**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:96-156`

**Function**: Synchronizes, indexes, and notifies after storage

**Process Flow**:
```
Stored Data
  ↓
Cross-Provider Sync: If enabled, sync to syncTargets
  ↓
Index Update: Update memory index for search
  ↓
Pattern Detection: Analyze for neural patterns (sequential, temporal)
  ↓
Event Emission: Emit memory:stored event
  ↓
Complete
```

**Side Effects**:
- Memory sync operations to multiple providers
- Memory index updates for search functionality
- Neural pattern analysis (sequential, temporal)
- Event notifications (memory:stored)

#### 3. Pre-Memory Retrieve Hook
**Type**: `pre-memory-retrieve`
**Priority**: 100
**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:160-208`

**Function**: Optimizes retrieval with caching and prefetch

**Process Flow**:
```
Retrieve Request: namespace, key
  ↓
Cache Check: Look in local cache
  If cached → Return with cache hit metric
  ↓
Related Keys: Find related keys for prefetch
  ↓
Prefetch: Background fetch of related keys
  ↓
Metrics: Track retrievals.{namespace}
```

**Optimization Strategies**:
- Local cache lookup (first check)
- Related key prefetching for sequential access patterns
- Cache hit/miss metrics

#### 4. Post-Memory Retrieve Hook
**Type**: `post-memory-retrieve`
**Priority**: 100
**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:212-266`

**Function**: Decompresses, caches, and tracks access patterns

**Process Flow**:
```
Retrieved Data
  ↓
Decompression: If compressed, decompress
  ↓
Access Pattern Tracking: Update access history
  ↓
Local Cache: Store for fast access (keep last 100 accesses)
  ↓
Latency Tracking: Measure and record latency
  ↓
Return Decompressed Value
```

**Metrics Tracked**:
- `memory.decompressions`: Decompress operations
- `memory.latency.{namespace}`: Retrieval latency

#### 5. Memory Sync Hook
**Type**: `memory-sync`
**Priority**: 100
**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:270-364`

**Function**: Manages three synchronization operations

**Operations**:

a) **Sync Operation**: Bidirectional synchronization
```
Detect Changes (namespace, provider)
  ↓
Apply Changes to Sync Targets
  ↓
Update Metrics: memory.sync.changes
```

b) **Persist Operation**: Long-term storage
```
Create Memory Snapshot
  ↓
Store: snapshot:{namespace}:{timestamp}
  ↓
Emit: memory:persisted event
```

c) **Expire Operation**: Cleanup expired entries
```
Find Expired Entries (based on TTL)
  ↓
Remove Entries
  ↓
Update Metrics: memory.expired count
```

#### 6. Memory Persist Hook
**Type**: `memory-persist`
**Priority**: 90
**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:368-418`

**Function**: Creates timestamped backups with checksums

**Backup Format**:
```json
{
  "timestamp": 1634567890000,
  "sessionId": "sess_xxx",
  "namespace": "swarm",
  "entries": [...],
  "size": 12345,
  "checksum": "abc123..."
}
```

**Storage**:
- Key: `backup:{namespace}:{sessionId}`
- TTL: 604800 seconds (7 days)
- Event: `memory:backup:created`

---

## Memory Namespacing Convention

### Namespace Hierarchy

```
hooks:pre-task          → Task preparation data
hooks:pre-edit          → File edit context
hooks:pre-bash          → Command preparation
hooks:post-task         → Task completion data
hooks:post-edit         → Edit results and formatting
hooks:post-bash         → Command results
hooks:post-search       → Search cache
hooks:mcp-initialized   → MCP session data
hooks:agent-spawned     → Agent roster
hooks:task-orchestrated → Orchestration data
hooks:neural-trained    → Neural training results
hooks:notify            → Notifications

agent-assignments       → Agent recommendations (TTL: 3600s)
coordination            → Edit context and decisions
file-history            → File modification history
neural-training         → Neural pattern data
performance-metrics     → Performance data
command-metrics         → Command execution metrics
command-history         → Historical commands
search-cache            → Search results (TTL: 3600s)
session-states          → Full session state
sessions                → Session metadata
agent-roster            → Agent activity log
task-index              → Task index entries
```

### Memory Key Patterns

**Task Keys**:
```
task:{taskId}                           → Task metadata
task:{taskId}:ruv-output               → RUV swarm output
task:index:{timestamp}                 → Task index
task:{taskId}:completed                → Completion data
```

**Edit Keys**:
```
edit:{editId}:pre                      → Pre-edit state
edit:{editId}:post                     → Post-edit results
edit-context:{editId}                  → Edit context (coordination ns)
file-history:{filename}:{timestamp}    → File history entry
agent-recommendation:{file}            → Agent recommendations
```

**Workflow Keys**:
```
workflow:session:{workflowId}:{sessionId}     → Workflow session
workflow:step:{workflowId}:{step}:{timestamp} → Step execution
workflow:history:{workflowId}                 → Execution history
workflow:learnings:{workflowId}               → Extracted learnings
decision:{workflowId}:{point}:{timestamp}     → Decision records
learning:{workflowId}:{type}:{timestamp}      → Learning entries
```

**Performance Keys**:
```
bottleneck:{component}:{timestamp}      → Bottleneck analysis
metrics:{taskId}                         → Task metrics
command-metrics:{bashId}                 → Command metrics
command-metrics-summary                  → Aggregated metrics
provider:health:{provider}               → Provider health status
```

---

## Workflow Hooks and Memory Integration

### Workflow Start Hook
**Type**: `workflow-start`
**File**: `src/services/agentic-flow-hooks/workflow-hooks.ts:20-82`

**Memory Operations**:
```javascript
store: {
  key: `workflow:session:${workflowId}:${context.sessionId}`,
  value: {
    startTime: Date.now(),
    provider: selectedProvider,
    learnings: lastLearnings, // Last 10 learnings
    predictions: generatedPredictions
  },
  ttl: 86400 // 24 hours
}
```

**Execution Flow**:
1. Load workflow history → `workflow:history:{workflowId}`
2. Load past learnings → `workflow:learnings:{workflowId}`
3. Select optimal provider based on historical success rate
4. Generate performance predictions
5. Store enhanced session state

### Workflow Step Hook
**Type**: `workflow-step`
**File**: `src/services/agentic-flow-hooks/workflow-hooks.ts:86-160`

**Memory Operations**:
```javascript
store: {
  key: `workflow:step:${workflowId}:${step}:${Date.now()}`,
  value: {
    step: stepName,
    state: summarizedState,
    timestamp: Date.now()
  },
  ttl: 86400
}
```

### Workflow Decision Hook
**Type**: `workflow-decision`
**File**: `src/services/agentic-flow-hooks/workflow-hooks.ts:164-262`

**Memory Operations**:
```javascript
store: {
  key: `decision:${workflowId}:${decision.point}:${Date.now()}`,
  value: {
    decision: adjustedDecision,
    alternatives: generatedAlternatives,
    state: summarizedState
  },
  ttl: 604800 // 7 days
}
```

**Decision Enhancement**:
1. Load historical outcomes → `outcomes:{workflowId}:{decisionPoint}`
2. Adjust confidence based on historical success rates
3. Generate alternative decision paths
4. Select best alternative if confidence > 20% improvement
5. Store complete decision record with alternatives

### Workflow Complete Hook
**Type**: `workflow-complete`
**File**: `src/services/agentic-flow-hooks/workflow-hooks.ts:266-378`

**Memory Operations**:
```javascript
// Store individual learnings
for (learning of extractedLearnings) {
  store: {
    key: `learning:${workflowId}:${learning.type}:${Date.now()}`,
    value: learning,
    ttl: 0 // Permanent
  }
}

// Neural pattern training
patterns: Pattern[] = {
  id: `workflow_success_${Date.now()}`,
  type: 'success',
  confidence: performanceScore,
  context: {
    workflowId, provider, duration, decisions
  }
}
```

### Workflow Error Hook
**Type**: `workflow-error`
**File**: `src/services/agentic-flow-hooks/workflow-hooks.ts:382-487`

**Error Pattern Analysis**:
```javascript
store: {
  key: `error:${workflowId}:${Date.now()}`,
  value: {
    error: { message, stack, type },
    pattern: errorPattern,
    state: summarizedState
  },
  ttl: 604800
}

// If recurring error (>5 occurrences):
store: {
  key: `learning:failure:${workflowId}:${Date.now()}`,
  value: failureLearning,
  ttl: 0 // Permanent
}
```

---

## Hook Execution Flow with Memory

### Memory Operation Pipeline

**File**: `src/services/agentic-flow-hooks/index.ts:95-117`

```
┌─────────────────────────────────────────┐
│  Memory Operation Pipeline              │
├─────────────────────────────────────────┤
│                                         │
│  Stage 1: Validation (Sequential)       │
│  └─ pre-memory-store hooks              │
│     • Check size limits                  │
│     • Validate namespace quota           │
│     • Verify key format                  │
│                                         │
│  Stage 2: Storage (Parallel)            │
│  └─ post-memory-store hooks             │
│     • Cross-provider sync                │
│     • Index update                       │
│     • Pattern detection                  │
│                                         │
│  Stage 3: Sync (Conditional Parallel)   │
│  └─ memory-sync hooks                   │
│     IF crossProvider === true:           │
│     • Bidirectional sync                 │
│     • Conflict resolution                │
│                                         │
└─────────────────────────────────────────┘
```

### CLI Hook Integration

**File**: `src/cli/simple-commands/hooks.js`

#### Pre-Task Hook Flow (Lines 111-211)
```
pre-task command
  ├─ Generate taskId
  ├─ Store to: namespace: 'hooks:pre-task'
  │           key: `task:${taskId}`
  ├─ Create index: namespace: 'task-index'
  │               key: `task-index:${Date.now()}`
  ├─ Attempt RUV swarm hook
  └─ Store RUV output: namespace: 'hooks:ruv-swarm'
```

**Memory Namespace**: `hooks:pre-task`
**Memory Keys**:
- `task:{taskId}` - Task metadata
- `task-index:{timestamp}` - Index entry
- `task:{taskId}:ruv-output` - RUV integration output

#### Post-Edit Hook Flow (Lines 480-658)
```
post-edit command
  ├─ Auto-format if requested
  │  └─ Store: namespace: 'hooks:post-edit'
  ├─ Update memory if enabled
  │  └─ Store: namespace: 'coordination'
  │           key: `edit-context:{editId}`
  ├─ Train neural patterns
  │  └─ Store: namespace: 'neural-training'
  │           key: `neural-pattern:{patternId}`
  └─ Update file history
     └─ Store: namespace: 'file-history'
              key: `file-history:{filename}:{timestamp}`
```

**Memory Namespaces**:
- `hooks:post-edit` - Edit results
- `coordination` - Edit context (for cross-agent access)
- `neural-training` - Pattern data
- `file-history` - Modification history

#### Session End Hook Flow (Lines 963-1083)
```
session-end command
  ├─ Gather task index (limit: 1000)
  ├─ Gather file history
  ├─ Gather command history
  ├─ Gather agent roster
  ├─ Calculate session metrics
  ├─ Store session: namespace: 'sessions'
  │                 key: `session:{sessionId}`
  ├─ Persist detailed state (if enabled)
  │  └─ Store: namespace: 'session-states'
  └─ Export metrics (if enabled)
     └─ Store: namespace: 'session-metrics'
```

**Metrics Stored**:
```javascript
{
  sessionDuration: milliseconds,
  totalTasks: number,
  totalEdits: number,
  totalCommands: number,
  uniqueAgents: number,
  commandSuccessRate: percentage,
  avgTasksPerMinute: number,
  avgEditsPerMinute: number,
  timestamp: ISO string
}
```

#### Session Restore Hook Flow (Lines 1085-1129)
```
session-restore command
  ├─ Retrieve latest session OR specific sessionId
  │  └─ Retrieve from: namespace: 'sessions'
  ├─ Load restoration event
  │  └─ Store: namespace: 'session-events'
  │           key: `session-restore:{timestamp}`
  └─ Display restored session data
```

---

## Automatic Memory Persistence Mechanisms

### 1. Side Effect Processing

**File**: `src/services/agentic-flow-hooks/hook-manager.ts:178-181`

Hook handlers return side effects that are automatically processed:

```javascript
interface SideEffect {
  type: 'memory' | 'neural' | 'metric' | 'notification' | 'log';
  action: string;
  data: any;
}
```

**Side Effect Types**:

| Type | Action | Purpose |
|------|--------|---------|
| memory | store | Automatic memory persistence |
| memory | sync | Cross-provider synchronization |
| neural | analyze | Pattern detection |
| neural | train | Neural model training |
| metric | increment | Counter metrics |
| metric | update | Gauge metrics |
| notification | emit | Event broadcasting |
| log | write | Logging |

### 2. Automatic Side Effect Processing

**File**: `src/services/agentic-flow-hooks/hook-manager.ts:179-181`

```typescript
if (result.sideEffects) {
  await this.processSideEffects(result.sideEffects, context);
}
```

**Execution**: Side effects are processed sequentially after each hook result.

### 3. Memory Store Operations

**Memory Store Side Effect**:
```javascript
sideEffects.push({
  type: 'memory',
  action: 'store',
  data: {
    key: 'unique-memory-key',
    value: dataToStore,
    ttl: 3600  // Optional TTL in seconds
  }
})
```

**Automatic Processing**:
1. Validates memory constraints
2. Compresses if > 1KB
3. Adds metadata (timestamp, sessionId)
4. Triggers pre-memory-store hooks
5. Stores to persistent backend
6. Triggers post-memory-store hooks
7. Updates memory indices

---

## Cross-Agent Memory Coordination

### Memory Coordination Pattern

**File**: `src/task/coordination.ts`

Agents coordinate through memory namespaces:

```javascript
// Agent 1 stores decision
await memoryStore.store('swarm/agent1/decision', {
  type: 'architecture',
  decision: 'Use microservices',
  timestamp: Date.now(),
  dependencies: ['database-design']
}, { 
  namespace: 'coordination'
});

// Agent 2 retrieves and builds on decision
const decision = await memoryStore.retrieve('swarm/agent1/decision', {
  namespace: 'coordination'
});

// Agent 2 stores its implementation
await memoryStore.store('swarm/agent2/implementation', {
  basedOn: 'swarm/agent1/decision',
  implementation: 'Service configuration',
  timestamp: Date.now()
}, {
  namespace: 'coordination'
});
```

### Hook-Based Coordination

**Pre-Task Hooks** load coordination data:
```bash
npx claude-flow@alpha hooks pre-task --description "Task" --load-memory true
```

**Post-Edit Hooks** update coordination data:
```bash
npx claude-flow@alpha hooks post-edit --file "file.js" --memory-key "swarm/agent/step"
```

**Session Hooks** persist coordination across sessions:
```bash
npx claude-flow@alpha hooks session-end --export-metrics true
```

---

## Memory Key Naming Conventions

### Hierarchical Structure

```
[namespace]:[category]:[subcategory]:[identifier]:[timestamp]

Examples:
  hooks:pre-task:init
  swarm:agent-name:decision:step-1
  coordination:edit:file-path:edit-id
  neural-training:pattern:model-id:timestamp
  performance:metrics:task-id
  workflow:session:workflow-id:session-id:timestamp
```

### Recommended Patterns

**For Swarm Coordination**:
```
swarm/{swarmId}/agent/{agentName}/{step}
swarm/{swarmId}/memory/{dataType}/{timestamp}
swarm/{swarmId}/decision/{point}/{timestamp}
```

**For Workflow Tracking**:
```
workflow:{workflowId}:session:{sessionId}
workflow:step:{workflowId}:{step}:{timestamp}
workflow:decision:{workflowId}:{point}:{timestamp}
```

**For File Operations**:
```
edit-context:{editId}
file-history:{filename}:{timestamp}
agent-recommendation:{filepath}
```

**For Learning/Patterns**:
```
learning:{workflowId}:{type}:{timestamp}
neural-pattern:{patternId}
pattern:optimization:{timestamp}
```

---

## Hook Configuration and Registry

### Hook Registration

**File**: `src/services/agentic-flow-hooks/hook-manager.ts:53-84`

```typescript
register(registration: HookRegistration): void {
  // Validate registration
  // Check for duplicate IDs
  // Insert sorted by priority (higher first)
  // Emit hook:registered event
  // Update metrics
}
```

### Default Pipelines

**File**: `src/services/agentic-flow-hooks/index.ts:69-144`

**1. LLM Call Pipeline**:
- Stage 1: pre-llm-call (sequential)
- Stage 2: call-execution
- Stage 3: post-llm-call (parallel)
- Error Strategy: continue

**2. Memory Operation Pipeline**:
- Stage 1: validation (pre-memory-store)
- Stage 2: storage (post-memory-store)
- Stage 3: sync (conditional parallel)
- Error Strategy: rollback

**3. Workflow Execution Pipeline**:
- Stage 1: initialization (workflow-start)
- Stage 2: execution (workflow-step, workflow-decision)
- Stage 3: completion (workflow-complete)
- Error Strategy: fail-fast

---

## CLI Integration for Hooks

### Hooks Command Structure

**File**: `src/cli/simple-commands/hooks.js:26-107`

**Commands**:
```bash
# Pre-Operation Hooks
npx claude-flow hooks pre-task [options]
npx claude-flow hooks pre-edit [options]
npx claude-flow hooks pre-bash [options]

# Post-Operation Hooks
npx claude-flow hooks post-task [options]
npx claude-flow hooks post-edit [options]
npx claude-flow hooks post-bash [options]
npx claude-flow hooks post-search [options]

# MCP Integration Hooks
npx claude-flow hooks mcp-initialized [options]
npx claude-flow hooks agent-spawned [options]
npx claude-flow hooks task-orchestrated [options]
npx claude-flow hooks neural-trained [options]

# Session Hooks
npx claude-flow hooks session-end [options]
npx claude-flow hooks session-restore [options]
npx claude-flow hooks notify [options]

# PreToolUse Modification Hooks (v2.0.10+)
npx claude-flow hooks modify-bash [stdin/stdout JSON]
npx claude-flow hooks modify-file [stdin/stdout JSON]
npx claude-flow hooks modify-git-commit [stdin/stdout JSON]
```

### Hook Memory Store

**Implementation**: `src/cli/simple-commands/hooks.js:10-19`

```javascript
const memoryStore = new SqliteMemoryStore();
await memoryStore.initialize();

// All hooks persist to .swarm/memory.db
```

**Database Location**: `.swarm/memory.db` (SQLite)

---

## Pattern Learning and Neural Integration

### Pattern Detection in Hooks

**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:497-526`

```javascript
// Sequential pattern detection
const accessHistory = await getAccessHistory(namespace, context);
if (isSequentialPattern(accessHistory)) {
  patterns.push({
    type: 'sequential',
    confidence: 0.8,
    suggestion: 'prefetch-next'
  });
}

// Temporal pattern detection
if (isTemporalPattern(accessHistory)) {
  patterns.push({
    type: 'temporal',
    confidence: 0.7,
    suggestion: 'cache-duration'
  });
}
```

### Neural Pattern Training

**File**: `src/services/agentic-flow-hooks/workflow-hooks.ts:304-327`

```javascript
const pattern: Pattern = {
  id: `workflow_success_${Date.now()}`,
  type: 'success',
  confidence: performance.score,
  occurrences: 1,
  context: {
    workflowId,
    provider: state.provider,
    duration: metrics?.duration || 0,
    decisions: countDecisions(state),
  },
};

context.neural.patterns.add(pattern);

sideEffects.push({
  type: 'neural',
  action: 'train',
  data: {
    patterns: [pattern],
    modelId: `workflow-optimizer-${workflowId}`,
  },
});
```

### Memory Keys for Neural Data

```
neural-pattern:{patternId}           → Pattern metadata
neural-training:{timestamp}          → Training logs
pattern:optimization:{timestamp}     → Optimization suggestions
pattern:sequential:{namespace}       → Sequential access patterns
pattern:temporal:{namespace}         → Temporal patterns
```

---

## Performance Tracking Through Hooks

### Performance Metrics Side Effects

**File**: `src/services/agentic-flow-hooks/memory-hooks.ts:73-80`

```javascript
sideEffects.push({
  type: 'metric',
  action: 'update',
  data: {
    name: `memory.usage.${namespace}`,
    value: getValueSize(enrichedValue),
  },
});
```

### Tracked Metrics

| Metric | Type | Trigger |
|--------|------|---------|
| memory.compressions | increment | compress value |
| memory.decompressions | increment | decompress value |
| memory.cache.hits | increment | cache hit |
| memory.retrievals.{namespace} | increment | retrieve |
| memory.latency.{namespace} | update | retrieve latency |
| memory.sync.changes | update | sync operation |
| memory.expired | update | cleanup |
| hooks.executions | increment | hook execution |
| hooks.{type}.executions | increment | specific hook |
| hooks.{type}.duration | update | execution time |

---

## Complete Hook Execution Example

### Scenario: Post-Edit Hook with Full Memory Integration

**File**: `src/cli/simple-commands/hooks.js:480-658`

```javascript
// User edits a file
npx claude-flow hooks post-edit --file "src/auth.js" \
  --format true \
  --update-memory true \
  --train-neural true \
  --memory-key "auth/login-implementation"
```

**Execution Chain**:

1. **Validation** (Pre-Edit)
   - Check file exists
   - Determine file type
   - Check memory store

2. **Format** (if enabled)
   - Determine formatter (prettier for JS)
   - Store result: `namespace: 'hooks:post-edit'`

3. **Memory Update** (if enabled)
   - Create edit context:
     ```javascript
     {
       file: "src/auth.js",
       editedAt: ISO_TIMESTAMP,
       editId: "edit-xxx",
       formatted: true,
       fileSize: 2048,
       directory: "src",
       basename: "auth.js"
     }
     ```
   - Store to: `namespace: 'coordination'`, `key: 'edit-context:{editId}'`
   - This triggers post-memory-store hooks (cross-provider sync, pattern detection, etc.)

4. **Neural Training** (if enabled)
   - Create pattern:
     ```javascript
     {
       fileType: ".js",
       fileName: "auth.js",
       editTime: ISO_TIMESTAMP,
       confidence: 0.75,
       patterns: [
         ".js_edit_pattern",
         "auth.js_modification",
         "edit_xxx_sequence"
       ]
     }
     ```
   - Store to: `namespace: 'neural-training'`

5. **File History**
   - Store to: `namespace: 'file-history'`
   - Key: `file-history:src_auth.js:{timestamp}`

6. **Return Results**
   ```json
   {
     "file": "src/auth.js",
     "formatted": true,
     "formatterUsed": "prettier",
     "updateMemory": true,
     "trainNeural": true,
     "formatResult": {...},
     "memoryUpdate": {...},
     "neuralTraining": {...}
   }
   ```

---

## File Reference Index

### Core Hook Files

| File | Purpose | Key Functions |
|------|---------|-----------------|
| `/src/services/agentic-flow-hooks/hook-manager.ts` | Central hook registry and execution | register, executeHooks, createPipeline |
| `/src/services/agentic-flow-hooks/types.ts` | Type definitions | AgenticHookContext, MemoryContext, all hook types |
| `/src/services/agentic-flow-hooks/memory-hooks.ts` | Memory persistence hooks | preMemoryStore, postMemoryStore, memorySyncHook |
| `/src/services/agentic-flow-hooks/workflow-hooks.ts` | Workflow lifecycle hooks | workflowStart, workflowDecision, workflowComplete |
| `/src/services/agentic-flow-hooks/index.ts` | System initialization | initializeAgenticFlowHooks, setupDefaultPipelines |
| `/src/cli/simple-commands/hooks.js` | CLI hook implementations | preTaskCommand, postEditCommand, sessionEndCommand |

### Configuration Files

| File | Purpose |
|------|---------|
| `/src/cli/simple-commands/init/templates/commands/hooks/pre-task.md` | Pre-task documentation |
| `/src/cli/simple-commands/init/templates/commands/hooks/post-edit.md` | Post-edit documentation |
| `/src/cli/simple-commands/init/templates/commands/hooks/session-restore.md` | Session restore documentation |

### Related Memory Files

| File | Purpose |
|------|---------|
| `/src/swarm/memory.ts` | Distributed memory system |
| `/src/memory/swarm-memory.ts` | Swarm-specific memory |
| `/src/hive-mind/core/Memory.ts` | Memory core implementation |
| `/src/task/coordination.ts` | Cross-agent coordination |

### Legacy Compatibility

| File | Purpose |
|------|---------|
| `/src/hooks/index.ts` | Legacy hook system (migration redirects) |
| `/src/verification/hooks.ts` | Verification system hooks |

---

## Summary

The hooks-memory integration system provides:

1. **Automatic Memory Persistence**: All hook executions trigger memory operations through side effects
2. **Cross-Provider Synchronization**: Memory sync hooks coordinate state across multiple providers
3. **Pattern Learning**: Neural hooks extract patterns from memory operations
4. **Session Continuity**: Session hooks enable context preservation across CLI invocations
5. **Performance Tracking**: Metrics hooks collect comprehensive performance data
6. **Agent Coordination**: Shared memory namespaces enable cross-agent communication
7. **Workflow Optimization**: Decision hooks leverage historical memory to improve selections
8. **Automatic Compression**: Large values are transparently compressed/decompressed
9. **Cache Optimization**: Pre-fetch and cache strategies based on access patterns
10. **Error Recovery**: Error patterns are analyzed and recovery strategies stored

This system enables sophisticated coordination across multiple agents and sessions while maintaining transparent persistence and performance optimization.

