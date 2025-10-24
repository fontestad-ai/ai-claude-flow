# Session Persistence - Quick Reference Guide

## Essential Files

| Component | Location | Purpose |
|-----------|----------|---------|
| Session Manager | `/src/mcp/session-manager.ts` | Core session lifecycle |
| Session CLI | `/src/cli/commands/session.ts` | Session commands |
| Checkpoint Manager | `/src/sdk/checkpoint-manager.ts` | Version control |
| Memory Manager | `/src/memory/advanced-memory-manager.ts` | Memory storage |
| Memory Hooks | `/src/services/agentic-flow-hooks/memory-hooks.ts` | Automatic sync |
| Session Serializer | `/src/memory/enhanced-session-serializer.js` | Serialization |
| SQLite Persistence | `/src/core/persistence.ts` | Database backend |
| JSON Persistence | `/src/core/json-persistence.ts` | File backend |

## Session Commands

```bash
# Create session
npx claude-flow session save --name "my-session" --auto

# List sessions
npx claude-flow session list --format table

# Restore session
npx claude-flow session restore <session-id> --force

# Export session
npx claude-flow session export <session-id> output.json --include-memory

# Import session
npx claude-flow session import input.json --name "restored-session"

# Show session info
npx claude-flow session info <session-id>

# Clean old sessions
npx claude-flow session clean --older-than 30 --dry-run
```

## Hook Commands

```bash
# Before starting work
npx claude-flow hooks session-restore \
  --session-id "prev-session" \
  --load-memory true \
  --restore-files true

# During work (after edits)
npx claude-flow hooks post-edit \
  --file "src/file.ts" \
  --memory-key "swarm/agent/step" \
  --telemetry true

# After work (cleanup)
npx claude-flow hooks session-end \
  --session-id "current-session" \
  --save-state true \
  --export-metrics true \
  --generate-summary true
```

## Checkpoint Commands

```bash
# Create checkpoint
npx claude-flow checkpoint create <session-id> "Checkpoint description"

# List checkpoints
npx claude-flow checkpoint list <session-id> --format json

# Get checkpoint info
npx claude-flow checkpoint info <checkpoint-id>

# Rollback to checkpoint
npx claude-flow checkpoint rollback <checkpoint-id> --force

# Delete checkpoint
npx claude-flow checkpoint delete <checkpoint-id> --force
```

## Storage Locations

```
~/.claude-flow/
├── sessions/              # Session files (JSON)
│   └── session-*.json     # Individual sessions
├── checkpoints/           # Checkpoint files
│   └── {uuid}.json        # Message UUID-based
├── memory/                # Memory storage
│   └── sqlite-store.db    # In-memory SQLite
├── claude-flow.db         # Main database
└── claude-flow-data.json  # JSON fallback
```

## Session Structure

```json
{
  "id": "session-1729608000000-abc123",
  "name": "dev-session-2024",
  "createdAt": "2025-10-23T10:00:00Z",
  "updatedAt": "2025-10-23T11:30:00Z",
  
  "state": {
    "agents": [{"id", "type", "status"}],
    "tasks": [{"id", "type", "status", "progress"}],
    "memory": [{"id", "type", "data"}],
    "configuration": {}
  },
  
  "metadata": {
    "version": "2.0.0",
    "platform": "linux",
    "checksum": "sha256hash"
  }
}
```

## Memory Operations

```typescript
// Store value
await memory.store("namespace", "key", value, {
  tags: ["tag1", "tag2"],
  accessLevel: "shared",
  ttl: 604800  // 7 days
});

// Retrieve value
const value = await memory.retrieve("namespace", "key");

// Query with filters
const results = await memory.query({
  namespace: "tasks",
  type: "pending",
  tags: ["high-priority"],
  limit: 10,
  sortBy: "createdAt",
  sortOrder: "desc"
});

// Export memory
const exported = await memory.export("json", {
  namespace: "tasks",
  compression: true
});

// Cleanup old entries
const stats = await memory.cleanup({
  removeExpired: true,
  removeOlderThan: 30,
  dryRun: false
});
```

## Key Limits

| Limit | Value |
|-------|-------|
| Max concurrent sessions | 100 |
| Session timeout | 3600s (1 hour) |
| Max session size | ~100MB |
| Max memory entry | 10MB |
| Max entries per session | 10,000 |
| Max checkpoints per session | 50 |
| Memory entry TTL | 604800s (7 days) |
| Cleanup interval | 60s |

## Performance Benchmarks

| Operation | Latency |
|-----------|---------|
| Session creation | 5-10ms |
| Session lookup | <1ms |
| Memory store | 10-100ms |
| Memory retrieve | 10-50ms |
| Checkpoint create | 50-200ms |
| Checkpoint rollback | 100-500ms |
| Session serialization | 50-1000ms |

## Data Persistence

### Persists Across Sessions
- Session metadata (ID, timestamps, checksum)
- Agent state (IDs, types, status)
- Task state (descriptions, progress)
- Memory entries (all data + metadata)
- File context (read, modified, created)
- Background process state
- Checkpoint history
- Performance metrics

### Does NOT Persist
- Live process outputs
- Authentication tokens
- Temporary variables
- Permissions (needs re-approval)
- SSH sessions
- Environment variables

## Troubleshooting

### Session won't restore
```bash
# Check session integrity
npx claude-flow session info <session-id>

# Try with merge mode
npx claude-flow session restore <session-id> --merge

# Check for corruption
# File will show warnings if corrupted
```

### Memory operations slow
```bash
# Cleanup old entries
npx claude-flow memory cleanup --older-than 7

# Check memory stats
npx claude-flow memory stats

# Optimize indices
npx claude-flow memory optimize
```

### Checkpoint restore failing
```bash
# List available checkpoints
npx claude-flow checkpoint list <session-id>

# Check checkpoint integrity
npx claude-flow checkpoint info <checkpoint-id>

# Verify checksum
# Output will show if valid
```

## Version Compatibility

- **v1.0.0**: Basic session storage
- **v2.0.0**: Enhanced serialization, compression, migration
  - Auto-migrates v1 → v2
  - Adds compression support
  - Improved metadata tracking
  - Neural pattern detection

## Security Notes

- Session IDs include timestamps + random suffix
- Authentication: token, basic, oauth support
- Passwords hashed with SHA-256
- Timing-safe comparison prevents timing attacks
- Memory access levels: private/shared/public
- Optional encryption for exports
- Checksum validation on restore

## API Examples

### Using Session Manager
```typescript
import { SessionManager } from './mcp/session-manager';

const manager = new SessionManager(config, logger);

// Create session
const session = manager.createSession('stdio');

// Initialize
manager.initializeSession(session.id, params);

// Authenticate
manager.authenticateSession(session.id, credentials);

// Get metrics
const metrics = manager.getSessionMetrics();
```

### Using Checkpoint Manager
```typescript
import { checkpointManager } from './sdk/checkpoint-manager';

// Create checkpoint
const cpId = await checkpointManager.createCheckpoint(
  sessionId,
  "Work milestone reached"
);

// Rollback
const query = await checkpointManager.rollbackToCheckpoint(cpId);

// List checkpoints
const cps = checkpointManager.listCheckpoints(sessionId);
```

### Using Memory Manager
```typescript
import { AdvancedMemoryManager } from './memory/advanced-memory-manager';

const memory = new AdvancedMemoryManager();
await memory.initialize();

// Store
await memory.store("tasks", "task-1", { status: "done" });

// Query
const results = await memory.query({
  namespace: "tasks",
  valueSearch: "done",
  limit: 10
});

// Stats
const stats = await memory.getStatistics();
```

## Related Documentation

- Full Architecture: `CROSS_SESSION_PERSISTENCE_ARCHITECTURE.md`
- Session Persistence Wiki: `/docs/wiki/session-persistence.md`
- Hook Documentation: `/bin/init/templates/commands/hooks/`
- MCP Integration: `/src/mcp/`

---
**Last Updated**: October 23, 2025
**Claude-Flow Version**: v2.5-alpha.130+
