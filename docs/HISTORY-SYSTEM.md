# OpenCode History System Documentation

**Version:** 1.1.0
**Created:** 2026-01-01
**Last Updated:** 2026-01-02
**PAI-OpenCode Version:** v0.6.0 (implementation deferred from v0.5.0)

**SCOPE CHANGE NOTE (2026-01-02):** This document was originally created for v0.5.0, but the History System implementation has been moved to v0.6.0. The v0.5.0 milestone now focuses on Plugin Infrastructure, which is a prerequisite for the complete History System implementation.

---

## Overview

OpenCode provides a native session storage system that captures all conversations, messages, and file changes in a hierarchical JSON structure. This document describes how OpenCode stores and retrieves session data.

**Key Features:**
- Persistent session storage across restarts
- Project-level and global session organization
- Hierarchical message and part tracking
- Built-in export/import functionality
- Public sharing capability

---

## Session Storage Location

OpenCode uses a centralized storage directory with project-level organization:

**Storage Root:**
```
~/.local/share/opencode/storage/
```

### Directory Structure

```
~/.local/share/opencode/storage/
├── session/
│   ├── global/                    # Non-git project sessions
│   │   └── ses_*.json
│   └── {project-hash}/            # Git repository sessions
│       └── ses_*.json
├── message/
│   └── ses_*/                     # Messages grouped by session
│       └── msg_*.json
├── part/
│   └── msg_*/                     # Parts grouped by message
│       └── prt_*.json
├── session_diff/
│   └── ses_*.json                 # Session diffs
├── project/                       # Project metadata
├── snapshot/                      # File snapshots
└── todo/                          # Todo items
```

**Storage is BOTH project-level and global:**
- **Project-level:** Git repositories get a unique hash directory under `session/{project-hash}/`
- **Global:** Non-git projects use `session/global/`

This dual-level organization ensures:
- Clean separation between different projects
- Session persistence across directory moves (for git repos)
- Easy bulk operations on project sessions

---

## Session Data Format

OpenCode uses JSON files with a hierarchical structure:

### Session File (`ses_*.json`)

**Location:** `~/.local/share/opencode/storage/session/{project-hash}/ses_*.json`

**Example:**
```json
{
  "id": "ses_4f797e66cffeR7hasQZd0QPy57",
  "version": "1.0.141",
  "projectID": "global",
  "directory": "/Users/steffen/workspace/tmp/warriorcon5",
  "title": "Anwenden-Analyse WarriorCon5 Vorbereitungsanweisungen",
  "time": {
    "created": 1765372598675,
    "updated": 1765372970195
  },
  "summary": {
    "additions": 0,
    "deletions": 0,
    "files": 0
  }
}
```

**Fields:**
- `id` - Session identifier with `ses_` prefix
- `version` - OpenCode version that created the session
- `projectID` - Project hash or "global"
- `directory` - Absolute path to working directory
- `title` - Session title (auto-generated or user-provided)
- `time.created` - Unix timestamp (milliseconds)
- `time.updated` - Last modification timestamp
- `summary` - File change statistics

### Message File (`msg_*.json`)

**Location:** `~/.local/share/opencode/storage/message/ses_*/msg_*.json`

**Example:**
```json
{
  "id": "msg_b7b44abf9001N9eK1J9Jpp9O9U",
  "sessionID": "ses_484bb5409ffeTcNE2DneLJuP6R",
  "role": "user",
  "time": {
    "created": 1767299656697
  },
  "summary": {
    "title": "PAI-OpenCode Overview",
    "diffs": []
  },
  "agent": "writer",
  "model": {
    "providerID": "anthropic",
    "modelID": "claude-sonnet-4-5"
  },
  "tools": {
    "todowrite": false,
    "todoread": false,
    "task": false
  }
}
```

**Fields:**
- `id` - Message identifier with `msg_` prefix
- `sessionID` - Parent session ID
- `role` - "user" or "assistant"
- `time.created` - Message timestamp
- `summary` - Message summary and file diffs
- `agent` - Agent name if delegated (e.g., "writer")
- `model` - AI provider and model used
- `tools` - Tool usage flags

### Part File (`prt_*.json`)

**Location:** `~/.local/share/opencode/storage/part/msg_*/prt_*.json`

**Example:**
```json
{
  "id": "prt_b7b44b638001sUF0GuLtP2DKnz",
  "sessionID": "ses_484bb5409ffeTcNE2DneLJuP6R",
  "messageID": "msg_b7b44abfc001L5pxfuAVX2WVYS",
  "type": "step-start",
  "snapshot": "0c87253449d8c5585c1396be962532529e7ca28b"
}
```

**Fields:**
- `id` - Part identifier with `prt_` prefix
- `sessionID` - Parent session ID
- `messageID` - Parent message ID
- `type` - Part type (e.g., "step-start", "step-end", "text", "tool-call")
- `snapshot` - File snapshot hash (if applicable)

---

## Session ID Format

OpenCode uses custom-encoded IDs with type prefixes:

| Type | Prefix | Example |
|------|--------|---------|
| Session | `ses_` | `ses_4f797e66cffeR7hasQZd0QPy57` |
| Message | `msg_` | `msg_b7b44abf9001N9eK1J9Jpp9O9U` |
| Part | `prt_` | `prt_b7b44b638001sUF0GuLtP2DKnz` |

**Encoding Characteristics:**
- NOT UUID or ULID format
- Custom base encoding with mixed case
- Type-safe via prefix
- Globally unique

**Timestamp Format:**
- Unix milliseconds in `time.created` and `time.updated` fields
- Example: `1765372598675` = 2025-12-31 01:43:18 UTC

---

## Persistence Behavior

### Automatic Persistence

OpenCode automatically saves sessions at these events:
1. **User message submission** - Creates new message file
2. **Assistant response** - Creates response message and parts
3. **Tool execution** - Creates tool-call and tool-result parts
4. **Session end** - Updates session summary and timestamp
5. **File changes** - Creates snapshots and updates session summary

### Cross-Restart Persistence

All session data persists across:
- OpenCode restarts
- System reboots
- Directory changes (for git projects)

**No data loss occurs** during normal operation. OpenCode writes to disk continuously throughout the session.

### Session State

Sessions track state via the `summary` field:
- `additions` - Lines added to files
- `deletions` - Lines deleted from files
- `files` - Number of files modified

This enables quick session filtering without loading full message history.

---

## Session Retrieval Methods

### CLI Commands

**Continue last session:**
```bash
opencode -c
opencode --continue
```

**Continue specific session:**
```bash
opencode -s ses_4f797e66cffeR7hasQZd0QPy57
opencode --session ses_4f797e66cffeR7hasQZd0QPy57
```

**Export session:**
```bash
opencode export ses_4f797e66cffeR7hasQZd0QPy57
```

Output: JSON file with complete session data (messages, parts, snapshots)

**Import session:**
```bash
opencode import session-export.json
opencode import https://opencode.ai/s/abc123
```

### TUI Commands

**List sessions:**
```
/sessions
```

Opens interactive session picker with:
- Session titles
- Creation dates
- Project context
- Message counts

**Share session:**
```
/share
```

Uploads session to `opencode.ai/s/<id>` for public sharing.

---

## Comparison to Claude Code

| Feature | OpenCode | Claude Code |
|---------|----------|-------------|
| **Storage Format** | JSON files (hierarchical) | JSONL files (flat) |
| **Location** | `~/.local/share/opencode/storage/` | `~/.claude/projects/{cwd}/` |
| **Organization** | Project hash directories | Working directory paths |
| **Session ID** | `ses_` + custom encoding | UUID-based |
| **Sharing** | Built-in `/share` command | None |
| **Real-time Sync** | SSE to all clients | Single terminal |
| **Session Hierarchy** | Parent/child supported | None |
| **Message Structure** | Hierarchical (session → message → part) | Flat event stream |

**Key Differences:**

1. **Storage Model:**
   - OpenCode: Structured database-like with relationships
   - Claude Code: Event log (append-only JSONL)

2. **Organization:**
   - OpenCode: Project-level with hash-based deduplication
   - Claude Code: Directory-level with full paths

3. **Retrieval:**
   - OpenCode: Structured queries via CLI/TUI
   - Claude Code: Manual file reading

4. **Collaboration:**
   - OpenCode: Built-in sharing and real-time sync
   - Claude Code: No native sharing mechanism

---

## Out of Scope for v1.0

The following PAI features are **NOT part of the base OpenCode history system** and are deferred to Phase 2 (Jeremy 2.0):

### PAI Knowledge Layer

- `history/learnings/` - Problem-solving narratives
- `history/research/` - Investigation results
- `history/decisions/` - Architecture Decision Records
- `history/ideas/` - Quick thought captures
- `history/projects/` - Multi-session project tracking

**Rationale:** These are PAI-specific organizational tools built ON TOP of session storage, not part of the OpenCode platform itself.

### PAI History Hooks

- `PostToolUse` - Capturing tool outputs
- `Stop` - Voice feedback integration
- `SessionEnd` - Summary generation

**Rationale:** These hooks provide PAI-specific behavior and will be implemented as OpenCode plugins in v0.7.

### Session Search

PAI's `session-search` CLI tool uses:
- Indexed keyword search
- Session summaries in `history/sessions/INDEX.json`
- Advanced filtering by date, keyword, agent

**Rationale:** This is PAI-specific tooling. OpenCode provides `/sessions` TUI and export commands natively.

---

## Implementation Notes

**IMPORTANT:** Full History System implementation is now targeted for v0.6 (moved from v0.5 due to plugin infrastructure dependency).

For PAI-OpenCode v0.6, session capture will be **fully functional** using OpenCode's native system plus PAI knowledge layer:

✅ **Working:**
- Session storage at `~/.local/share/opencode/storage/`
- Session persistence across restarts
- Session retrieval via `-c` and `-s` flags
- Session export/import
- Session sharing

❌ **Not Yet Implemented (Phase 2):**
- PAI knowledge layer (learnings, research, decisions)
- Session summaries with keyword extraction
- Voice feedback on session end
- Automated session categorization

**For v1.0:** Both layers (OpenCode sessions + PAI knowledge layer) will be fully functional. This provides complete parity with Claude Code PAI.

**Dependency on v0.5 Plugin Infrastructure:**
- v0.5 must complete plugin translation before v0.6 History System
- Plugins provide event capture (PostToolUse, SessionEnd, Stop)
- History System consumes these events to populate knowledge layer
- Original ROADMAP had this dependency inverted - corrected 2026-01-02

---

## Acceptance Criteria

### Research Phase (Completed 2026-01-01)

| ID | Criteria | Status |
|----|----------|--------|
| AC-1 | Session storage location documented | ✅ PASS |
| AC-2 | Session transcripts captured correctly | ✅ PASS |
| AC-3 | Sessions persist across restarts | ✅ PASS |
| AC-4 | Session content retrievable | ✅ PASS |
| AC-5 | No data loss during sessions | ✅ PASS |
| AC-6 | Session capture meets functional requirements | ✅ PASS |

### Implementation Phase (v0.6 - Not Yet Started)

| ID | Criteria | Status |
|----|----------|--------|
| AC-7 | Plugin infrastructure complete (v0.5 dependency) | ⏳ PENDING |
| AC-8 | PAI knowledge directories created | ⏳ PENDING |
| AC-9 | Session summarization plugin working | ⏳ PENDING |
| AC-10 | Learning extraction plugin working | ⏳ PENDING |
| AC-11 | session-search CLI tool ported | ⏳ PENDING |
| AC-12 | Complete workflow tested (capture → store → retrieve) | ⏳ PENDING |

---

## References

- OpenCode Repository: https://github.com/opencodelabs/opencode
- OpenCode Documentation: https://github.com/opencodelabs/opencode/wiki
- Session Storage Research: `research/opencode-session-storage-research.md`

---

## Document Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-01 | Initial documentation for v0.5 research phase |
| 1.1.0 | 2026-01-02 | Updated for v0.6 implementation (scope swap), added dependency notes, updated acceptance criteria with implementation phase |

---

**Document Version:** 1.1.0
**Last Updated:** 2026-01-02
**Author:** PAI-OpenCode Team
