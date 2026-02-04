# Roadmap: Claude Learning Hub

**Created:** 2025-02-03
**Phases:** 5
**Depth:** Standard
**Requirements:** 19 mapped

---

## Phase 1: Foundation

**Goal:** Core data layer and basic browsing capability so links can be stored and viewed with contributor attribution.

**Dependencies:** None (starting phase)

**Requirements:**
- ATTR-01: Each link shows who added it (Slack display name)

**Success Criteria:**
1. User can view a page listing all links in the database
2. Each link displays the Slack display name of the person who added it
3. Database can store links with URL, title, contributor info, timestamps
4. Vue/Inertia pages render correctly on desktop and mobile browsers

**Notes:** Foundation phase is intentionally minimal. Establishes the data layer and proves the stack works before adding complexity. ATTR-01 is placed here because contributor display is fundamental to every link card throughout the application.

---

## Phase 2: Browse Experience

**Goal:** Users can effectively find and explore links through search, filtering, and responsive card-based UI.

**Dependencies:** Phase 1 (database and basic pages)

**Requirements:**
- BRWS-01: User can search links by title and tags
- BRWS-02: User can filter links by Claude topic
- BRWS-03: User can filter links by platform
- BRWS-04: Browse interface is mobile-responsive
- BRWS-05: Links displayed as cards with title, thumbnail, platform icon, contributor
- BRWS-06: User can switch between list view and thumbnail grid view
- BRWS-07: Topic sections show count of links

**Success Criteria:**
1. User can type a search term and see only links matching that term in title or tags
2. User can select a Claude topic (e.g., "Prompting") and see only links in that category, with count displayed
3. User can select a platform (e.g., "YouTube") and see only links from that platform
4. User can toggle between grid view (thumbnails) and list view, with preference persisting
5. Interface remains usable and readable on mobile devices (375px width minimum)

**Notes:** This phase validates the core value proposition: can users actually find and explore links? Placeholders acceptable for thumbnails until Phase 4 delivers metadata extraction.

---

## Phase 3: Slack Capture

**Goal:** Users can add links by posting to Slack, with duplicate prevention and confirmation feedback.

**Dependencies:** Phase 1 (database to store links)

**Requirements:**
- CAPT-01: User can post a link to designated Slack channel and system captures it
- CAPT-02: System prevents duplicate links with friendly "already added" message
- CAPT-03: System confirms successful capture back to user in Slack

**Success Criteria:**
1. User posts a URL to the designated Slack channel and it appears in the browse interface within 30 seconds
2. User posts a URL that already exists and receives a friendly "already added" message in Slack (not an error)
3. User posts a new URL and receives a confirmation message in Slack acknowledging the capture
4. System handles malformed URLs gracefully without crashing

**Notes:** Critical security: Slack webhook signature verification must be implemented before this phase ships. The 3-second Slack response limit requires queue-based processing for metadata (Phase 4), but capture confirmation can be immediate.

---

## Phase 4: Metadata Enrichment

**Goal:** Captured links are automatically enriched with title, thumbnail, platform, and Claude topic categorization.

**Dependencies:** Phase 3 (links must exist to enrich)

**Requirements:**
- META-01: System extracts title from URL via OpenGraph/meta tags
- META-02: System extracts thumbnail/preview image from URL
- META-03: System detects platform from URL (YouTube, TikTok, Instagram, LinkedIn, Twitter, Blog)
- META-04: System auto-categorizes link by Claude topic using AI

**Success Criteria:**
1. When a link is captured, its title is automatically extracted and displayed (fallback to URL if extraction fails)
2. When a link is captured, its thumbnail/preview image is automatically extracted and displayed in the browse interface
3. Links are automatically tagged with their source platform based on URL domain
4. Links are automatically categorized into Claude topics (prompting, Claude Code, MCP, tool use, best practices) using AI analysis
5. Metadata extraction failures are handled gracefully (link still usable with defaults)

**Notes:** Background queue processing is essential here. Extraction runs asynchronously after Slack confirmation. Consider rate limiting for external URL fetches.

---

## Phase 5: Engagement Features

**Goal:** Users can signal what they're researching and endorse valuable links.

**Dependencies:** Phase 2 (browse interface to display status), Phase 1 (users/attribution)

**Requirements:**
- STAT-01: User can mark themselves as "researching" a specific topic
- STAT-02: Browse interface shows who is currently researching each topic
- STAT-03: User can clear their "researching" status when done
- ATTR-02: User can endorse/react to links (+1 or emoji)

**Success Criteria:**
1. User can click a topic and mark themselves as "researching" it
2. Browse interface displays "Chad is researching Prompting" when a user has active research status
3. User can clear their research status with a single action
4. User can react/endorse a link with +1 or emoji, and the count displays on the link card
5. Multiple users can be researching the same topic simultaneously

**Notes:** This is the key differentiator identified in research. Keep it simple: nullable user_id + topic for research status. Reactions can be a simple pivot table with user + link + reaction type.

---

## Phase Progress

| Phase | Name | Status | Requirements | Plans |
|-------|------|--------|--------------|-------|
| 1 | Foundation | Pending | 1 | 0/? |
| 2 | Browse Experience | Pending | 7 | 0/? |
| 3 | Slack Capture | Pending | 3 | 0/? |
| 4 | Metadata Enrichment | Pending | 4 | 0/? |
| 5 | Engagement Features | Pending | 4 | 0/? |

## Requirement Coverage

| Category | Requirements | Phase(s) |
|----------|--------------|----------|
| Capture | CAPT-01, CAPT-02, CAPT-03 | Phase 3 |
| Metadata | META-01, META-02, META-03, META-04 | Phase 4 |
| Browse | BRWS-01 through BRWS-07 | Phase 2 |
| Attribution | ATTR-01, ATTR-02 | Phase 1, Phase 5 |
| Status | STAT-01, STAT-02, STAT-03 | Phase 5 |

**Total:** 19/19 requirements mapped

---

*Roadmap created: 2025-02-03*
*Last updated: 2025-02-03*
