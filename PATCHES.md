# Community Patches

This branch (`my-fixes`) layers community fixes on top of the official AnythingLLM upstream.
It is intended for local/self-hosted Docker use and is kept in sync with upstream as releases progress.

## Version: 1.11.1-fixes.1

**Based on upstream:** `v1.11.1` (`Mintplex-Labs/anything-llm` @ `c927eda1`)

### Applied Patches

| Commit | Description |
|--------|-------------|
| `ae0ac9a3` | fix: increase Anthropic agent provider max_tokens to 8192 |
| `340bed7c` | fix: use per-model output token limits for Anthropic agent provider |
| `f1e5582d` | fix: replace UA-based mobile detection with viewport size check for workspace modal |
| `8a0d7fc9` | fix: detect actual image mime type from magic bytes for Anthropic provider |

### Excluded Patches (needs more testing)

| Commit | Description |
|--------|-------------|
| `642007a5` | feat: add raw input mode to agent flows |

### Already Merged Upstream (removed from patches)

| PR | Description |
|----|-------------|
| [#5131](https://github.com/Mintplex-Labs/anything-llm/pull/5131) | fix: show actionable error when LMStudio model listing fails or returns empty |

---

## Updating This Branch

When upstream releases a new version, update this branch by:

```bash
git fetch upstream
git checkout my-fixes
git rebase v<new-version>   # rebase onto new stable release tag
# Update version in package.json and docker/Dockerfile
# Update this file's "Based on upstream" line and version
git tag v<new-version>-fixes.<n>
git push origin my-fixes --force --tags
```
