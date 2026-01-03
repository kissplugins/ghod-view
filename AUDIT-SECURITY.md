# GHOD View Security Audit

Date: 2024-11-23  
Scope: Single-page React client (`index.html`) that calls GitHub REST APIs directly from the browser. No backend components in this repo.

## Key Findings
- **High – Token persistence in localStorage** (`index.html`): Personal Access Tokens (PATs) are stored indefinitely in `localStorage`, making them readable by any script that runs in the origin (e.g., malicious CDN response, injected script). This exposes full `repo`/`gist` privileges if the browser is compromised.
- **High – Unpinned third-party scripts without integrity** (`index.html`): React, ReactDOM, and Babel are loaded from `unpkg` without Subresource Integrity (SRI) and rely on CDN-hosted code. A tampered CDN response would have access to the stored PAT and issue data.
- **Medium – Inline Babel compilation plus permissive scripting surface** (`index.html`): Inline JSX compiled at runtime requires `unsafe-inline` in CSP and keeps all application logic directly on the page, increasing the blast radius of any script injection and making it harder to apply strict CSP.
- **Medium – Broad PAT scope guidance** (`README.md`): Documentation recommends classic tokens with `repo`, `gist`, and `read:user`. `repo` grants wide write access (including private repos); device/OAuth flows or fine-grained tokens are not offered.
- **Medium – Sensitive notes stored unencrypted** (`index.html`): User notes persist in `localStorage` alongside the token. If the origin is ever hosted on a domain with other scripts, both notes and tokens are exposed to any injected JS.
- **Low – Limited error and abuse handling** (`index.html`): API errors (rate limit, 401) are shown generically; there is no backoff or warning on repeated failures, which can create a poor UX and encourage users to escalate scopes unnecessarily.

## Recommendations (Prioritized)
1) **Stop storing the PAT in `localStorage`; prefer in-memory/session storage.** Provide a “Remember this device” opt-in that stores only in `sessionStorage` (clears on browser close) or, ideally, keeps tokens purely in memory after entry. Add a “clear token” control and automatic token wipe on inactivity.
2) **Vendor and pin dependencies; remove runtime Babel.** Replace CDN React/ReactDOM/Babel imports with pinned versions in the repo or a local build artifact, and add SRI if any CDN is retained. Run a build step (Vite/Rollup/webpack) to transpile JSX ahead of time and serve a minimal JS bundle.
3) **Add a strict Content Security Policy.** Example once inline scripts are removed: `default-src 'none'; script-src 'self'; style-src 'self'; connect-src https://api.github.com; img-src https: data:; font-src 'self' data:; base-uri 'none'; form-action 'none'; frame-ancestors 'none';`. Include `rel="noopener noreferrer"` on all external links (already present on most) and avoid inline event handlers.
4) **Switch to fine-grained or OAuth device-flow tokens with least privilege.** Document and enforce scopes narrowly (e.g., read-only repo access unless writing gists is enabled). Detect missing scopes at runtime and give precise remediation instead of asking for broad `repo`.
5) **Harden local data handling.** Namespace storage keys, consider encrypting notes with a user-supplied passphrase, and add a “Wipe local data” control that clears both notes and tokens in one click.
6) **Improve error handling and user messaging.** Detect rate limiting (HTTP 403 with `X-RateLimit-Remaining: 0`) and show guidance instead of retrying blindly; surface 401/403 as authentication/scope issues without encouraging broader scopes.

## Quick Wins
- Add a “Remember token on this device” toggle that defaults to off and keeps the token in memory/session only.
- Pin React/ReactDOM versions with SRI (or self-hosted files) and remove Babel Standalone by introducing a minimal build step.
- Ship a CSP tailored to the final asset list and keep inline scripts/styles out of the page.
- Update README to recommend fine-grained or device-flow tokens with minimal scopes and to warn explicitly about localStorage risks.
