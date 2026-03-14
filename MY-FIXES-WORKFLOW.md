# my-fixes Branch Workflow

## What This Is

`origin/my-fixes` is a personal build branch on top of the official AnythingLLM stable releases.
It lives at `https://github.com/elevatingcreativity/anything-llm` and is intended for local/self-hosted Docker use.

Current version: `v1.11.1-fixes.3`
Based on: upstream stable tag `v1.11.1` (`c927eda1`)

## Applied Patches (as of fixes.3)

| Commit | Description |
|--------|-------------|
| `340bed7c` | fix: use per-model output token limits for Anthropic agent provider |
| `f1e5582d` | fix: replace UA-based mobile detection with viewport size check for workspace modal |
| `8a0d7fc9` | fix: detect actual image mime type from magic bytes for Anthropic provider |

## Branch Structure

- `origin/master` — tracks `upstream/master` (Mintplex-Labs). Do all bug fix work here in separate branches. Can be PR'd upstream.
- `origin/my-fixes` — personal build branch. Cherry-pick tested fixes from master branches into here.

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

3. **Test it. When happy, cherry-pick it into my-fixes:**
   ```bash
   git checkout my-fixes
   git cherry-pick <commit-hash>
   # resolve any conflicts
   ```

4. **Bump the fixes version** (e.g. fixes.3 → fixes.4):
   - `docker/Dockerfile` — update `ENV DEPLOYMENT_VERSION=1.11.1-fixes.4`
   - `package.json` — update `"version": "1.11.1-fixes.4"`
   - `PATCHES.md` — add the new patch to the table, update version line

5. **Commit, tag, push:**
   ```bash
   git add package.json docker/Dockerfile PATCHES.md
   git commit -m "chore: version 1.11.1-fixes.4 — add <description>"
   git tag -a v1.11.1-fixes.4 -m "AnythingLLM v1.11.1 + morgan's fixes (fixes.4)"
   git push origin my-fixes --force --tags
   ```

6. **Rebuild Docker image:**
   ```bash
   cd docker
   docker-compose build
   docker tag anythingllm-anything-llm:latest anythingllm-my-fixes:1.11.1-fixes.4
   ```

7. **Save tar for remote Mac:**
   ```bash
   docker save anythingllm-my-fixes:1.11.1-fixes.4 | gzip > ~/Desktop/anythingllm-my-fixes-1.11.1-fixes.4.tar.gz
   ```

---

## Workflow: Upgrading to a New Upstream Stable Release

When Mintplex-Labs releases e.g. `v1.12.0`:

1. **Fetch upstream:**
   ```bash
   git fetch upstream
   ```

2. **Check what the new tag is and what changed:**
   ```bash
   git log --oneline v1.11.1..v1.12.0
   ```

3. **Check if any of your patches are already merged upstream:**
   ```bash
   git log v1.12.0 --oneline --grep="per-model output token"   # etc for each patch
   ```
   Remove any already-merged patches from the cherry-pick list.

4. **Recreate my-fixes from the new tag:**
   ```bash
   git checkout my-fixes
   git reset --hard v1.12.0
   ```

5. **Cherry-pick remaining patches** (check PATCHES.md for the list):
   ```bash
   git cherry-pick <hash1> <hash2> ...
   # resolve any conflicts
   ```

6. **Update versions and PATCHES.md**, bump to `v1.12.0-fixes.1`, commit, tag, push, rebuild, save tar — same as steps 4–7 above.

---

## Deploying to Remote Mac

1. Transfer tar to remote Mac (AirDrop, shared drive, etc.)
2. On remote Mac:
   ```bash
   docker load -i ~/Desktop/anythingllm-my-fixes-<version>.tar.gz
   docker tag anythingllm-anything-llm:latest anythingllm-my-fixes:<version>
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
git checkout my-fixes
```

Then add upstream remote:
```bash
git remote add upstream https://github.com/Mintplex-Labs/anything-llm.git
```
