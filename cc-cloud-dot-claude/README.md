# Claude Configuration Backup

## Backup Information

- **Backup Date**: 2025-10-24T00:06:00.000Z
- **Backup File**: `etc-root-dot-claude.tar.gz`
- **Backup Size**: 1.2 MB (compressed)
- **Original Size**: ~5.3 MB (uncompressed)
- **Compression Ratio**: ~77% reduction

---

## Source Directories

### ✅ `/root/.claude` (Backed Up)
- **Location**: Root user's Claude Code configuration directory
- **Size**: 5.3 MB
- **Files**: 21 files
- **Directories**: 7 directories

**Contents**:
- `settings.json` - Claude Code settings
- `stop-hook-git-check.sh` - Git check stop hook script
- `projects/` - Project-specific session data
- `session-env/` - Session environment data
- `shell-snapshots/` - Shell state snapshots
- `statsig/` - Analytics and feature flags
- `todos/` - Todo state files

### ❌ `/etc/claude-code` (Not Found)
- **Status**: Directory does not exist on this system
- **Note**: Configuration may be stored in `/root/.claude` instead

---

## Archive Contents

**Total Entries**: 31 (files and directories)

### Directory Structure:
```
root/.claude/
├── settings.json
├── stop-hook-git-check.sh
├── statsig/
│   ├── statsig.session_id.2656274335
│   ├── statsig.cached.evaluations.09f291055f
│   ├── statsig.last_modified_time.evaluations
│   └── statsig.stable_id.2656274335
├── todos/
│   ├── 76aec28b-1d95-40c3-b6c0-d5c428c9cfd6-agent-*.json
│   ├── 2938f292-cfba-4522-93cf-5d051d0a4538-agent-*.json
│   ├── 9185c534-3928-43c6-917e-4368733c492a-agent-*.json
│   ├── d37edd2d-ecca-4a35-9140-944bf8b45cd4-agent-*.json
│   └── bea50ca1-6547-43da-bf7c-ebdb3adb2a82-agent-*.json
├── shell-snapshots/
│   ├── snapshot-bash-1761262544263-kntlzy.sh
│   ├── snapshot-bash-1761260794527-99at33.sh
│   ├── snapshot-bash-1761257462776-kuv0do.sh
│   ├── snapshot-bash-1761264308794-zdyecw.sh
│   └── snapshot-bash-1761262125782-sxnyuu.sh
├── session-env/
│   ├── 2803aae7-aae0-4b4f-b1a6-025efb213d96/
│   ├── 316cfd36-9fd8-48a0-bcff-44f9d1bc6fe6/
│   └── 82139739-03a1-4574-9d71-2135d6a3c763/
└── projects/
    └── -home-user-ai-claude-flow/
        ├── 2803aae7-aae0-4b4f-b1a6-025efb213d96.jsonl
        ├── 514fe40d-05cb-4133-a9a4-cd634a009e81.jsonl
        ├── 1126a293-c322-4805-8496-24c98118d6ec.jsonl
        └── 316cfd36-9fd8-48a0-bcff-44f9d1bc6fe6.jsonl
```

---

## What's Included

### 1. User Settings
- Claude Code configuration
- User preferences
- Custom settings

### 2. Session Data
- Active and historical sessions
- Session environment state
- Project-specific session files

### 3. Shell Snapshots
- Shell state backups
- Command history snapshots
- Environment snapshots

### 4. Todo States
- Agent-specific todo lists
- Task tracking data
- Progress states

### 5. Analytics & Telemetry
- Statsig session data
- Feature flag cache
- Analytics metadata

### 6. Hooks & Scripts
- Git check stop hook
- Custom hook scripts
- Automation scripts

---

## Archive Integrity

✅ **Verified**: Archive integrity check passed
✅ **Complete**: All 31 entries successfully archived
✅ **Compressed**: tar.gz format with gzip compression
✅ **Tested**: Archive can be extracted successfully

---

## Restore Instructions

### Full Restore
```bash
# Extract to original location (requires root)
sudo tar -xzf etc-root-dot-claude.tar.gz -C /

# Verify restore
ls -la /root/.claude
```

### Extract to Custom Location
```bash
# Extract to current directory
tar -xzf etc-root-dot-claude.tar.gz

# Extract to specific directory
tar -xzf etc-root-dot-claude.tar.gz -C /path/to/destination/
```

### View Contents Without Extracting
```bash
# List all files
tar -tzf etc-root-dot-claude.tar.gz

# List specific directory
tar -tzf etc-root-dot-claude.tar.gz | grep session-env

# Extract single file
tar -xzf etc-root-dot-claude.tar.gz root/.claude/settings.json
```

---

## Backup Verification

### Verify Archive Integrity
```bash
# Test archive
tar -tzf etc-root-dot-claude.tar.gz > /dev/null && echo "OK"

# Check file count
tar -tzf etc-root-dot-claude.tar.gz | wc -l

# Show archive info
tar -tvzf etc-root-dot-claude.tar.gz
```

### Compare with Original
```bash
# Compare file counts
diff <(find /root/.claude -type f | sort) \
     <(tar -tzf etc-root-dot-claude.tar.gz | grep -v '/$' | sed 's/^root\//\/root\//' | sort)
```

---

## Important Notes

⚠️ **Security Considerations**:
- This backup contains sensitive configuration data
- Includes session information and user preferences
- Store in a secure location
- Restrict access permissions

📋 **What's NOT Included**:
- `/etc/claude-code` (directory does not exist)
- Temporary files
- Cache data (not in `.claude`)
- System-wide configurations (if any)

🔄 **Recommended Backup Schedule**:
- Before major updates
- After significant configuration changes
- Weekly automated backups
- Before system migrations

---

## File Details

| Property | Value |
|----------|-------|
| **Filename** | etc-root-dot-claude.tar.gz |
| **Location** | cc-cloud-dot-claude/ |
| **Size (Compressed)** | 1.2 MB |
| **Size (Uncompressed)** | ~5.3 MB |
| **Format** | tar.gz (gzip compressed tarball) |
| **Entries** | 31 (files and directories) |
| **Compression** | gzip (-z flag) |
| **Archive Method** | GNU tar |

---

## Changelog

### 2025-10-24
- **Created**: Initial backup of `/root/.claude`
- **Note**: `/etc/claude-code` not found on system
- **Verified**: Archive integrity confirmed
- **Size**: 1.2 MB compressed
- **Entries**: 31 total entries

---

## Related Documentation

- [Session Archive 011CUQtutJdEPu3jTnB7QzRH](../cc-cloud-sessions/session-011CUQtutJdEPu3jTnB7QzRH/README.md)
- Claude Code Documentation: https://docs.claude.com/claude-code

---

**Created**: 2025-10-24T00:06:00.000Z
**Created By**: Claude Code Backup System
**Status**: ✅ Complete and Verified
