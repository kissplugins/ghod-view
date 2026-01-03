# The GitHub Daily View You've Always Wanted and Deserved

<img width="1493" height="951" alt="ghod-view" src="https://github.com/user-attachments/assets/0660f062-7cac-4580-88a7-7e5adb7f1ee3" />

A lightweight, client-side GitHub activity aggregator that finally gives you a unified view of your work across all repositories and organizations. No more juggling between repos to answer "what did I do today?"

## Why This Exists

GitHub's native UI makes it frustratingly hard to see your activity across multiple organizations and repositories. This tool solves that problem with a clean, consolidated daily activity feed.

**Built for developers who:**
- Work across multiple GitHub organizations
- Need a quick "what did I ship today/yesterday" view
- Want to track assigned issues in one place
- Prefer simple, client-side tools over heavy SaaS platforms

## Features

✅ **Unified Activity Feed** - All your commits, PRs, issues, and comments in one chronological timeline  
✅ **Multi-Organization Support** - Automatically aggregates activity across all your orgs  
✅ **Assigned Issues Sidebar** - See your 5 most recent assigned issues at a glance  
✅ **Daily Notes** - Add personal notes that persist locally for each session  
✅ **Active Repos Bar** - Quick links to all repositories you've touched  
✅ **Zero Infrastructure** - Pure client-side, no server or database required  
✅ **Privacy First** - Your token stays in your browser's localStorage

## Quick Start

### 1. Clone or Download
```bash
git clone https://github.com/yourusername/ghod-view.git
cd ghod-view
```

### 2. Get a GitHub Personal Access Token
1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "GHOD View")
4. Select the following scopes:
   - ✅ `repo` (Full control of private repositories)
   - ✅ `read:user` (Read user profile data)
   - ✅ `gist` (Create and read gists) - **Required for saving notes to GitHub Gist**
5. Click "Generate token" at the bottom
6. **Important:** Copy the token immediately - you won't be able to see it again!

### 3. Open in Browser
Simply open `index.html` in your browser (double-click or drag to browser).

### 4. Sign In
Paste your token and hit "Sign In". Your activity will load automatically.

## How It Works

**Technology Stack:**
- Pure HTML + React 18 (via CDN)
- Babel Standalone for JSX transformation
- GitHub REST API v3
- localStorage for token and notes persistence

**Architecture:**
- 100% client-side - no backend server
- Single HTML file (~300 lines)
- Token stored locally in browser
- API calls made directly from browser to GitHub

**API Endpoints Used:**
- `GET /user` - Fetch authenticated user info
- `GET /users/{username}/events` - Fetch user's public events
- `GET /issues` - Fetch assigned issues

**Rate Limits:**
- 5,000 requests/hour (authenticated)
- Typically uses 2-3 requests per page load

## Project Structure

```
ghod-view/
├── index.html              # Single-file application (HTML + React + CSS)
├── README.md               # This file
├── LICENSE                 # License file
└── PROJECT-PHASE-1.md      # Original project planning doc
```

## Customization

**Change Date Format:**
```javascript
// Line ~140 in index.html
const formatted = date.toLocaleDateString('en-US', { 
  weekday: 'long', 
  year: 'numeric', 
  month: 'long', 
  day: 'numeric' 
});
```

**Adjust Issue Count:**
```javascript
// Line ~85 in index.html
const issuesResponse = await fetch(
  'https://api.github.com/issues?filter=assigned&state=open&sort=updated&per_page=5',
  { headers }
);
// Change per_page=5 to any number
```

**Modify Event Icons:**
```javascript
// Line ~119 in index.html
const icons = {
  'PushEvent': { label: '➜', class: 'push' },
  'CreateEvent': { label: '+', class: 'create' },
  // Add your own...
};
```

## Security Considerations

⚠️ **Token Storage:** Your Personal Access Token is stored in browser localStorage. This is less secure than server-side storage but acceptable for a client-side tool.

**Best Practices:**
- Use a token with minimal required scopes (`repo`, `read:user`, `gist`)
- Don't use tokens with `admin` or `delete` permissions
- Revoke tokens periodically at [github.com/settings/tokens](https://github.com/settings/tokens)
- Don't share your token or commit it to version control

## Limitations

- Shows last 90 events (GitHub API limit)
- No server-side caching or data persistence beyond localStorage
- Token required for each browser/device
- Rate limited to 5,000 requests/hour

## Future Enhancements (Phase 2+)

Potential features for future iterations:
- [ ] Auto-refresh every N minutes
- [ ] Filter by repository or event type
- [ ] Export activity to Markdown/JSON
- [ ] Dark mode toggle
- [ ] Weekly/monthly summary views
- [ ] Organization-specific filtering
- [ ] Keyboard shortcuts

## Contributing

This is a personal tool but contributions are welcome! Feel free to:
- Open issues for bugs or feature requests
- Submit PRs for improvements
- Fork and customize for your needs

## License

Fair Source License (FSL-1.1-MIT) with 5-user limit - See LICENSE file for details.

**Usage Limitation:** Free for up to 5 users within your organization. For more than 5 users, a commercial license is required.

**MIT Transition:** This project will automatically become MIT licensed on January 2, 2028.

For commercial licensing inquiries: info@hypercart.io

## Support

Having issues? Check these common solutions:

**"Failed to load GitHub data"**
- Verify your token has `repo` and `read:user` scopes
- Check token hasn't expired
- Ensure you're not hitting rate limits

**"No activity showing"**
- GitHub only returns last 90 events
- Make sure you have recent activity
- Try a different date range

**"Issues not loading"**
- Ensure you have issues assigned to you
- Check the `filter=assigned` parameter in API call

---

Built with ☕ by developers, for developers. Because we all deserve better GitHub activity views.

GHOD View © Copyright 2026 Hypercart DBA Neochrome, Inc. | info@hypercart.io
