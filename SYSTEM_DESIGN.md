# Liberture — System Design

## Vision Evolution

**Old:** "Biological Operating System" — 6 pillars, gamification, content library, tracking app.

**New:** Open-source platform for building **AI systems focused on human performance**. Combines:
- **Knowledge** (curated protocols, studies, frameworks)
- **Functions** (agent skills, tools, calculators)
- **Community** (open source, contributor-driven)

Think: "OpenClaw for human optimization" — a foundation others can build on.

---

## Core Components

### 1. Knowledge Base
Curated, structured content across performance domains.

```
/knowledge
├── sleep/
│   ├── fundamentals/        # Sleep architecture, cycles, stages
│   ├── protocols/           # CBT-I, sleep restriction, light therapy
│   ├── tools/               # Trackers, apps, devices
│   └── research/            # Key papers, meta-analyses
├── nutrition/
├── exercise/
├── mind/
├── cognition/
└── finance/
```

**Format:** Markdown files with frontmatter (tags, sources, confidence level, last updated).

**Key principle:** First-principles explanations, not surface summaries. Link to primary sources.

### 2. Functions / Skills
Packaged capabilities that AI agents can use.

```
/skills
├── sleep-cycle-calculator/
│   ├── SKILL.md             # Interface spec
│   ├── calculate.js         # Implementation
│   └── test.js
├── macro-calculator/
├── 1rm-calculator/
├── breathing-timer/
└── pomodoro-ultradian/
```

**Each skill has:**
- `SKILL.md` — Description, inputs, outputs, examples
- Implementation (JS/Python/etc.)
- Tests

**Integration:** Skills can be called by OpenClaw agents, embedded in web UI, or used standalone.

### 3. Web Interface
The liberture.com experience.

```
/web
├── index.html               # Landing
├── explore/                 # Browse knowledge base
├── tools/                   # Interactive tools/calculators
├── agents/                  # AI agent interface (chat with Liberture)
└── contribute/              # How to contribute
```

**Design principles:**
- Clean, fast, no bloat
- Dark mode default
- Mobile-first
- Tools work offline (PWA potential)

### 4. Agent Layer
AI agents that use the knowledge + functions.

**Example agents:**
- **Sleep Coach** — Analyzes your schedule, recommends bedtimes, explains sleep science
- **Nutrition Advisor** — Calculates macros, suggests protocols based on goals
- **Habit Builder** — Implementation intentions, habit stacking, accountability

**Architecture:**
- Agents have access to `/knowledge` as context
- Agents can call `/skills` as tools
- Community can build custom agents on the platform

---

## Information Architecture

```
liberture.com/
│
├── /                        # Landing — what is Liberture, quick links
├── /explore                 # Browse knowledge by domain
│   ├── /explore/sleep
│   ├── /explore/nutrition
│   └── ...
│
├── /tools                   # Interactive tools
│   ├── /tools/sleep-calculator
│   ├── /tools/macro-calculator
│   └── ...
│
├── /chat                    # Talk to Liberture AI
│
├── /learn                   # Structured learning paths
│   ├── /learn/sleep-101
│   └── ...
│
├── /contribute              # How to contribute knowledge/skills
│
└── /about                   # Mission, team, open source
```

---

## Tech Stack (Proposed)

- **Content:** Markdown in Git (versioned, forkable)
- **Web:** Static site (Astro/Next.js static export) + client-side JS for tools
- **Agent API:** OpenClaw-compatible skill format
- **Hosting:** Vercel/Cloudflare Pages (free tier viable)
- **Search:** Client-side full-text (Pagefind/Lunr) or Algolia

---

## Differentiators

| Competitors | Liberture |
|-------------|-----------|
| Huberman podcast/notes | Structured, searchable, actionable |
| Random biohacking blogs | Curated, first-principles, sourced |
| Tracking apps (Whoop, Oura) | Knowledge + tools, not just data |
| ChatGPT for health | Domain-specific, higher quality context |

**Core differentiator:** Not another content site. A **platform** that agents and developers can build on.

---

## MVP Scope

**Phase 1: Foundation**
- [ ] Landing page (vision, quick links)
- [ ] 2-3 interactive tools (sleep calculator, macro calculator, breathing timer)
- [ ] Knowledge structure defined (empty folders with README explaining each domain)
- [ ] Contribution guide

**Phase 2: Content**
- [ ] Seed knowledge base with 5-10 articles per domain
- [ ] Link tools to relevant knowledge
- [ ] Basic search

**Phase 3: Agent**
- [ ] Chat interface connected to knowledge base
- [ ] Agent can call tools
- [ ] Community can suggest/contribute skills

---

## Open Questions

1. **Branding:** Keep "Biological Operating System" tagline or evolve messaging?
2. **Content licensing:** CC-BY-SA? MIT? How to handle contributed content?
3. **Agent hosting:** Self-hosted OpenClaw instance or API-based?
4. **León's involvement:** What's he building vs. what do we build?
5. **Existing content:** What from current liberture.com carries over?
