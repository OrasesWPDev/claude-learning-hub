# Claude Learning Hub

## What This Is

A team knowledge base for Claude and Claude Code learning resources. Team members can quickly capture links from anywhere (Instagram, YouTube, TikTok, LinkedIn, etc.) via Slack while on their phones, and the system auto-organizes them by topic and platform. A browsable web interface lets the team filter, explore, and signal when they're diving deeper into specific topics.

## Core Value

Frictionless mobile capture — if adding a link is hard, people won't do it. Everything else builds on this.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Slack integration for link capture (post link to channel, system captures it)
- [ ] Auto-extract metadata from URLs (title, thumbnail, platform detection)
- [ ] Auto-categorize by Claude topic (prompting, Claude Code, MCP, tool use, best practices)
- [ ] Browsable web interface with filtering by topic and platform
- [ ] Simple "researching" status signaling (e.g., "Chad is researching prompt engineering")
- [ ] Works for mixed-technical team (non-technical users can browse and add)

### Out of Scope

- Note-taking or annotation features — just link collection and status
- Mobile app — Slack is the mobile capture mechanism
- User accounts beyond Slack identity — Slack auth is sufficient
- Content hosting — we link to external resources, don't store media
- AI-generated summaries of content — keep it simple for v1

## Context

**Learning goal:** This project doubles as a Laravel learning opportunity. Code quality and Laravel best practices matter as much as shipping features.

**Team environment:** Company Slack workspace with full admin access. Team is mixed technical/non-technical, so the browsing interface needs to be approachable.

**Content scope:** Exclusively Claude-related content — prompting techniques, Claude Code usage, MCP servers, tool use, best practices, tips and tricks.

## Constraints

- **Stack:** Laravel + Vue.js (Inertia.js) — learning Laravel is a project goal
- **Hosting:** Laravel Forge + DigitalOcean — Laravel's standard deployment path
- **Integration:** Must work with existing company Slack workspace
- **Audience:** Interface must be usable by non-technical team members

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Laravel + Vue (Inertia) | Learning Laravel is a goal; Inertia provides seamless Laravel/Vue integration | — Pending |
| Slack for capture | Frictionless mobile input; team already uses Slack; full admin access available | — Pending |
| Forge + DigitalOcean | Laravel's official deployment path; simple, well-documented | — Pending |
| No user accounts | Slack identity is sufficient; reduces complexity | — Pending |

---
*Last updated: 2025-02-03 after initialization*
