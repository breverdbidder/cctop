# cctop — Security & Value Assessment

**Date:** March 17, 2026
**Assessed By:** Claude AI (AI Architect, BidDeed.AI)
**Decision:** ✅ ADOPT (Score: 84/100)

## Tool Info

| Field | Value |
|---|---|
| Tool | cctop v0.1.0 |
| Source | github.com/DeanLa/cctop |
| Fork | github.com/breverdbidder/cctop |
| Author | DeanLa (SentinelOne engineer) |
| License | MIT |
| Cost | $0 (local-only, no API) |

## Scorecard

| Category | Score | Notes |
|---|---|---|
| Code Quality | 8.5/10 | Clean 3-component architecture. Atomic writes. Incremental JSONL parsing with byte offsets + inode tracking. |
| Security | 8.0/10 | Local-only, no network, no API keys. One eval-from-jq concern (mitigated by @sh quoting). |
| Documentation | 9.0/10 | Layered docs (README/CLAUDE.md/CONTRIBUTING). Reference docs with when-to-read guide. |
| Test Coverage | 8.0/10 | ~700 LOC tests. Unit + headless TUI integration. Mocked subprocess. |
| **Overall** | **8.4/10** | Solid v0.1.0. |

## Security Analysis

### Attack Surface
- Data flow: strictly local JSON files in ~/.cctop/
- No network calls, no API keys, no auth
- Dependencies: textual, jq, uv (all well-known)

### Identified Risks
| Risk | Severity | Mitigation |
|---|---|---|
| eval in bash hook | LOW | jq @sh quoting + trusted input from Claude Code |
| ~/.cctop/ permissions | LOW | Default umask on single-user dev machines |
| shutil.rmtree --reset | MINIMAL | Scoped to ~/.cctop/ only |

## Value to BidDeed.AI
- **Ctx% column** directly supports 50% context kill rule (SESSION HYGIENE)
- Monitors auto-mode sessions across 5+ repos without terminal switching
- Error column flags stuck sessions faster
- Status column shows idle sessions waiting for input (ADHD context-switch reduction)
- Complements cc-status-line as outer-loop monitor
- Zero cost

## Installation
```bash
# Prerequisites: jq, uv, claude CLI
curl -fsSL https://raw.githubusercontent.com/DeanLa/cctop/main/install.sh | bash
cctop  # launch in separate terminal
```

**Note:** Only sessions started AFTER install are tracked.
