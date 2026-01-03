# Changelog

All notable changes to GHOD View will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-02

### Added
- **Complete GitHub Activity Feed** - Unified view of all GitHub activity across organizations
  - Daily grouped activity with formatted dates (e.g., "Friday, January 2, 2026")
  - Activity type badges with color-coded circular icons
  - Event descriptions showing commits, branches, and issue details
  - Timestamp for each activity
  - Clickable repository links

- **User Authentication**
  - Login screen with GitHub Personal Access Token input
  - Token stored securely in localStorage
  - Auto-load activity on page refresh
  - Sign Out functionality
  - Display authenticated user's GitHub username in header

- **My Notes for Today**
  - Textarea for daily notes with localStorage persistence
  - Copy to Clipboard button with success feedback
  - Save to GitHub Gist button (creates private gist with date)
  - Privacy notice about local-only storage

- **Assigned Issues Sidebar**
  - Shows 5 most recently updated assigned issues
  - Issue titles with clickable links
  - Repository names and issue numbers
  - Card-based design matching main interface

- **Active Repositories Bar**
  - Displays unique repositories from recent activity
  - Alphabetically sorted with clickable links
  - Positioned below header for easy navigation

- **Professional UI/UX**
  - GitHub-inspired design with clean cards and spacing
  - Branded header with logo, title, and date
  - Responsive layout with sidebar
  - Color-coded activity icons (blue=push, green=create, red=delete, yellow=issues, purple=member)
  - Footer with copyright and license link

- **Documentation**
  - Comprehensive README with setup instructions
  - Quick start guide with step-by-step token generation
  - Technical architecture details
  - Security considerations
  - Customization examples
  - Troubleshooting section

### Changed
- **License** - Switched from Apache 2.0 to Fair Source License (FSL-1.1-MIT)
  - Free for up to 5 users per organization
  - Commercial license required for 6+ users
  - Automatic MIT transition after 2 years (January 2, 2028)
  - Contact: info@hypercart.io

### Technical Details
- **Stack**: Pure HTML + React 18 (CDN) + Babel Standalone
- **API**: GitHub REST API v3
- **Storage**: localStorage for token and notes
- **Architecture**: 100% client-side, single HTML file
- **Required Scopes**: `repo`, `read:user`, `gist`

### Fixed
- Merge conflict resolution in CSS styling
- Missing `</ul>` closing tags in activity items
- Date formatting issues
- GitHub username dynamic fetching

### Security
- Token stored in browser localStorage only
- No server-side storage or transmission
- Minimal scope requirements documented
- Periodic token rotation recommended
- Warning against admin/delete permissions

---

## Future Roadmap (Phase 2+)

Potential features for future releases:
- Auto-refresh every N minutes
- Filter by repository or event type
- Export activity to Markdown/JSON
- Dark mode toggle
- Weekly/monthly summary views
- Organization-specific filtering
- Keyboard shortcuts

---

GHOD View © Copyright 2026 Hypercart DBA Neochrome, Inc. | info@hypercart.io
