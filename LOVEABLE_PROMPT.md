# LLM Prompt for Reproducing GHOD View in Loveable (or similar app builders)

## Project Overview
Build a single-page web application called "GHOD View" (GitHub Daily Activity Stream) - a lightweight, client-side GitHub activity aggregator that provides a unified view of developer work across all repositories and organizations.

## Core Problem Statement
GitHub's native UI makes it difficult to see activity across multiple organizations and repositories. This tool solves that with a clean, consolidated daily activity feed that answers "what did I do today/yesterday?"

## Technology Stack
- **Frontend Framework**: React 18 with TypeScript (or vanilla React if TypeScript not supported)
- **Styling**: CSS with CSS variables for theming (GitHub-inspired design system)
- **API Integration**: GitHub REST API v3 (client-side only, no backend)
- **Storage**: Browser localStorage for token and notes persistence
- **Build Tool**: Vite or similar modern bundler

## Required GitHub API Scopes
The app requires a GitHub Personal Access Token with these scopes:
- `repo` - Full control of private repositories
- `read:user` - Read user profile data  
- `gist` - Create and read gists (for saving notes)

## Core Features & Components

### 1. Authentication System
**Login Screen:**
- Centered card layout with logo, title, and subtitle
- Password input field for GitHub Personal Access Token
- "Sign In" button (Enter key support)
- Help text with link to generate token and required scopes
- Store token in localStorage as `gh_token`

**Authenticated State:**
- Display username in header (from `/user` endpoint)
- "Sign Out" button that clears token and resets state

### 2. Header Component
- Left side: Logo image + "GHOD View" title + "GitHub Daily Activity Stream" subtitle
- Right side: Username emoji + username, calendar emoji + today's date, Sign Out button
- White background with bottom border

### 3. Active Repositories Bar
- Horizontal bar below header showing unique repos from current day filter
- Bubble/chip style links (14px, light blue background `#ddf4ff`)
- Alphabetically sorted
- Clickable links to GitHub repos
- Updates based on day filter selection

### 4. Day Filter Bar
- Toggle buttons: "Today" and "Previous Day"
- Active button has blue background (`#0969da`)
- Settings gear icon to toggle debug panel (optional feature)
- Shows current selected day label

### 5. Main Activity Feed (Left Column, 2/3 width)
**Day Section Cards:**
- White card with border for each day
- Day header: Full date format (e.g., "Friday, January 2, 2026")
- Activity items with:
  - Circular colored icon (32px) based on event type:
    - Push: Blue `#0969da` with "➜"
    - Create: Green `#1a7f37` with "+"
    - Delete: Red `#cf222e` with "×"
    - Issues: Yellow `#d29922` with "○"
    - Member: Purple `#8250df` with "★"
  - Event type label (small, grey)
  - Repository name as grey pill chip (14px, `#f6f8fa` background)
  - Branch link (for push/create/delete branch events)
  - Pull Request title link (for PR events)
  - Event description with commit message summary
  - Timestamp (12-hour format)

**Event Type Handling:**
- **PushEvent**: Show first commit message + count of additional commits
- **CreateEvent**: "Created new [branch/tag] 'name'"
- **DeleteEvent**: "Removed deprecated [branch/tag] 'name'"
- **IssuesEvent**: "Opened issue: [title]"
- **IssueCommentEvent**: "Commented on issue"
- **PullRequestEvent**: Show PR title with link

### 6. Sidebar (Right Column, 1/3 width, max 380px)

**My Notes for Today Card:**
- Textarea for daily notes (min-height 120px)
- Auto-save to localStorage as `daily_notes` on change
- "Copy to Clipboard" button with success feedback
- "Save to GitHub Gist" button:
  - Creates private gist with filename `ghod-notes-YYYY-MM-DD.md`
  - Opens gist in new tab on success
  - Shows loading state while saving
- Privacy notice: "These notes will only appear on this computer"

**Assigned Issues Card:**
- Shows 5 most recently updated assigned issues
- Each issue displays:
  - Issue title (clickable link)
  - Repository name as small pill chip (12px, `#f6f8fa` background)
  - Issue number in grey badge

## API Integration Details

### Endpoints to Use:
1. `GET https://api.github.com/user` - Get authenticated user info
2. `GET https://api.github.com/users/{username}/events?per_page=100&page={page}` - Fetch user events (paginate up to 3 pages)
3. `GET https://api.github.com/issues?filter=assigned&state=open&sort=updated&per_page=5` - Fetch assigned issues
4. `GET https://api.github.com/repos/{owner}/{repo}/pulls/{number}` - Fetch PR details for missing titles (backfill)
5. `POST https://api.github.com/gists` - Create gist for notes

### Authentication Header:
```javascript
headers: { 'Authorization': `token ${githubToken}` }
```

### Event Pagination Logic:
- Fetch up to 3 pages of 100 events each
- Stop early once you have events from a day before today
- This ensures "Previous Day" filter works correctly

### Local Timezone Day Grouping:
- Group events by local date (not UTC)
- Use `new Date(event.created_at).toLocaleDateString('en-CA')` for YYYY-MM-DD keys
- Sort day keys in reverse chronological order

### PR Title Backfill:
- Some PR events don't include titles in the event payload
- For missing titles, fetch from `/repos/{owner}/{repo}/pulls/{number}`
- Cache fetched titles to avoid duplicate API calls
- Limit to 15 backfill requests to preserve rate limits
- Use fallback `#123` format if fetch fails

## Design System (CSS Variables)

```css
:root {
  --fs-lg: 24px;   /* Large text (headings) */
  --fs-md: 16px;   /* Medium text (body) */
  --fs-sm: 14px;   /* Small text (metadata) */
  --fs-xs: 12px;   /* Extra small (labels, chips) */
}
```

**Color Palette:**
- Background: `#f6f8fa`
- Card background: `white`
- Border: `#d0d7de`
- Text primary: `#24292f`
- Text secondary: `#57606a`
- Link blue: `#0969da`
- Success green: `#1a7f37`
- Warning yellow: `#d29922`
- Error red: `#cf222e`
- Purple: `#8250df`

**Typography:**
- Font family: `-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica', 'Arial', sans-serif`
- Use CSS variables for all font sizes
- Maintain 4-tier size system

## Layout Structure
```
┌─────────────────────────────────────────────────────┐
│ Header (logo, title, username, date, sign out)     │
├─────────────────────────────────────────────────────┤
│ Active Repos Bar (chip links)                      │
├─────────────────────────────────────────────────────┤
│ Day Filter Bar (Today/Previous Day buttons)        │
├─────────────────────────────────────────────────────┤
│ ┌─────────────────────┬─────────────────────────┐ │
│ │ Main Activity Feed  │ Sidebar                 │ │
│ │ (2/3 width)         │ (1/3 width, max 380px)  │ │
│ │                     │ ┌─────────────────────┐ │ │
│ │ Day Section Card    │ │ My Notes for Today  │ │ │
│ │ - Activity items    │ │ - Textarea          │ │ │
│ │                     │ │ - Copy button       │ │ │
│ │                     │ │ - Gist button       │ │ │
│ │                     │ └─────────────────────┘ │ │
│ │                     │ ┌─────────────────────┐ │ │
│ │                     │ │ Assigned Issues     │ │ │
│ │                     │ │ - Issue list        │ │ │
│ │                     │ └─────────────────────┘ │ │
│ └─────────────────────┴─────────────────────────┘ │
├─────────────────────────────────────────────────────┤
│ Footer (copyright, email, license link)            │
└─────────────────────────────────────────────────────┘
```

## State Management

### React State Variables:
- `token` - GitHub PAT (synced with localStorage)
- `user` - Authenticated user object from `/user` endpoint
- `activity` - Object grouped by day keys: `{ "2026-01-02": [events...], ... }`
- `assignedIssues` - Array of issue objects
- `notes` - Daily notes text (synced with localStorage)
- `loading` - Boolean for loading state
- `dayFilter` - "today" | "previous"
- `prCache` - Object mapping `"owner/repo#123"` to `{ title, url }`
- `copySuccess` - Boolean for clipboard feedback
- `gistSuccess` - Boolean for gist save feedback
- `savingGist` - Boolean for gist save loading state

### localStorage Keys:
- `gh_token` - GitHub Personal Access Token
- `daily_notes` - User's daily notes content

## User Flow

### Initial Load:
1. Check localStorage for `gh_token`
2. If no token: Show login screen
3. If token exists: Auto-fetch activity and show main interface

### Login Flow:
1. User enters GitHub PAT
2. Click "Sign In" or press Enter
3. Store token in localStorage
4. Fetch user info, events, and assigned issues
5. Show main interface

### Activity Fetching:
1. Fetch authenticated user (`/user`)
2. Fetch events with pagination (up to 3 pages of 100 events)
3. Group events by local day key (YYYY-MM-DD)
4. Fetch assigned issues
5. Backfill missing PR titles (up to 15 requests)

### Day Filter:
- "Today" shows events from current local date
- "Previous Day" shows events from yesterday (or nearest earlier day if yesterday has no events)
- Active Repos bar updates to show repos from filtered day

### Notes:
- Auto-save to localStorage on every keystroke
- Copy button uses `navigator.clipboard.writeText()`
- Gist button creates private gist and opens in new tab

## Error Handling
- Show alert if GitHub API calls fail
- Display "Failed to load GitHub data. Please check your token." message
- Gracefully handle missing PR titles with `#123` fallback
- Ignore individual PR backfill failures (don't break the feed)
- Handle empty states: "No activity for [date]" when no events

## Performance Considerations
- Limit event fetching to 3 pages (300 events max)
- Limit PR title backfill to 15 requests
- Use React.useEffect cleanup for cancelled async operations
- Cache PR titles to avoid duplicate API calls

## Optional Debug Features (Nice to Have)
- Settings panel to toggle debug UI
- Show day keys and fetch errors in debug panel
- Typography inspector overlay (hover to see font properties)
- These are optional and can be omitted for MVP

## Responsive Design
- Max container width: 1400px
- Main content: flex 2
- Sidebar: flex 1, max-width 380px
- Cards have consistent padding (16-20px)
- All interactive elements have hover states

## Accessibility
- Semantic HTML (header, main, aside, footer)
- Alt text for logo image
- ARIA labels where appropriate
- Keyboard navigation support (Enter key on login)
- Focus states on interactive elements

## Testing Checklist
- [ ] Login with valid GitHub token
- [ ] Auto-load on page refresh
- [ ] Sign out clears token and state
- [ ] Activity feed shows grouped by day
- [ ] Today/Previous Day filters work correctly
- [ ] Active Repos bar updates with filter
- [ ] Branch links work for push/create/delete events
- [ ] PR titles display (with backfill for missing ones)
- [ ] Assigned issues load in sidebar
- [ ] Notes auto-save to localStorage
- [ ] Copy to clipboard works
- [ ] Save to Gist creates private gist and opens in new tab
- [ ] Empty states display correctly
- [ ] Error handling shows appropriate messages

## Rate Limiting
- GitHub API: 5,000 requests/hour (authenticated)
- Typical usage: 2-5 requests per page load
- PR backfill adds up to 15 additional requests
- Total: ~20 requests per full page load (well within limits)

## Security Notes
- Token stored in browser localStorage (client-side only)
- No server-side storage or transmission
- Warn users to use minimal scopes
- Recommend periodic token rotation
- Never commit tokens to version control

## Future Enhancements (Out of Scope for MVP)
- Auto-refresh every N minutes
- Filter by repository or event type
- Export activity to Markdown/JSON
- Dark mode toggle
- Weekly/monthly summary views
- Organization-specific filtering
- Keyboard shortcuts
- Browser extension version

## Asset Requirements
- Logo image: `hyperview-octo.png` (40x40px container, image scales to fit)
- Favicon (optional)

## Footer Content
```
GHOD View © Copyright 2026 Hypercart | info@hypercart.io | FSL 5 License
```

---

## Implementation Notes for Loveable

When building this in Loveable or similar app builders:

1. **Start with the login screen** - Get authentication working first
2. **Build the header and layout structure** - Establish the visual hierarchy
3. **Implement activity fetching and display** - Core functionality
4. **Add day filtering** - Today/Previous Day toggle
5. **Build the sidebar** - Notes and assigned issues
6. **Add Active Repos bar** - Dynamic repo chips
7. **Implement PR title backfill** - Enhanced UX
8. **Polish with error handling and loading states**

The app should be fully functional as a single-page application with no backend required. All data fetching happens client-side directly to GitHub's API.

**Key Success Criteria:**
- Clean, GitHub-inspired UI
- Fast, responsive interactions
- Reliable day filtering with timezone awareness
- Graceful error handling
- Persistent notes and token storage


