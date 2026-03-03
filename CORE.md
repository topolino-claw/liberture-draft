# Liberture — Core Concept

## One-liner

**Community-driven skills and tools for OpenClaw, organized around human performance.**

## The Idea

OpenClaw is exploding. People are building AI agents that do real things.

Liberture is a **skill library** focused on the 6 pillars of human optimization:
- 😴 Sleep & Recovery
- 🥗 Nutrition
- 🏋️ Exercise
- 🧠 Cognition & Work
- 🧘 Mind & Stress
- 💰 Finance

Anyone can contribute. Everything is OpenClaw-compatible. Copy a skill into your agent and it just works.

## Why This Works

1. **Riding the wave** — OpenClaw has momentum. Liberture is symbiotic, not competing.
2. **Community-driven** — Skills come from the community. We curate and organize.
3. **Practical value** — Day one, you can add a sleep calculator or macro tracker to your agent.
4. **Low barrier** — Skills are just markdown + simple code. Anyone can contribute.
5. **Vertical focus** — Not "all skills" — specifically human performance. Clear identity.

## What Liberture Is NOT

- ❌ A standalone app
- ❌ A tracking platform
- ❌ A content site with articles
- ❌ A replacement for OpenClaw

## What Liberture IS

- ✅ A skill library (OpenClaw-compatible)
- ✅ A community hub for biohacking tools
- ✅ A directory organized by the 6 pillars
- ✅ Templates and examples for creating new skills
- ✅ Documentation on integrating skills with your agent

---

## The 6 Pillars

| Pillar | Focus | Example Skills |
|--------|-------|----------------|
| 😴 **Sleep** | Cycles, recovery, circadian rhythm | Sleep calculator, jet lag planner, sleep debt tracker |
| 🥗 **Nutrition** | Macros, fasting, hydration | Macro calculator, fasting timer, hydration tracker |
| 🏋️ **Exercise** | Strength, cardio, mobility | 1RM calculator, workout logger, rest timer |
| 🧠 **Cognition** | Focus, productivity, learning | Pomodoro timer, spaced repetition, focus tracker |
| 🧘 **Mind** | Stress, meditation, resilience | Breathing timer, meditation guide, mood logger |
| 💰 **Finance** | Savings, investing, independence | FIRE calculator, savings rate, expense tracker |

---

## Anatomy of a Liberture Skill

Every skill follows OpenClaw's format:

```
skills/
└── sleep-cycle-calculator/
    ├── SKILL.md          # Description, interface, examples
    ├── calculate.js      # Implementation (or .py, .sh, etc.)
    ├── test.js           # Tests
    └── README.md         # Human-readable docs
```

### SKILL.md Structure

```markdown
# Sleep Cycle Calculator

**Pillar:** Sleep  
**Author:** @username  
**Version:** 1.0.0

## Description
Calculate optimal bedtimes based on 90-minute sleep cycles.

## Interface

### Inputs
- `wake_time` (string, HH:MM) — When you need to wake up
- `fall_asleep_minutes` (number, optional, default 15) — Time to fall asleep

### Outputs
- `bedtimes` (array) — List of optimal bedtimes with cycle count

## Example

Input:
```json
{ "wake_time": "07:00" }
```

Output:
```json
{
  "bedtimes": [
    { "time": "21:45", "cycles": 6, "duration": "9h 0m" },
    { "time": "23:15", "cycles": 5, "duration": "7h 30m", "optimal": true },
    { "time": "00:45", "cycles": 4, "duration": "6h 0m" }
  ]
}
```

## Installation

Copy this folder to your OpenClaw skills directory:
```bash
cp -r sleep-cycle-calculator ~/.openclaw/skills/
```

Or reference directly in your agent config.
```

---

## User Journey

### For Users (Installing Skills)

1. Browse liberture.com → find a skill
2. Copy to `~/.openclaw/skills/` (or git clone)
3. Your agent now has that capability

### For Contributors (Creating Skills)

1. Fork the liberture repo
2. Use the skill template
3. Implement, test, document
4. Submit PR
5. We review, tag by pillar, merge

---

## Website Structure

```
liberture.com/
│
├── /                    # Hero + explain the concept
├── /skills              # Browse all skills
│   ├── /skills/sleep
│   ├── /skills/nutrition
│   └── ...
├── /create              # How to create a skill (template + guide)
├── /integrate           # How to integrate with OpenClaw
└── /community           # Contributors, Discord, etc.
```

---

## MVP Priorities

### Phase 1: Foundation
- [ ] Landing page that explains the concept clearly
- [ ] Skill template (copy-paste ready)
- [ ] 3-5 example skills (one per pillar minimum)
- [ ] Integration guide for OpenClaw

### Phase 2: Directory
- [ ] Skill browser by pillar
- [ ] Search/filter
- [ ] Contributor profiles

### Phase 3: Community
- [ ] Submission flow
- [ ] Review process
- [ ] Versioning

---

## Open Questions

1. **Hosting:** GitHub org (liberture/) or monorepo?
2. **Skill format:** Strict OpenClaw SKILL.md or our own variant?
3. **Quality control:** Open submissions or curated?
4. **Nodes:** What does "nodes" mean in this context? Physical devices? Community members?
