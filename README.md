# AI Skill Files

Skill files that make Claude Code think like a specialist instead of a generalist.

Built and presented at **AI Tinkerers Doha — June 2026** by [Alaa Elbaz](https://www.linkedin.com/in/alaaelbaz/).

---

## What is a skill file?

A skill file is a markdown file that gives Claude a role, a workflow, and rules — before it touches your code.

Instead of prompting Claude and hoping for a useful response, a skill file enforces a structured process. Claude doesn't answer generically. It follows phases, asks the right questions first, and produces consistent output every time.

```
~/.claude/skills/
  your-skill-name/
    SKILL.md        ← the skill definition
```

That's it. Just a folder and a markdown file.

---

## Skills in this repo

### `/design-coach`

A 3-phase design coaching workflow that thinks like a product designer.

**Phase 1 — Research**
Asks 5 UX research questions one at a time. Challenges assumptions that don't align with user behavior. Won't move forward until your concept makes sense.

**Phase 2 — Prototype**
Generates a plain HTML prototype to validate your flow before any real design work begins.

**Phase 2.5 — Template Selection**
Offers 5 real design language references (Figma, Revolut, Spotify, Stripe, Wise) to base your system on — or builds from scratch.

**Phase 3 — Design System**
Builds `design-system.css` with CSS tokens, `components.html` with all states, and `prototype-v2.html` with the system applied.

**Trigger:** `/design-coach` or say "coach my design", "review my design"

---

### `/security-review`

A senior cybersecurity audit workflow that scans like an attacker and fixes like an engineer.

**Phase 0 — Scoping**
Asks 5 questions to build a threat model before touching anything.

**Phase 1 — Scan**
Runs 12 real bash checks: exposed secrets, client-side key exposure, auth gaps, XSS, CSRF, weak crypto, rate limiting, SSRF, dependency vulns, Stripe webhook verification, sensitive data in logs, and CORS misconfig.

**Phase 2 — Report**
Structured report with CWE/OWASP references, exact file:line, code snippets, attack scenarios, and exploitability ratings. Severity: Critical / High / Medium / Low.

**Phase 3 — Fix**
Before/after diffs for every fix. Asks for confirmation before touching any file. Ends with a manual actions list for things only you can change.

**Trigger:** `/security-review` or say "security audit", "check for vulnerabilities"

---

## How to install

```bash
# Clone this repo
git clone https://github.com/alaakodes-kinda/ai-skill-files.git

# Copy skills to your Claude skills directory
cp -r ai-skill-files/skills/design-coach ~/.claude/skills/
cp -r ai-skill-files/skills/security-review ~/.claude/skills/
```

Then in any Claude Code session, type `/design-coach` or `/security-review`.

---

## How to write your own

Every skill file follows the same pattern:

```markdown
---
name: your-skill-name
description: One sentence — when should Claude trigger this skill?
---

# Skill Name

You are a [role]. Your job is [specific mandate].

## Phase 0: [Context gathering]
Ask X before doing anything else.

## Phase 1: [Analysis / Scan / Research]
Do the work. Be specific about what to look for.

## Phase 2: [Report / Output]
Present findings in a structured format. Reference exact locations.

## Phase 3: [Action]
Ask for confirmation. Execute. Verify.
```

**The pattern that makes it work:**
1. **Phase 0 always** — context before output. Generic input = generic output.
2. **Enforce the workflow in the file** — not in the prompt. The file is the memory.
3. **Name the role explicitly** — "You are a senior security engineer" produces different output than "help me with security."

---

## The talk

These skills were built live as part of a 15-minute demo at AI Tinkerers Doha (June 2026).

The thesis: **structure > prompts**. A 30-line skill file outperforms a 500-word prompt because it enforces a process, not just a tone.

---

## License

MIT — take it, modify it, build your own specialist.
