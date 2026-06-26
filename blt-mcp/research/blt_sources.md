# OWASP BLT Sources

## Source 1: OWASP BLT Main Repository

- **URL**: https://github.com/OWASP-BLT/BLT
- **Title**: OWASP-BLT/BLT - Bug Logging Tool
- **Author/Organization**: OWASP Foundation / OWASP-BLT Community
- **Summary**: Main repository for the OWASP Bug Logging Tool (BLT) — a gamified crowd-sourced QA testing and vulnerability disclosure platform. Allows users to report bugs across websites, apps, and git repositories. 310+ stars, AGPL-3.0 license.
- **Important Technical Details**:
  - Primary language: HTML (with Django backend)
  - Key features: QA testing, vulnerability disclosure, bug bounties, gamification (leaderboards, badges), staking system (BACON/blockchain), AI-powered tools
  - Community-driven with automated leaderboard bot
  - CSRF configuration examples in settings
  - GitHub Actions workflows for leaderboard generation
  - Uses Black, isort, ruff for code quality
- **Why It Is Useful**: This is the core project that BLT-MCP serves. Understanding its features, data model, and workflows is essential for building the MCP interface.

---

## Source 2: OWASP BLT Official Project Page

- **URL**: https://owasp.org/www-project-bug-logging-tool/
- **Title**: OWASP Bug Logging Tool - OWASP Foundation
- **Author/Organization**: OWASP Foundation
- **Summary**: OWASP project page for BLT with release history, feature overview, and community links.
- **Important Technical Details**:
  - Release History:
    - v2.1 (Nov 2025): Dark mode, bounty payouts, security labs, hackathon features
    - v2.0 (Mar 2025): Major redesign, notification system, Slack integration, BACON rewards, security labs
    - v1.5 (Jun 2024): AI chatbot, website monitoring, cryptocurrency payments
    - v1.4 (Mar 2024): New homepage, sidebar navigation, improved UI/UX
  - Became OWASP production project in May 2023
  - Participating in GSoC since 2023
  - Bug verification is community-driven
  - Companies can launch sponsored bug bounties
- **Why It Is Useful**: Provides project context, version history, and understanding of BLT's evolution.

---

## Source 3: OWASP BLT Documentation Repository

- **URL**: https://github.com/OWASP-BLT/BLT-docs
- **Title**: OWASP-BLT/BLT-docs - Official Documentation
- **Author/Organization**: OWASP-BLT
- **Summary**: Official documentation for OWASP BLT. Includes getting started guides, feature overviews, user guides, and developer documentation.
- **Important Technical Details**:
  - Deployed to GitHub Pages
  - Sections: Getting Started, Features, User Guide (auth, bug reporting, leaderboards, company management, bug hunts), Developer Guide (contributing, bot setup, dev environment)
  - Content sourced from main BLT repo's `docs/` and `website/documents/` directories
- **Why It Is Useful**: Developer guide is critical for understanding how to contribute to BLT and integrate with it.

---

## Source 4: OWASP BLT Website

- **URL**: https://owaspblt.org
- **Title**: OWASP BLT Website
- **Author/Organization**: OWASP-BLT
- **Summary**: Live instance of the OWASP BLT platform for bug reporting and community engagement.
- **Important Technical Details**:
  - Legacy site: https://legacy.owaspblt.org
  - Live platform for filing bugs and participating in bounties
- **Why It Is Useful**: The live platform that BLT-MCP will interface with. Understanding the user-facing side informs MCP server design.

---

## Source 5: OWASP-BLT/BLT-API

- **URL**: https://github.com/OWASP-BLT/BLT-API
- **Title**: OWASP-BLT/BLT-API - Full-featured REST API
- **Author/Organization**: OWASP-BLT
- **Summary**: Edge-deployed REST API for BLT built with Python on Cloudflare Workers. Uses D1 (SQLite) for data persistence. Provides global, low-latency access to all BLT data.
- **Important Technical Details**:
  - Edge-deployed on Cloudflare Workers
  - D1 Database (SQLite) for persistence
  - CORS enabled, API key authentication (X-BLT-API-Key header)
  - Endpoints: `/bugs`, `/users`, `/domains`, `/organizations`, auth, health
  - Versioned under `/v2` prefix
  - Static API key authentication (public routes: /, /v2, /health remain open)
  - GET /routes for programmatic API discoverability
- **Why It Is Useful**: BLT-MCP will likely consume this REST API internally. Understanding its endpoints, auth model, and data structures is essential.

---

## Source 6: OWASP-BLT/BLT-Next

- **URL**: https://github.com/OWASP-BLT/BLT-Next
- **Title**: OWASP-BLT/BLT-Next - Next Generation Architecture
- **Author/Organization**: OWASP-BLT
- **Summary**: Next-generation architecture for BLT migrating from Django monolith to static frontend on GitHub Pages with Cloudflare Python Workers.
- **Important Technical Details**:
  - Static frontend + Cloudflare Workers backend
  - Features: bug reporting, leaderboard, JWT authentication, user profiles, projects, rewards
  - HTMX for seamless UI interactions
  - Part of the modernization initiative
- **Why It Is Useful**: BLT-Next represents the future architecture of BLT. BLT-MCP should align with this direction.

---

## Source 7: OWASP-BLT/BLT-Pages

- **URL**: https://github.com/OWASP-BLT/BLT-Pages
- **Title**: OWASP-BLT/BLT-Pages - Pages Based UI
- **Author/Organization**: OWASP-BLT
- **Summary**: Community-powered bug-reporting platform built on GitHub Pages. Supports public bug reports, anonymous reporting, and security vulnerability reports via BLT-Zero.
- **Important Technical Details**:
  - Pure HTML + Tailwind CSS (CDN)
  - GitHub Actions for leaderboard generation (every 6 hours)
  - BLT-API for anonymous report submission
  - Structured YAML bug report template
- **Why It Is Useful**: Demonstrates the lightweight, static-first approach that BLT is adopting.

---

## Source 8: OWASP BLT GSoC 2026 Landing Page

- **URL**: https://gsoc.owaspblt.org/
- **Title**: OWASP BLT GSoC Landing Page
- **Author/Organization**: OWASP-BLT
- **Summary**: Google Summer of Code landing page for OWASP BLT, showcasing 2026 projects, mentors, and application information.
- **Important Technical Details**:
  - GSoC 2026 is active with accepted students
  - Lists all project ideas and assigned mentors
  - Application template and guidelines
- **Why It Is Useful**: Provides context for BLT-MCP as an official GSoC 2026 project idea, including mentorship and timeline.

---

## Source 9: OWASP BLT Organization on GitHub

- **URL**: https://github.com/orgs/OWASP-BLT
- **Title**: OWASP-BLT GitHub Organization
- **Author/Organization**: OWASP-BLT
- **Summary**: GitHub organization containing 53+ public repositories related to the BLT ecosystem.
- **Important Technical Details**:
  - 95 followers, verified organization
  - Key repos: BLT (main), BLT-API, BLT-Next, BLT-Flutter, BLT-Rewards, BLT-MCP, BLT-Ideas, BLT-GSOC, BLT-Pages
  - Email: blt-support@owasp.org
- **Why It Is Useful**: Comprehensive view of the entire BLT ecosystem. BLT-MCP must integrate with multiple sibling projects.

---

## Source 10: OWASP-BLT/BLT-Rewards

- **URL**: https://github.com/OWASP-BLT/BLT-Rewards
- **Title**: OWASP-BLT/BLT-Rewards - BACON Token System
- **Author/Organization**: OWASP-BLT
- **Summary**: BACON is a Bitcoin-based token system (using Runes protocol) for incentivizing contributions within the BLT ecosystem.
- **Important Technical Details**:
  - Bitcoin-based token via Runes protocol
  - Integrates with Bitcoin Core
  - Gamified reward system for contributors
- **Why It Is Useful**: BLT-MCP's Tools layer includes `award_bacon` functionality. Understanding the rewards system is essential.
