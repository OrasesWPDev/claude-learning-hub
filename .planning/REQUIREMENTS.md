# Requirements: Claude Learning Hub

**Defined:** 2025-02-03
**Core Value:** Frictionless mobile capture — if adding a link is hard, people won't do it

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Capture

- [ ] **CAPT-01**: User can post a link to designated Slack channel and system captures it
- [ ] **CAPT-02**: System prevents duplicate links with friendly "already added" message
- [ ] **CAPT-03**: System confirms successful capture back to user in Slack

### Metadata

- [ ] **META-01**: System extracts title from URL via OpenGraph/meta tags
- [ ] **META-02**: System extracts thumbnail/preview image from URL
- [ ] **META-03**: System detects platform from URL (YouTube, TikTok, Instagram, LinkedIn, Twitter, Blog)
- [ ] **META-04**: System auto-categorizes link by Claude topic using AI (prompting, Claude Code, MCP, tool use, best practices)

### Browse

- [ ] **BRWS-01**: User can search links by title and tags
- [ ] **BRWS-02**: User can filter links by Claude topic
- [ ] **BRWS-03**: User can filter links by platform
- [ ] **BRWS-04**: Browse interface is mobile-responsive
- [ ] **BRWS-05**: Links displayed as cards with title, thumbnail, platform icon, contributor
- [ ] **BRWS-06**: User can switch between list view and thumbnail grid view
- [ ] **BRWS-07**: Topic sections show count of links ("Prompting (47)")

### Attribution

- [ ] **ATTR-01**: Each link shows who added it (Slack display name)
- [ ] **ATTR-02**: User can endorse/react to links (+1 or emoji)

### Status

- [ ] **STAT-01**: User can mark themselves as "researching" a specific topic
- [ ] **STAT-02**: Browse interface shows who is currently researching each topic
- [ ] **STAT-03**: User can clear their "researching" status when done

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Enhanced Capture

- **CAPT-10**: Smart link detection — auto-capture ANY link posted to channel (not just explicit mentions)

### Enhanced Browse

- **BRWS-10**: "New since last visit" indicator for returning users
- **BRWS-11**: Full-text search across link content (not just title/tags)

### Content Enhancement

- **META-10**: AI-generated summary of linked content
- **META-11**: Auto-extract key timestamps from video content

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Wiki/notes features | Scope creep killer — 70% of wikis fail. We're a link hub, not Notion |
| Separate user accounts | Team already in Slack. Separate accounts = friction |
| Content hosting/media storage | We link to content, don't host it. Avoids storage/rights issues |
| Comments/discussion threads | Discussion belongs in Slack where team communicates |
| Complex permissions/RBAC | Everyone can view, everyone can add. That's it |
| Browser extension | Slack-first means Slack-only for v1 |
| Notification system | Use Slack's native notifications |
| Gamification (badges, leaderboards) | Feels corporate, creates perverse incentives |
| RSS/feed generation | Validate core first, add if requested |
| AI-generated summaries (v1) | Latency, quality varies — defer to v2 |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| CAPT-01 | TBD | Pending |
| CAPT-02 | TBD | Pending |
| CAPT-03 | TBD | Pending |
| META-01 | TBD | Pending |
| META-02 | TBD | Pending |
| META-03 | TBD | Pending |
| META-04 | TBD | Pending |
| BRWS-01 | TBD | Pending |
| BRWS-02 | TBD | Pending |
| BRWS-03 | TBD | Pending |
| BRWS-04 | TBD | Pending |
| BRWS-05 | TBD | Pending |
| BRWS-06 | TBD | Pending |
| BRWS-07 | TBD | Pending |
| ATTR-01 | TBD | Pending |
| ATTR-02 | TBD | Pending |
| STAT-01 | TBD | Pending |
| STAT-02 | TBD | Pending |
| STAT-03 | TBD | Pending |

**Coverage:**
- v1 requirements: 19 total
- Mapped to phases: 0
- Unmapped: 19 ⚠️

---
*Requirements defined: 2025-02-03*
*Last updated: 2025-02-03 after initial definition*
