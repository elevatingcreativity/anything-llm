# my-fixes Branch Workflow

## What This Is

`origin/my-fixes-v1.11.2` is a personal build branch on top of the official AnythingLLM stable releases.
It lives at `https://github.com/elevatingcreativity/anything-llm` and is intended for local/self-hosted Docker use.

Current version: `v1.11.2-fixes.1`
Based on: upstream stable tag `v1.11.2`

## Applied Patches (as of fixes.1)

| Commit | Description |
|--------|-------------|
| `6c397174` | fix: detect actual image mime type from magic bytes for Anthropic provider |
| `1b9f26ef` | fix: replace UA-based mobile detection with viewport size check for workspace modal |
| `1b60f524` | fix: use per-model output token limits for Anthropic agent provider |

See `PATCHES.md` for full details on each patch.

## Branch Structure

- `origin/master` — tracks `upstream/master` (Mintplex-Labs). Do all bug fix work here in separate branches. Can be PR'd upstream.
- `origin/my-fixes-v1.11.2` — personal build branch for v1.11.2. Cherry-pick tested fixes from master branches into here.

---

## Workflow: Adding a New Bug Fix

1. **Make sure you're on master and it's current:**
   ```bash
   git checkout master
   git fetch upstream
   git merge upstream/master  # or rebase, your preference
   git push origin master
   ```

2. **Create a fix branch off master:**
   ```bash
   git checkout -b fix/your-fix-name
   # ... do work, commit ...
   git push origin fix/your-fix-name
   ```

3. **Test it. When happy, cherry-pick it into the fixes branch:**
   ```bash
   git checkout my-fixes-v1.11.2
   git cherry-pick <commit-hash>
   # resolve any conflicts
   ```

4. **Bump the fixes version** (e.g. fixes.1 → fixes.2):
   - `docker/Dockerfile` — update `ENV DEPLOYMENT_VERSION=1.11.2-fixes.2`
   - `package.json` — update `"version": "1.11.2-fixes.2"`
   - `PATCHES.md` — add the new patch, update version line

5. **Commit, tag, push:**
   ```bash
   git add package.json docker/Dockerfile PATCHES.md MY-FIXES-WORKFLOW.md
   git commit -m "chore: version 1.11.2-fixes.2 — add <description>"
   git tag -a v1.11.2-fixes.2 -m "AnythingLLM v1.11.2 + morgan's fixes (fixes.2)"
   git push origin my-fixes-v1.11.2 --tags
   ```

6. **Build Docker image:**
   ```bash
   docker build -f docker/Dockerfile -t anythingllm-my-fixes:1.11.2-fixes.2 .
   docker save anythingllm-my-fixes:1.11.2-fixes.2 | gzip > ~/Desktop/anythingllm-my-fixes-1.11.2-fixes.2.tar.gz
   ```

---

## Workflow: Rebasing onto a New Upstream Release

1. **Fetch upstream:**
   ```bash
   git fetch upstream --tags
   ```

2. **Check what the new tag is and what changed:**
   ```bash
   git log --oneline v1.11.2..v1.12.0
   ```

3. **Check if any of your patches are already merged upstream:**
   ```bash
   git diff v1.12.0 HEAD -- server/utils/agents/aibitat/providers/anthropic.js
   # etc for each patched file — look for your changes already present
   ```
   Check PATCHES.md for the file list. Remove any already-merged patches from the plan.

4. **Create a new fixes branch from the new tag:**
   ```bash
   git checkout -b my-fixes-v1.12.0 v1.12.0
   ```

5. **Cherry-pick remaining patches** (check PATCHES.md for current commit hashes):
   ```bash
   git cherry-pick <hash1> <hash2> <hash3>
   # resolve any conflicts
   ```

6. **Update versions and PATCHES.md**, bump to `v1.12.0-fixes.1`, commit, tag, push, rebuild.

---

## Deploying to Remote Mac

1. Transfer tar to remote Mac (AirDrop, shared drive, etc.)
2. On remote Mac:
   ```bash
   docker load -i ~/Desktop/anythingllm-my-fixes-<version>.tar.gz
   docker tag anythingllm-my-fixes:<version> anythingllm-my-fixes:latest
   docker stop anythingllm-my-fixes   # or whatever the running container is named
   ```
3. Run:
   ```
   docker run --name=anythingllm-my-fixes --cap-add=SYS_ADMIN --restart=unless-stopped -p 3001:3001 --env=STORAGE_DIR=/app/server/storage --volume="/Users/morgan/Library/Application Support/AnythingLLM-Docker/storage:/app/server/storage" --volume="/Users/morgan/Library/Application Support/AnythingLLM-Docker/.env:/app/server/.env" -d anythingllm-my-fixes:<version>
   ```

---

## Cloning on a New Machine

```bash
git clone https://github.com/elevatingcreativity/anything-llm.git
cd anything-llm
git checkout my-fixes-v1.11.2
```

Then add upstream remote:
```bash
git remote add upstream https://github.com/Mintplex-Labs/anything-llm.git
```
