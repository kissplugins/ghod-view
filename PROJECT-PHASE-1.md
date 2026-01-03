# GitHub Activity Viewer - Phase 1 Plan

## Project Overview

**Goal:** Build a lightweight, client-side web application to aggregate and display GitHub activity across multiple organizations and repositories in a unified daily feed view.

**Target User:** Developer working across multiple GitHub organizations who wants a consolidated "what did I do today/yesterday" view that GitHub's native UI doesn't provide.

## Technical Approach

### Architecture Decision: Client-Side First

**Selected Approach:** Pure browser-based application (HTML + React via CDN)

**Rationale:**
- Zero infrastructure/deployment complexity
- Fastest path to working prototype
- GitHub API supports CORS for browser requests
- Can upgrade to Node.js backend later if needed

**Trade-offs Accepted:**
- Personal Access Token stored in browser localStorage (less secure than backend env vars)
- Rate limiting applies to user's personal quota (5000 req/hour with auth)
- No server-side caching or scheduled updates
- Limited to browser storage for data persistence

## Phase 1 Scope

### Core Features

1. **Authentication**
   - GitHub Personal Access Token input
   - Token stored in localStorage
   - Clear token functionality

2. **Activity Aggregation**
   - Fetch authenticated user's events via `/users/{username}/events`
   - Fetch organization-specific events via `/users/{username}/events/orgs/{org}`
   - Merge and deduplicate events from multiple sources
   - Group events by date (day-level granularity)

3. **Display Interface**
   - Chronological timeline view (newest first)
   - Day-grouped activity sections
   - Event type badges (PushEvent, PullRequestEvent, IssueCommentEvent, etc.)
   - Repository names as links
   - Basic event details (commit messages, PR titles, issue numbers)

4. **User Experience**
   - Manual refresh button
   - Loading states
   - Error handling and display
   - Date range selector (today, yesterday, last 7 days, last 30 days)

### Out of Scope for Phase 1

- Automated refresh/polling
- Advanced filtering (by repo, event type, organization)
- Search functionality
- Data export
- Notification integration
- Webhook support
- Multi-user support
- Backend server
- Database persistence

## Technical Specifications

### Tech Stack

**Core:**
- HTML5
- React 18 (via CDN - no build step)
- Babel Standalone (for JSX transformation in browser)
- Vanilla CSS (no framework dependencies)

**APIs:**
- GitHub REST API v3
- Endpoints: `/users/{username}/events`, `/users/{username}/events/orgs/{org}`

### File Structure

```
gh-activity-viewer/
├── index.html              # Single-file application
└── README.md              # Setup instructions
```

### Data Model

**Event Object (from GitHub API):**
```javascript
{
  id: string,
  type: string,              // PushEvent, PullRequestEvent, etc.
  actor: {
    login: string,
    avatar_url: string
  },
  repo: {
    name: string,
    url: string
  },
  payload: object,           // Event-specific data
  created_at: string,        // ISO 8601 timestamp
  org: object               // Optional, if org event
}
```

**Grouped Activity Structure:**
```javascript
{
  "2026-01-02": [
    { /* event 1 */ },
    { /* event 2 */ }
  ],
  "2026-01-01": [
    { /* event 3 */ }
  ]
}
```

### GitHub API Integration

**Authentication:**
- Personal Access Token with scopes: `repo`, `read:org`
- Passed via `Authorization: token {TOKEN}` header

**Rate Limiting:**
- 5000 requests/hour with authentication
- Check `X-RateLimit-Remaining` header
- Display remaining quota in UI

**API Calls Pattern:**
```javascript
// 1. Fetch user events (public + private)
GET https://api.github.com/users/{username}/events

// 2. For each organization
GET https://api.github.com/users/{username}/events/orgs/{org}

// 3. Merge results, deduplicate by event.id
```

**Limitations:**
- Events limited to last 90 days
- Max 300 events per endpoint
- Public events vs authenticated events differ

### Component Architecture

**Main Components:**

1. **App** (root component)
   - State management for token, activity data, loading, errors
   - Orchestrates data fetching
   - Renders child components

2. **TokenInput**
   - Controlled input for PAT
   - Save/clear token actions
   - Token visibility toggle

3. **ActivityFeed**
   - Receives grouped activity data
   - Renders DaySection components
   - Handles empty states

4. **DaySection**
   - Displays single day's events
   - Date header
   - List of EventItem components

5. **EventItem**
   - Formats individual event for display
   - Event type icon/badge
   - Repository link
   - Event-specific details (commit message, PR title, etc.)

6. **Controls**
   - Refresh button
   - Date range selector
   - Organization filter (multi-select)

### Event Type Handling

**Priority Event Types (Phase 1):**
- `PushEvent` - Show commit count, messages
- `PullRequestEvent` - Show PR title, action (opened/closed/merged)
- `IssueCommentEvent` - Show issue title, comment preview
- `IssuesEvent` - Show issue title, action
- `CreateEvent` - Show created ref (branch/tag)
- `DeleteEvent` - Show deleted ref
- `WatchEvent` - Show starred repo
- `ForkEvent` - Show forked repo

**Display Strategy:**
- Map event types to human-readable labels
- Extract relevant payload data per type
- Truncate long text (commit messages, comments)
- Link to GitHub for full details

## Implementation Plan

### Step 1: Minimal Viable Display (2 hours)

**Deliverable:** Single HTML file that fetches and displays basic event data

**Tasks:**
1. Create `index.html` with React/Babel CDN includes
2. Build basic `App` component with state hooks
3. Implement `fetchActivity()` function for `/users/{username}/events`
4. Create simple `TokenInput` component with localStorage persistence
5. Display raw event JSON in `<pre>` tags grouped by day
6. Add manual refresh button
7. Basic error handling (network errors, auth failures)

**Success Criteria:**
- Can input PAT and fetch events
- Events display grouped by day
- Manual refresh works
- Token persists across page reloads

### Step 2: Event Formatting & UX (2-3 hours)

**Deliverable:** Polished event display with proper formatting

**Tasks:**
1. Build `EventItem` component with type-specific formatters
2. Add event type badges/icons (using emoji or simple CSS)
3. Extract and display meaningful data per event type
4. Link repository names to GitHub
5. Add loading spinner during fetch
6. Implement error state UI (invalid token, rate limit exceeded)
7. Add basic CSS styling (clean, readable timeline)

**Success Criteria:**
- Events display in human-readable format
- Different event types have appropriate formatting
- UI provides clear feedback for loading/error states
- Visual hierarchy makes scanning easy

### Step 3: Multi-Org Support (1-2 hours)

**Deliverable:** Fetch and merge events from multiple organizations

**Tasks:**
1. Add organization input (comma-separated list)
2. Store org list in localStorage
3. Implement parallel fetching for multiple orgs
4. Merge and deduplicate events by `event.id`
5. Sort merged results by `created_at` descending
6. Display org name in event items where applicable

**Success Criteria:**
- Can specify multiple organizations
- Events from all orgs appear in unified feed
- No duplicate events
- Performance acceptable with 3-5 orgs

### Step 4: Date Filtering & Polish (1-2 hours)

**Deliverable:** Production-ready Phase 1 release

**Tasks:**
1. Add date range selector (today, yesterday, last 7/30 days)
2. Filter displayed events by selected range
3. Add rate limit remaining indicator
4. Implement "clear cache" button (clear localStorage)
5. Add help text/tooltips for setup
6. Final CSS polish for responsive layout
7. Write README with setup instructions

**Success Criteria:**
- Can filter by date ranges
- Rate limit status visible
- Mobile-friendly layout
- Clear documentation for first-time users

## Risk Assessment

### Technical Risks

**Risk:** GitHub API rate limiting
- **Mitigation:** Display remaining quota, warn before exhaustion, cache results in sessionStorage
- **Severity:** Medium - affects power users checking frequently

**Risk:** Token security in browser storage
- **Mitigation:** Document risk, recommend read-only token with minimal scopes
- **Severity:** Medium - acceptable for local dev tool, not for production service

**Risk:** Browser CORS restrictions
- **Mitigation:** GitHub API already supports CORS; if issues arise, can proxy via backend
- **Severity:** Low - well-tested pattern

### Usability Risks

**Risk:** Complex token setup for non-technical users
- **Mitigation:** Step-by-step instructions with screenshots in README
- **Severity:** Low - target user is developer

**Risk:** Overwhelming UI with high activity volume
- **Mitigation:** Pagination, collapsible days, event type filtering (Phase 2)
- **Severity:** Medium - can iterate based on usage

## Success Metrics

**Phase 1 Goals:**
1. **Functional:** Successfully fetches and displays activity for 1+ organizations
2. **Usable:** Can answer "what did I do yesterday?" in under 10 seconds
3. **Maintainable:** Codebase simple enough to extend in Phase 2
4. **Deliverable:** Works as standalone HTML file, no build/deploy required

**Validation:**
- Developer can open file, input token, see activity within 30 seconds
- Activity from last 7 days displays accurately
- UI is scannable and doesn't require scrolling to find recent work

## Phase 2 Preview (Future Scope)

**Potential Enhancements:**
- Vite migration for better DX
- Advanced filtering (repo, event type, author)
- Search across activity
- Export to Markdown/CSV
- Dark mode
- Keyboard shortcuts
- Browser extension version
- Optional backend for caching/scheduled updates
- Integration with time tracking tools
- Commit stats/analytics dashboard

## Development Timeline

**Total Estimated Effort:** 6-9 hours

**Week 1:**
- Day 1-2: Steps 1-2 (Minimal viable display + formatting)
- Day 3: Step 3 (Multi-org support)
- Day 4: Step 4 (Polish + documentation)

**Delivery:** Single `index.html` file ready to use

## Getting Started Checklist

**Prerequisites:**
- [ ] GitHub account with Personal Access Token
- [ ] Token scopes: `repo`, `read:org`
- [ ] List of organization names to track
- [ ] Modern browser (Chrome, Firefox, Safari, Edge)

**Development Setup:**
- [ ] Create project directory
- [ ] Create `index.html` from template
- [ ] Test token authentication
- [ ] Verify CORS requests work
- [ ] Begin Step 1 implementation

## Notes

**Development Philosophy:**
- Bias towards simplicity and shipping
- Perfect is enemy of good - get working prototype, then iterate
- Client-side first, backend only if clearly needed
- Document assumptions and limitations
- Build for developer audience (yourself as user zero)

**Key Decisions:**
- Single file = easy to share, version, and backup
- No build step = fast iteration, no toolchain complexity
- localStorage = good enough for personal tool
- React via CDN = familiar DX without webpack

**This document is a living plan** - update as implementation reveals new insights or requirements change.