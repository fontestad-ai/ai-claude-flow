# Memory Management Architecture Analysis - ai-claude-flow

## Executive Summary

The ai-claude-flow project implements a **sophisticated, multi-tier memory management system** with persistent storage, intelligent caching, and advanced optimization techniques. The system supports both local SQLite persistence and fallback in-memory storage, with comprehensive memory scoping across agents, swarms, and sessions.

---

## 1. MEMORY STORAGE MECHANISMS

### 1.1 Primary Storage Tiers

#### Tier 1: High-Performance LRU Cache (In-Memory)
- **Location**: `/src/hive-mind/core/Memory.ts` (lines 23-116)
- **Implementation**: `HighPerformanceCache<T>` class
- **Characteristics**:
  - Maximum size: 10,000 entries (configurable)
  - Maximum memory: 100 MB (configurable)
  - Implements LRU (Least Recently Used) eviction
  - Hit/miss tracking and statistics
  - Memory pressure handling

**Key Features**:
```typescript
class HighPerformanceCache<T> {
  - get(key): T | undefined         // O(1) with LRU update
  - set(key, data): void             // O(1) with size checking
  - evictLRU(): void                 // Evicts least recently used
  - getStats(): CacheStats           // Hit rate, utilization %
}
```

#### Tier 2: SQLite Database (Persistent)
- **Location**: `/src/memory/sqlite-store.js`
- **Backend**: better-sqlite3 driver
- **Storage Directory**: `.swarm/` (project root)
- **Database File**: `memory.db` or `unified-memory.db`

**Database Configuration**:
- **WAL Mode**: Enabled for better concurrency
- **Synchronous Mode**: NORMAL (balance between safety and speed)
- **Cache Size**: 1000 pages
- **Temp Store**: In-memory

**Schema** (from `/src/memory/shared-memory.js` lines 18-81):
```sql
CREATE TABLE memory_store (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  key TEXT NOT NULL,
  namespace TEXT NOT NULL DEFAULT 'default',
  value TEXT NOT NULL,
  type TEXT NOT NULL DEFAULT 'json',
  metadata TEXT,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  accessed_at INTEGER NOT NULL,
  access_count INTEGER NOT NULL DEFAULT 0,
  ttl INTEGER,
  expires_at INTEGER,
  compressed INTEGER DEFAULT 0,
  size INTEGER NOT NULL DEFAULT 0,
  tags TEXT,
  UNIQUE(key, namespace)
);

-- Performance Indexes
CREATE INDEX idx_memory_namespace ON memory_store(namespace);
CREATE INDEX idx_memory_expires ON memory_store(expires_at);
CREATE INDEX idx_memory_accessed ON memory_store(accessed_at);
CREATE INDEX idx_memory_key_namespace ON memory_store(key, namespace);
CREATE INDEX idx_memory_tags ON memory_store(tags);
```

#### Tier 3: JSON File Fallback
- **Location**: `/src/memory/unified-memory-manager.js`
- **Path**: `memory/memory-store.json` (project root)
- **Purpose**: Fallback when SQLite unavailable (npx environments)
- **Format**: JSON with namespace organization

**Example Structure** (from `/memory/memory-store.json`):
```json
{
  "default": [
    {
      "key": "project-name",
      "value": "claude-flow-demo",
      "namespace": "default",
      "timestamp": 1758288473182
    }
  ],
  "swarm": [...],
  "sessions": [...]
}
```

#### Tier 4: In-Memory Fallback (Runtime)
- **Location**: `/src/memory/in-memory-store.js`
- **Implementation**: Map-based storage
- **Use Case**: NPX environments without native module support
- **Data Structure**: `Map<namespace, Map<key, entry>>`
- **Cleanup**: Automatic every 60 seconds for expired entries

---

## 2. DATA STRUCTURES FOR MEMORY MANAGEMENT

### 2.1 Memory Entry Structure
**From**: `/src/memory/advanced-memory-manager.ts` (lines 16-36)

```typescript
interface MemoryEntry {
  id: string;
  key: string;
  value: any;
  type: string;
  namespace: string;
  tags: string[];
  metadata: Record<string, any>;
  owner: string;
  accessLevel: 'private' | 'shared' | 'public';
  createdAt: Date;
  updatedAt: Date;
  lastAccessedAt: Date;
  expiresAt?: Date;
  version: number;
  size: number;
  compressed: boolean;
  checksum: string;
  references: string[];
  dependencies: string[];
}
```

### 2.2 Memory Index Structure
**From**: `/src/memory/indexer.ts` (lines 8-49)

```typescript
// Fast lookup indexes
- agentIndex: SimpleIndex<string>    // Agent ID -> Entry IDs
- sessionIndex: SimpleIndex<string>  // Session ID -> Entry IDs
- typeIndex: SimpleIndex<string>     // Type -> Entry IDs
- tagIndex: SimpleIndex<string>      // Tag -> Entry IDs
- timeIndex: Map<string, number>     // Entry ID -> Timestamp

// SimpleIndex implementation
class SimpleIndex<T> {
  private index = new Map<T, Set<string>>();
  get(key: T): Set<string>;
  add(key: T, entryId: string): void;
  remove(key: T, entryId: string): void;
}
```

### 2.3 TTL Map for Automatic Expiration
**From**: `/src/swarm/optimizations/ttl-map.ts` (lines 6-59)

```typescript
interface TTLItem<T> {
  value: T;
  expiry: number;
  createdAt: number;
  accessCount: number;
  lastAccessedAt: number;
}

class TTLMap<K, V> {
  private items = new Map<K, TTLItem<V>>();
  
  set(key: K, value: V, ttl?: number): void
  get(key: K): V | undefined
  delete(key: K): boolean
  startCleanup(): void
  stopCleanup(): void
  
  // Stats tracking
  stats: {
    hits: number;
    misses: number;
    evictions: number;
    expirations: number;
  }
}
```

### 2.4 Object Pool for Memory Reuse
**From**: `/src/hive-mind/core/Memory.ts` (lines 121-159)

```typescript
class ObjectPool<T> {
  private pool: T[] = [];
  
  acquire(): T                    // Get object from pool or create
  release(obj: T): void           // Return object to pool
  getStats()                      // Reuse rate, allocation count
}

// Pools initialized for:
- memoryEntry: MemoryEntry objects
- searchResult: Search result objects
```

---

## 3. MEMORY LIFECYCLE

### 3.1 Creation Phase

**Storage Operation** (from `/src/hive-mind/core/Memory.ts` lines 298-380):
```typescript
async store(key: string, value: any, namespace: string, ttl?: number) {
  1. Smart serialization with compression detection
  2. Database storage with transaction
  3. Async MCP storage (non-blocking)
  4. Cache update (high-performance LRU)
  5. Access pattern tracking
  6. Namespace stats update
  7. Event emission
}
```

**Metadata Added**:
- `swarmId`: Swarm identifier
- `compressed`: Compression flag
- `originalSize`: Pre-compression size
- `timestamp`: Creation time
- `provider`: Memory provider type

### 3.2 Update/Access Phase

**Retrieval Operation** (from `/src/hive-mind/core/Memory.ts` lines 417-477):
```typescript
async retrieve(key: string, namespace: string) {
  1. Check high-performance cache first (L1)
  2. Database lookup with prepared statements (L2)
  3. Decompression if needed
  4. Parse value (JSON or string)
  5. Async cache update
  6. Update access statistics
  7. Fallback to MCP memory (non-blocking)
}
```

**Access Tracking**:
- `accessCount`: Incremented on each access
- `lastAccessedAt`: Updated to current time
- `accessPattern`: Weighted by operation type
  - `cache_hit`: 0.5 weight
  - `db_hit`: 1.0 weight
  - `write`: 2.0 weight
  - `miss`: 0.1 weight

### 3.3 Expiry/Cleanup Phase

**TTL-Based Expiration** (from `/src/memory/in-memory-store.js`):
```typescript
- TTL field: Time-to-live in seconds
- expiresAt: Calculated as createdAt + (ttl * 1000)
- Cleanup interval: Every 60 seconds (configurable)
- Garbage collection: Every 5 minutes
```

**Eviction Strategies**:
1. **LRU Eviction**: When cache size exceeded
2. **TTL Expiration**: Automatic cleanup of aged entries
3. **Access-based**: Remove entries with low access count
4. **Compression-based**: Compress old, large, rarely-accessed entries

**Memory Cleanup Trigger** (from `/src/hive-mind/core/Memory.ts` lines 872-918):
```typescript
// Runs every 5 minutes
async performMemoryCleanup() {
  1. Evict expired entries from database
  2. Optimize object pools
  3. Clean up old access patterns (< 0.5 weight)
  4. Namespace optimization
  5. Database maintenance
}
```

### 3.4 Deletion Phase

**Hard Delete Operation** (from `/src/hive-mind/core/Memory.ts` lines 607-624):
```typescript
async delete(key: string, namespace: string) {
  1. Remove from LRU cache
  2. Remove from database
  3. Remove from MCP memory
  4. Emit deletion event
}
```

---

## 4. MEMORY SCOPING

### 4.1 Namespace Organization

**Predefined Namespaces** (from `/src/memory/index.js` lines 15-23):
```javascript
SWARM_NAMESPACES = {
  AGENTS: 'swarm:agents',
  TASKS: 'swarm:tasks',
  COMMUNICATIONS: 'swarm:communications',
  CONSENSUS: 'swarm:consensus',
  PATTERNS: 'swarm:patterns',
  METRICS: 'swarm:metrics',
  COORDINATION: 'swarm:coordination',
}
```

**Default Namespaces** (from `/src/hive-mind/core/Memory.ts` lines 788-830):
```typescript
1. 'default': Default memory (persistent, max 10,000 entries)
2. 'task-results': Task results (TTL: 7 days)
3. 'agent-state': Agent state (TTL: 1 day)
4. 'learning-data': ML training data (persistent, max 50,000)
5. 'performance-metrics': Performance data (TTL: 30 days)
6. 'decisions': Strategic decisions (persistent, max 10,000)
```

### 4.2 Memory Access Levels

**From**: `/src/memory/advanced-memory-manager.ts` (line 25)
```typescript
accessLevel: 'private' | 'shared' | 'public'

- private: Only accessible to owner/agent
- shared: Accessible to team/swarm members
- public: Globally accessible
```

### 4.3 Agent-Level Memory

**Agent Memory Management** (from `/src/memory/swarm-memory.ts` lines 63-86):
```typescript
async storeAgent(agentId, agentData) {
  - Key pattern: 'agent:{agentId}'
  - Namespace: 'swarm:agents'
  - Tags: ['agent', agentType, agentStatus]
  - Cache: Agent-specific cache maintained
  - Metadata: swarmId, agentType tracking
}
```

### 4.4 Swarm-Level Memory

**SwarmMemory Class** (from `/src/memory/swarm-memory.js` lines 27-86):
- Extends SharedMemory
- Maintains agent cache
- Task cache
- Pattern cache
- Cross-agent sharing capabilities

### 4.5 Session-Level Memory

**Session Management** (from `/src/memory/enhanced-memory.js` lines 37-62):
```typescript
async saveSessionState(sessionId, state) {
  - Key: 'session:{sessionId}'
  - Namespace: 'sessions'
  - Tracks: userId, projectPath, activeBranch, lastActivity
  - Context and environment preservation
}
```

---

## 5. SERIALIZATION & DESERIALIZATION

### 5.1 Advanced TypeScript-Aware Serialization

**Location**: `/src/memory/advanced-serializer.js` (lines 12-77)

**Supported Types**:
- Date, Map, Set, RegExp, Error, BigInt
- ArrayBuffer, TypedArray
- Function, Symbol (with restrictions)
- Custom serializers (extensible)

**Serialization Markers**:
```javascript
__Date__: ISO string representation
__Map__: Entries array
__Set__: Values array
__RegExp__: Pattern and flags
__Error__: Message and stack
__BigInt__: String representation
__ArrayBuffer__: Base64 encoded
__TypedArray__: Type and values
__Function__: Source code (if allowed)
__Symbol__: Description
__undefined__: Special marker
__NaN__: Special marker
__Infinity__: Special marker
```

### 5.2 Session Serialization

**Location**: `/src/memory/enhanced-session-serializer.js` (lines 35-60)

**Features**:
- Compression support (threshold: 1 KB)
- Validation and migration support
- Version tracking (currently v2.0.0)
- Node version and platform info preservation

**Process**:
```typescript
1. Preprocess session data
2. Add session metadata
3. Serialize with advanced serializer
4. Optional compression if > 1 KB
5. Return serialized string
```

**Deserialization**:
```typescript
1. Deserialize with advanced serializer
2. Validate metadata
3. Version migration if needed
4. Remove metadata
5. Return processed data
```

### 5.3 Compression Strategy

**From**: `/src/services/agentic-flow-hooks/memory-hooks.ts` (lines 49-58)

**Compression Decision**:
```typescript
shouldCompress(value) {
  - Threshold: 1 KB default
  - Compresses: Large JSON objects, nested structures
  - Returns: Compressed flag, original size tracking
}

compressValue(value) {
  - Uses: zlib in production (placeholder in code)
  - Stores: { compressed: true, data: string }
  - Tracks: compression ratio
}
```

---

## 6. MEMORY PERSISTENCE

### 6.1 Database Optimization Settings

**From**: `/src/hive-mind/core/Memory.ts` (lines 285-293)

```typescript
// WAL Mode: Write-Ahead Logging for better concurrency
db.pragma('journal_mode = WAL')
db.pragma('synchronous = NORMAL')
db.pragma('cache_size = 1000')
db.pragma('temp_store = memory')
```

### 6.2 Backup & Restore

**Backup Operation** (from `/src/hive-mind/core/Memory.ts` lines 735-755):
```typescript
async backup(path: string) {
  1. Get all memory entries
  2. Create backup object with:
     - swarmId, timestamp, entries
     - namespaces array
     - learned patterns
  3. Store via MCP with key: 'backup/{swarmId}/{timestamp}'
  4. TTL: 7 days retention
  5. Emit backup:created event
}
```

**Restore Operation** (from `/src/hive-mind/core/Memory.ts` lines 760-783):
```typescript
async restore(backupId: string) {
  1. Retrieve backup from MCP
  2. Clear existing memory
  3. Clear cache
  4. Restore each entry
  5. Emit memory:restored event
}
```

### 6.3 Memory Snapshots

**Snapshot Creation** (from `/src/services/agentic-flow-hooks/memory-hooks.ts` lines 608-620):
```typescript
async createMemorySnapshot(namespace) {
  1. Capture current namespace state
  2. Create snapshot with: timestamp, entries, size
  3. Store as: 'snapshot:{namespace}:{timestamp}'
  4. No expiration (persistent)
  5. Used for disaster recovery
}
```

### 6.4 Export/Import Capabilities

**From**: `/src/memory/advanced-memory-manager.ts` (lines 73-97)

```typescript
export interface ExportOptions {
  format: 'json' | 'csv' | 'xml' | 'yaml';
  namespace?: string;
  type?: string;
  includeMetadata?: boolean;
  compression?: boolean;
  encryption?: {
    enabled: boolean;
    algorithm?: string;
    key?: string;
  };
  filtering?: QueryOptions;
}

import interface ImportOptions {
  format: 'json' | 'csv' | 'xml' | 'yaml';
  namespace?: string;
  conflictResolution: 'overwrite' | 'skip' | 'merge' | 'rename';
  validation?: boolean;
  transformation?: {
    keyMapping?: Record<string, string>;
    valueTransformation?: (value: any) => any;
    metadataExtraction?: (entry: any) => Record<string, any>;
  };
  dryRun?: boolean;
}
```

---

## 7. MEMORY OPTIMIZATION TECHNIQUES

### 7.1 Caching Strategy

**Multi-Level Caching**:
1. **L1 Cache**: High-Performance LRU (in-process, ~100ms latency)
2. **L2 Cache**: SQLite Database (disk-based, ~5-10ms with indexes)
3. **L3 Cache**: MCP Memory (distributed, ~50-100ms)

**Cache Configuration** (from `/src/hive-mind/core/Memory.ts` lines 188-189):
```typescript
cacheSize: 10000         // Max entries
cacheMemoryMB: 100       // Max memory in MB
```

### 7.2 Compression Optimization

**Automatic Compression** (from `/src/hive-mind/core/Memory.ts` lines 317-320):
```typescript
if (serializedValue.length > this.compressionThreshold) {
  serializedValue = await this.compressData(serializedValue);
  compressed = true;
}
```

**Compression Threshold**: 10 KB default (from `/src/hive-mind/core/Memory.ts` line 172)

**Selective Compression Logic** (from `/src/hive-mind/core/Memory.ts` lines 1244-1252):
```typescript
shouldCompress(entry) {
  const ageInDays = (Date.now() - entry.createdAt.getTime()) / (1000 * 60 * 60 * 24);
  const isOld = ageInDays > 7;
  const isLarge = entry.value.length > 10000;
  const isRarelyAccessed = entry.accessCount < 5;
  
  return isOld && isLarge && isRarelyAccessed;
}
```

### 7.3 Pattern Analysis & Learning

**Access Pattern Tracking** (from `/src/hive-mind/core/Memory.ts` lines 1037-1068):
```typescript
updateAccessPattern(key, operation) {
  // Weight by operation type
  weight = {
    cache_hit: 0.5,
    db_hit: 1.0,
    write: 2.0,
    miss: 0.1
  }[operation]
  
  // Limit patterns size to 10,000
  // Remove least accessed when exceeded
}
```

**Pattern Learning** (from `/src/hive-mind/core/Memory.ts` lines 668-702):
```typescript
async learnPatterns() {
  1. Get top 20 accessed keys
  2. Identify co-access patterns (correlation)
  3. Train neural patterns via MCP
  4. Emit patterns:analyzed event
}
```

### 7.4 Garbage Collection

**Periodic GC Configuration** (from `/src/hive-mind/core/Memory.ts` lines 859-883):
```typescript
// Cache optimization: every 30 seconds
// Metrics update: every 10 seconds
// Memory cleanup: every 5 minutes
// Pattern analysis: every 2 minutes

startOptimizedManagers() {
  setInterval(() => optimizeCache(), 30000);
  setInterval(() => updatePerformanceMetrics(), 10000);
  setInterval(() => performMemoryCleanup(), 300000);
  setInterval(() => analyzeAccessPatterns(), 120000);
}
```

### 7.5 Object Pool Optimization

**Pool Statistics** (from `/src/hive-mind/core/Memory.ts` lines 151-158):
```typescript
getStats() {
  return {
    poolSize: this.pool.length,
    allocated: this.allocated,
    reused: this.reused,
    reuseRate: (this.reused / (this.allocated + this.reused)) * 100
  };
}
```

**Pool Optimization Alerts** (from `/src/hive-mind/core/Memory.ts` lines 1104-1112):
```typescript
optimizeObjectPools() {
  if (stats.reuseRate < 30 && stats.poolSize < 500) {
    // Alert: pool might be too small
    emit('poolOptimizationSuggested', { name, stats });
  }
}
```

### 7.6 Performance Monitoring

**Metrics Tracked** (from `/src/hive-mind/core/Memory.ts` lines 1073-1099):
```typescript
updatePerformanceMetrics() {
  // For each operation (store, retrieve, search, etc.):
  - avg: average duration
  - count: total operations
  - max: maximum duration
  - min: minimum duration
  
  // Cache statistics:
  - size: current entries
  - hitRate: percentage
  - memoryUsage: bytes
  - evictions: count
  
  // Pool statistics:
  - poolSize, allocated, reused, reuseRate
  
  // Access patterns:
  - hotKeys: top 10 accessed
  - total: pattern count
}
```

### 7.7 Memory Health Checks

**Health Status** (from `/src/hive-mind/core/Memory.ts` lines 1387-1436):
```typescript
async healthCheck() {
  // Score calculation (0-100):
  - Cache hit rate < 50: -20 points
  - Memory utilization > 90: -30 points (warning)
  - DB retrieval > 100ms: -15 points
  - Low pool reuse rate: -10 points each
  
  // Status levels:
  - healthy: score > 80
  - warning: score 60-80
  - critical: score < 60
  
  // Recommendations generated based on issues
}
```

---

## 8. CONFIGURATION SETTINGS

### 8.1 Memory Manager Configuration

**From**: `/src/core/config.ts` (default values):
```typescript
{
  directory: '.hive-mind',
  filename: 'memory.db',
  cacheSize: 1000,
  cacheMemoryMB: 50,
  compressionThreshold: 10240,  // 10KB
  gcInterval: 300000,           // 5 minutes
  enableWAL: true,
  enableVacuum: true,
  syncInterval: 30000           // 30 seconds
}
```

### 8.2 Hive Mind Configuration

**From**: `/benchmark/.hive-mind/config.json`:
```json
{
  "version": "2.0.0",
  "defaults": {
    "queenType": "strategic",
    "maxWorkers": 8,
    "consensusAlgorithm": "majority",
    "memorySize": 100,
    "autoScale": true,
    "encryption": false
  },
  "mcpTools": {
    "enabled": true,
    "parallel": true,
    "timeout": 60000
  }
}
```

---

## 9. KEY FILES & LOCATIONS

### Core Memory Files
| File | Purpose | Lines |
|------|---------|-------|
| `/src/hive-mind/core/Memory.ts` | Main memory engine with caching & optimization | 1437 |
| `/src/memory/manager.ts` | Memory manager with backend abstraction | 560 |
| `/src/memory/shared-memory.js` | Unified memory for .swarm/ and .hive-mind/ | 1000+ |
| `/src/memory/swarm-memory.ts` | Swarm-specific memory extensions | 100+ |
| `/src/swarm/memory.ts` | Distributed memory with partitioning | 100+ |

### Storage Backends
| File | Purpose |
|------|---------|
| `/src/memory/sqlite-store.js` | SQLite persistence layer |
| `/src/memory/in-memory-store.js` | RAM-based fallback |
| `/src/memory/fallback-store.js` | Auto-fallback logic |
| `/src/memory/backends/sqlite.ts` | SQLite backend interface |

### Optimization & Monitoring
| File | Purpose |
|------|---------|
| `/src/hive-mind/core/MemoryMonitor.ts` | Performance monitoring |
| `/src/memory/cache.ts` | LRU cache implementation |
| `/src/memory/indexer.ts` | Memory indexing |
| `/src/swarm/optimizations/ttl-map.ts` | TTL-based expiration |

### Serialization
| File | Purpose |
|------|---------|
| `/src/memory/advanced-serializer.js` | TypeScript-aware serialization |
| `/src/memory/enhanced-session-serializer.js` | Session serialization |

### Hooks & Integration
| File | Purpose |
|------|---------|
| `/src/services/agentic-flow-hooks/memory-hooks.ts` | Memory hooks integration |

---

## 10. SUMMARY TABLE

| Aspect | Details |
|--------|---------|
| **Storage Backends** | SQLite (primary), In-Memory, JSON (fallback) |
| **Cache Strategy** | 3-tier: LRU (L1), SQLite (L2), MCP (L3) |
| **Cache Size** | 10K entries, 100 MB RAM |
| **Memory Lifecycle** | Create → Update → Access → Expire → Delete |
| **Namespaces** | 6 default + custom |
| **Access Levels** | private, shared, public |
| **Scoping** | Agent, Swarm, Session, Global |
| **Serialization** | Advanced TypeScript-aware |
| **Compression** | Automatic (1 KB+), compression ratio tracked |
| **TTL Support** | Configurable per entry, auto-cleanup |
| **Persistence** | WAL mode SQLite, backups, snapshots |
| **Optimization** | LRU eviction, object pools, pattern learning |
| **Monitoring** | Performance tracking, health checks, alerts |
| **GC Intervals** | 30s (cache), 10s (metrics), 5m (cleanup), 2m (patterns) |
| **Indexes** | namespace, expires_at, accessed_at, key:namespace, tags |

---

**Analysis Date**: October 23, 2025
**Repository**: ai-claude-flow
**Version**: 2.7.1
