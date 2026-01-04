# GHOD View – Top 3 Practical Improvements (local `index.html` version)

These are intentionally **small, achievable** upgrades that keep the app fully client-side and don’t require changing the overall architecture.

---

## 1) Safer token persistence (default in-memory / session-only)

**Why**
- Today the PAT is stored indefinitely in `localStorage` (`gh_token`). This is convenient, but it’s the highest-risk part of the local version.

**What to change (minimal scope)**
- Add a **"Remember token on this device"** checkbox on the login screen.
  - Default: **OFF**.
  - If OFF: keep token in React state only (memory). Do **not** write it to `localStorage`.
  - If ON: store it in **`sessionStorage`** (clears on browser close) or keep current `localStorage` behavior if you really need persistence.
- Add a single “Wipe local data” action in Settings that clears:
  - `gh_token`, `daily_notes`, and any new caches (see idea #3)

**Implementation notes (fits current code style)**
- Keep existing storage keys as-is when used (`gh_token`, `daily_notes`) to avoid migration pain.
- If you introduce session storage, you can still *read* legacy `localStorage` tokens to avoid breaking existing users, but prefer writing to session/memory going forward.

**Acceptance checks**
- Refreshing the page does **not** keep you signed in unless “Remember token” is enabled.
- Sign Out always clears in-memory token and any stored token.

---

## 2) Better GitHub failure + rate limit UX (no more generic alert-only)

**Why**
- Currently, most fetch failures end in a generic alert. When GitHub rate limits (403) or auth fails (401), users don’t get actionable guidance.

**What to change (minimal scope)**
- Capture and display key response metadata for the main GitHub calls:
  - `X-RateLimit-Remaining`
  - `X-RateLimit-Reset`
  - response `status` and a short reason bucket:
    - 401/403 → “token invalid/missing scope” vs “rate limited”
- Replace the alert-only pattern with an **inline banner** (top of main content or debug panel) that:
  - says what happened (Auth issue vs Rate limit vs Network)
  - suggests a fix (re-enter token, wait until reset time, etc.)

**Implementation notes**
- In `fetchActivity()`, after each `fetch()`, read headers:
  - `response.headers.get('x-ratelimit-remaining')`
  - `response.headers.get('x-ratelimit-reset')`
- Store this in React state (extend `debugInfo` or add `rateLimitInfo`).
- Detect rate limiting specifically:
  - `status === 403` and remaining is `0` → rate limit exceeded.

**Acceptance checks**
- When token is bad, UI says “Auth failed (401/403)” and doesn’t just say “check your token”.
- When rate limited, UI shows reset time and does not encourage broader scopes.

---

## 3) Persist PR-title backfill cache (reduces API calls and improves consistency)

**Why**
- PR title backfill is great, but right now it’s in-memory only (`prCache`). On refresh, you may repeat up to 15 extra API calls.

**What to change (minimal scope)**
- Store `prCache` in `localStorage` (or `sessionStorage`) with a lightweight TTL and size cap.
  - Example key: `ghod_pr_cache_v1`
  - Store entries like:
    - `{ "owner/repo#123": { title, url, updatedAt } }`
  - On load: hydrate state from storage.
  - On write: prune to N entries (e.g., 300–500) and drop stale entries (e.g., older than 30 days).

**Implementation notes**
- Keep the current backfill limiter (`slice(0, 15)`) and resilience (“ignore per-PR fetch errors”).
- Persisting the cache should be a tiny effect:
  - `useEffect(() => localStorage.setItem(...), [prCache])` with a debounce or simple throttling.

**Acceptance checks**
- Refreshing the page does not trigger repeated PR backfill calls for already-seen PRs.
- Cache never grows without bound.

---

### If you want one ultra-low-effort “bonus” (optional)
- Add a "Refresh" button that re-runs `fetchActivity()` manually, and show “Last updated: <time>”. This is very small and helps when users keep the tab open.
