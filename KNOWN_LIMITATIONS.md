# Known Limitations (v1.3)

This document lists features that are planned but not yet implemented.

---

## v1.3 Limitations

### Model Tiers Require Dev Build
- **Limitation:** `model_tiers` configuration in `opencode.json` requires OpenCode dev build (PR #11377)
- **Stable releases:** Only support the default `model` field per agent (no tier routing)
- **Workaround:** Use default models or switch to dev build
- **See:** [docs/ADVANCED-SETUP.md](docs/ADVANCED-SETUP.md)

## Deferred to v1.2

### Observability Dashboard
- Event logging to `/tmp/pai-opencode-debug.log` works
- No visualization dashboard yet
- **Planned:** v1.2 - Web dashboard for session history

### Multi-Channel Notifications
- Voice notifications now working (v1.1)
- ntfy/Discord integration optional
- **Planned:** v1.2 - Full notification routing config

---

## Optional Features (Not Required)

### Skill Customizations
- System exists at `.opencode/skills/CORE/USER/SKILLCUSTOMIZATIONS/`
- Not populated by default
- Skills work without customizations
- **Setup:** Create customization files as needed

### Voice Server
- Voice notifications work with multiple backends
- ElevenLabs via Voice Server (if running on localhost:8888)
- Google Cloud TTS (if credentials configured)
- macOS `say` command (automatic fallback)
- **All fallbacks are graceful** - no errors if services unavailable

---

## Working in v1.1

### Core Systems
- [x] Core plugin system (auto-discovery, no config needed)
- [x] All 29 skills functional
- [x] TELOS/USER context loading
- [x] Security validation on tool execution
- [x] Memory structure (capture ready)
- [x] Skill routing and execution
- [x] 3 Provider Presets supported (Anthropic Max, ZEN PAID, ZEN FREE) + Custom via [ADVANCED-SETUP.md](.opencode/ADVANCED-SETUP.md)

### PAI 2.5 Algorithm (NEW in v1.1)
- [x] Full 7-phase Algorithm v0.2.25
- [x] ISC Validator with TaskCreate/TaskList
- [x] Capability Selection with Thinking Tools Assessment
- [x] Two-Pass capability selection
- [x] Parallel-by-default execution

### Handlers (13 total - NEW in v1.1)
- [x] `context-loader.ts` - CORE context injection
- [x] `security-validator.ts` - Dangerous command blocking
- [x] `rating-capture.ts` - User rating capture
- [x] `isc-validator.ts` - ISC criteria validation
- [x] `learning-capture.ts` - Learning to MEMORY
- [x] `work-tracker.ts` - Work session tracking
- [x] `skill-restore.ts` - Skill context restore
- [x] `agent-capture.ts` - Agent output capture
- [x] `voice-notification.ts` - TTS notifications (NEW)
- [x] `implicit-sentiment.ts` - Sentiment detection (NEW)
- [x] `tab-state.ts` - Kitty tab updates (NEW)
- [x] `update-counts.ts` - Skill/workflow counting (NEW)
- [x] `response-capture.ts` - ISC tracking (NEW)

---

## Troubleshooting

### Plugin not loading?
```bash
# Check plugin log
tail -f /tmp/pai-opencode-debug.log

# Verify plugin exists
ls -la .opencode/plugins/pai-unified.ts
```

### Context not injected?
```bash
# Check context files exist
ls -la .opencode/skills/CORE/USER/TELOS/
ls -la .opencode/skills/CORE/USER/DAIDENTITY.md
```

### Security validation blocking commands?
The security validator blocks dangerous commands by design. If a legitimate command is blocked, review the security rules in the plugin.

---

## Troubleshooting

### Plugin not loading?
```bash
# Check plugin log
tail -f /tmp/pai-opencode-debug.log

# Verify plugin exists
ls -la .opencode/plugins/pai-unified.ts
```

### Context not injected?
```bash
# Check context files exist
ls -la .opencode/skills/CORE/USER/TELOS/
ls -la .opencode/skills/CORE/USER/DAIDENTITY.md
```

### Voice notifications not working?
Voice notifications are optional. They fail gracefully if:
- Voice Server not running on localhost:8888
- Google Cloud TTS not configured
- Not on macOS (for `say` fallback)

Check the log for details:
```bash
grep -i voice /tmp/pai-opencode-debug.log
```

### Security validation blocking commands?
The security validator blocks dangerous commands by design. If a legitimate command is blocked, review the security rules in the plugin.

---

*Last updated: 2026-02-10*
*Version: 1.3.0*
