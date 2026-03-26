# Claude Code Build Framework
## The Governing Layer for AI-Assisted Software Development

You bring the idea. The framework selects the right development process, governs it with battle-tested rules, and ensures production readiness.

---

## What This Is

This is a **governing build contract** that sits above the development frameworks you use with Claude Code. It doesn't replace tools like Superpowers, GSD, or BMAD. It orchestrates them, adding the layers they don't have: security architecture, production readiness, hard gates that prevent skipping steps, and stack-specific lessons learned from real builds.

Describe what you want to build. The framework assesses your project's complexity and requirements clarity, then auto-selects the right development framework and build mode. You review, override if needed, and go.

---

## The Problem It Solves

Every AI coding framework solves a different problem:

**[Superpowers](https://github.com/obra/superpowers)** (100k+ stars) excels at structured brainstorming, TDD, and two-stage code review. Best when you know what you're building and quality matters.

**[GSD](https://github.com/gsd-build/get-shit-done)** (23k+ stars) excels at lightweight, incremental development with adversarial plan verification. Best when requirements are unclear and you're discovering the product as you build.

**[BMAD](https://github.com/bmad-code-org/BMAD-METHOD)** excels at full agile team simulation with 9 specialized agents and audit-grade documentation. Best when requirements are locked and compliance matters.

None of them handle security architecture, production deployment, third-party service setup, or the hard-won lessons from specific technology stacks. That's what this framework provides, regardless of which development framework runs underneath.

---

## How It Works

### Step 1: Describe What You Want to Build

Record a voice note, write out your idea, paste a transcript. Any format.

### Step 2: Generate Project Files

Open a new Claude chat. Paste the kickoff prompt template. Attach the master `claude.md` and `prd-template.md`. Paste your project context. Send it.

The framework assesses two dimensions:

**Build complexity:** Express Build (simple apps, one-shot execution) or Full Build (complex apps, phased with verification gates).

**Requirements clarity:** Clear and stable (Superpowers), unclear and experimental (GSD), or locked with compliance needs (BMAD).

It produces four files: a project-specific build contract, a completed PRD, a Claude Code kickoff prompt, and an open questions summary.

### Step 3: Resolve Open Questions (Mandatory Gate)

Claude walks you through every open question interactively. Architecture decisions, auth choices, data model questions, UX decisions. Once all questions are resolved, the files are updated and re-delivered with zero unresolved items. **This gate is skipped for GSD** (GSD discovers requirements incrementally).

### Step 4: Set Up Your Project

Create a project folder. Place `CLAUDE.md` and `prd.md` in the root. Start Claude Code with bypass permissions:

```bash
claude --dangerously-skip-permissions
```

Install the selected framework and common plugins:

```
# Framework (one of these, based on selection):
/plugin install superpowers@claude-plugins-official
# OR: npx get-shit-done-cc --claude --local
# OR: npx bmad-method install

# Common (all projects):
/plugin install context7
/plugin install frontend-design@claude-plugins-official
```

### Step 5: Start the Build

Paste the kickoff prompt. Claude Code reads the build contract and PRD, runs the selected framework's initialization, and produces an architecture plan. For Superpowers and BMAD, it generates setup guides using the three-category model:

- **Category 1 (Human-Only):** Create accounts, create a NEW project in each service, copy base credentials into `.env`. This is the only manual setup you do.
- **Category 2 (Automated):** Claude Code handles everything else via CLI/MCP -- schema creation, auth provider configuration, RLS policies, Edge Function deployment, storage buckets. No dashboard clicking.
- **Category 3 (Post-Build):** Production hardening after the build ships -- custom domains, tier upgrades, security review.

Claude Code presents the Category 1 checklist and waits for your confirmation before writing any code. All MCP/CLI connections are scoped to your specific new project -- never touching existing projects or production data.

### Step 6: Execute

Claude Code works autonomously. Hooks block dangerous operations. The selected framework handles brainstorming, planning, and execution quality. You approve at phase gates (Full Build) or review at the end (Express Build). You can walk away or let it run overnight.

### Step 7: Ship

The freeze audit checklist must pass before deployment (full 69 items for Superpowers/BMAD, lighter MVP readiness for GSD). After shipping, review `lessons-learned.md` and fold systemic patterns back into the master build contract for the next project.

---

## What Each Layer Provides

### This Framework (Always Present)

No matter which development framework is selected, the build contract provides:

- **Security architecture** -- RLS policies, auth patterns (Discord OAuth, Clerk), JWT handling, data isolation, permission enforcement at the data layer
- **Stack-specific lessons learned** -- Dozens of battle-tested patterns for Supabase, React, Railway. The `getSession()` vs `getUser()` deadlock. The two-effect auth initialization. The `.npmrc` cross-platform fix. Every bug fought gets encoded as a mandatory rule.
- **Hard gates** -- Open questions must be resolved before handoff. Pre-build setup must be confirmed. `.env` verified against `.env.example`. These exist because without them, critical steps get forgotten.
- **Automation-first setup** -- Services with CLI/MCP support (Supabase, Railway) are configured programmatically by Claude Code during the build. Setup guides shrink to just account creation and credential copying. All automation is scoped to a new project and never touches production data.
- **Production readiness** -- 70-item freeze audit, setup guide generation for third-party services, deployment discipline, observability requirements, PITR backup guidance
- **Hooks safety layer** -- `pre_tool_use.py` blocks destructive operations even with bypass permissions enabled
- **Build state persistence** -- `STATE.md` survives context compaction and session restarts (Superpowers builds)
- **Component architecture rules** -- Mandatory decomposition, ~200 line file ceiling, extract-then-share protocol
- **Code hygiene** -- React render stability, no context objects in useCallback deps, error/re-render/retry loop prevention

### Superpowers (Default Framework)

Structured brainstorming with formal spec review, implementation planning with bite-sized tasks, subagent-driven execution with two-stage review (spec compliance then code quality), verification-before-completion, systematic debugging. Our discuss phase adds a supplemental UI/UX pass.

### GSD (Experimental/MVP Projects)

Lightweight spec, incremental phase-by-phase planning, parallel research agents, adversarial plan verification, atomic git commits per task, aggressive context rot prevention. Requirements are discovered as you build, not locked upfront.

### BMAD (Enterprise/Compliance Projects)

9 specialized agents (Business Analyst, Product Manager, UX Designer, System Architect, Scrum Master, Developer, QA Engineer, Tech Writer, Solo Dev), formal PRD and architecture creation, epic and story generation, story-by-story implementation with QA review, audit-grade documentation chains.

### Common Plugins (All Frameworks)

**[Context7](https://github.com/upstash/context7)** -- Real-time, version-specific library documentation. Prevents outdated API hallucinations.

**[Frontend Design](https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design)** -- Anthropic's official skill for distinctive, production-grade UI. Auto-activates during frontend work.

---

## Framework Selection Matrix

| Requirements | Simple Project | Complex Project |
|---|---|---|
| **Clear, stable** | Superpowers + Express | Superpowers + Full Build |
| **Unclear, experimental** | GSD + Express | GSD + Full Build |
| **Locked, compliance-grade** | Superpowers + Express | BMAD + Full Build |

The kickoff prompt detects this automatically. You can always override.

---

## The Three Files

| File | Purpose | When to Edit |
|---|---|---|
| `claude.md` | Master build contract -- the governing rules | After each build, when lessons learned are folded back in |
| `prd-template.md` | Blank PRD template | When you want to add new standard sections |
| `kickoff-prompt-template.md` | Prompt that generates project files from raw context | When you change the deliverable structure or workflow |

---

## How the Build Contract Learns

The master `claude.md` is a living document. Every time a build uncovers a new failure pattern, the fix gets encoded as a mandatory rule. During each build, Claude Code writes failures and fixes to `lessons-learned.md`. After shipping, you fold systemic lessons back into the master build contract. The system gets smarter with every project.

---

## Prerequisites

- **Claude Pro, Max, Team, or Enterprise subscription** (for Claude Code access)
- **Claude Code installed** -- [Install instructions](https://code.claude.com)
- **Git + GitHub** for version control
- **Node.js** for React projects
- Accounts for your stack (Supabase, Railway, Discord, etc.)

---

## Adapting for Your Stack

The framework ships with defaults for React + Supabase + Discord OAuth + Railway, but it's designed to be adapted. Replace stack-specific sections with equivalents for your tools while keeping the structural principles: authorization at the data layer, plan before code, lessons learned encoded as rules.

---

## Credits and Acknowledgments

- **[Superpowers](https://github.com/obra/superpowers)** by Jesse Vincent / Prime Radiant -- Development workflow skills for AI coding agents
- **[GSD](https://github.com/gsd-build/get-shit-done)** by TACHES -- Lightweight spec-driven development with context rot prevention
- **[BMAD](https://github.com/bmad-code-org/BMAD-METHOD)** by BMad Code -- Full agile team simulation for enterprise development
- **[Context7](https://github.com/upstash/context7)** by Upstash -- Real-time library documentation MCP server
- **[Frontend Design](https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design)** by Anthropic -- Production-grade frontend design skill
- **[Claude Code](https://code.claude.com)** by Anthropic -- The AI coding agent this framework governs

---

## License

MIT License -- see [LICENSE](LICENSE) for details.

This framework was developed through iterative real-world application builds. You are free to use, adapt, and modify these files for your own projects. If you find it useful, consider contributing your own lessons learned back to the community.
