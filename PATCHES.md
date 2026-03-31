# PATCHES.md — custom fixes on top of upstream AnythingLLM

Base: upstream v1.11.2 (tag `v1.11.2`)
Our version: `1.11.2-fixes.1` (branch `my-fixes-v1.11.2`)

## Applied patches

### 1. fix: detect actual image mime type from magic bytes for Anthropic provider
**Commit:** `6c397174`
**File:** `server/utils/AiProviders/anthropic/index.js`
**Why:** When the desktop overlay sends a screenshot with an incorrect declared MIME type,
Anthropic rejects it with `invalid_request_error`. We inspect magic bytes of the
base64-decoded image to detect the actual format (PNG/JPEG/GIF/WebP) before sending
to the Anthropic API, falling back to the declared type if detection fails.

### 2. fix: replace UA-based mobile detection with viewport size check for workspace modal
**Commit:** `1b9f26ef`
**Files:** `frontend/src/components/Modals/ManageWorkspace/index.jsx`,
`frontend/src/components/Sidebar/ActiveWorkspaces/index.jsx`,
`frontend/src/locales/en/common.js`
**Why:** `isMobile` from `react-device-detect` checks the User-Agent, so "Request Desktop
Site" on iPad was ignored and the modal was blocked. Replaced with `window.innerWidth < 560
|| window.innerHeight < 600` so viewport size is the authority. Also fixes iOS double-tap
issue on the upload button in the sidebar by adding an `onTouchEnd` handler. Removes
`md:` prefix on `overflow-y-auto` so the modal scrolls correctly on iPad mini in portrait.

### 3. fix: use per-model output token limits for Anthropic agent provider
**Commit:** `1b60f524`
**Files:** `server/utils/agents/aibitat/providers/anthropic.js`,
`server/__tests__/utils/agents/aibitat/providers/anthropic.test.js`
**Why:** Upstream hardcodes `max_tokens: 4096` in both `stream()` and `complete()`. This
causes truncation on claude-3.5+ models (which support 8192) and would cause a 400 API
error if a future model only supported fewer tokens. Replaced with a `MODEL_MAX_OUTPUT_TOKENS`
lookup table (claude-3.7: 64000, claude-3.5: 8192, legacy: 4096, unknown: 4096 fallback).
Conflict resolved with v1.11.2's added usage tracking — both changes coexist cleanly.
