# Claude Code Configuration Backup

**Created**: 2025-10-24
**Archive**: etc-root-dot-claude.tar.gz
**Session**: 011CUQebft8eKCkMSAgUbR5F

---

## Backup Contents

This archive contains Claude Code configuration and session data from:

### Directories Backed Up

- `/root/.claude/` (57MB uncompressed)

**Note**: `/etc/claude-code/` directory was not found on this system.

### Directory Structure

```
.claude/
├── projects/ (55MB) - Claude Code project workspaces
├── shell-snapshots/ (1.8MB) - Shell session snapshots
├── session-env/ (20KB) - Session environment data
├── statsig/ (29KB) - Analytics and feature flags
├── todos/ (8KB) - Todo list data
├── settings.json (512B) - Claude Code settings
└── stop-hook-git-check.sh (2KB) - Git check hook
```

### Archive Details

- **Compression**: gzip
- **Format**: tar.gz
- **Total Files**: All files and subdirectories from /root/.claude
- **Integrity**: Verified with gzip -t

---

## Restoration

To restore this backup:

```bash
# Extract to /root/.claude/
cd /root
tar -xzf /path/to/etc-root-dot-claude.tar.gz

# Or extract to a different location
tar -xzf etc-root-dot-claude.tar.gz -C /target/directory/
```

---

## Contents Description

### settings.json
Claude Code configuration including:
- MCP server configurations (claude-flow, ruv-swarm, etc.)
- Permission settings
- Editor preferences

### projects/
User project workspaces with:
- Project-specific settings
- Session histories
- Context data

### shell-snapshots/
Terminal session snapshots for:
- Session restoration
- Command history
- Shell state preservation

### todos/
Task management data:
- Active todo lists
- Completed tasks
- Task history

### session-env/
Session environment variables and state

### statsig/
Feature flag and analytics data

---

## Backup Schedule

This backup was created as part of session archival process.

For automated backups, consider:
```bash
# Daily backup script
tar -czf "backup-$(date +%Y%m%d).tar.gz" -C /root .claude
```

---

## Security Notes

**WARNING**: This archive contains:
- API keys and tokens (in settings.json)
- Session data and project files
- Potentially sensitive configuration

**Storage Recommendations**:
- Keep in private repository only
- Do not share publicly
- Encrypt for long-term storage
- Rotate backups regularly

---

**Backup Created By**: Claude Code Session 011CUQebft8eKCkMSAgUbR5F
**Date**: 2025-10-24 00:05 UTC
