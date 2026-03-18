---
name: obsidian-openclaw-memory
description: >
  Set up Obsidian + OpenClaw as a persistent AI memory system for software development
  and business/work contexts. Use when helping configure a workspace so the AI
  remembers project context, decisions, and work tasks across sessions. Covers
  vault setup, secure file structure, QMD semantic search, and heartbeat-based
  memory maintenance with trusted-zone enforcement.
---
 
# Obsidian + OpenClaw Memory System
## (Software Dev & Business — Balanced Security Profile)
 
## Overview
 
The AI doesn't *have* memory — it *reads* memory. OpenClaw injects workspace files
into the system prompt at session start, giving the AI persistent context across
sessions. Obsidian visualizes the knowledge graph. QMD provides semantic search so
the AI finds relevant context without loading everything.
 
**Three components:**
 
- **OpenClaw** — reads workspace files at session start (injected into system prompt)
- **Obsidian** — vault pointed at the workspace; Graph View shows connections
- **QMD** — on-device semantic search; finds relevant files without loading them all
 
---
 
## Security Model (Balanced)
 
This workspace uses a **two-zone trust model**. Files in the **trusted zone** are
injected into the system prompt and treated as authoritative instructions. Files in
the **intake zone** are read-only reference material and are never injected as
instructions.
 
### Trusted Zone (system-prompt level)
Only files you author directly. Never paste raw external content here.
 
```
MEMORY.md
AGENTS.md
SOUL.md
USER.md
HEARTBEAT.md
directives/
projects/
```
 
### Intake Zone (reference only — never injected as instructions)
External content, pasted docs, client materials, third-party specs.
 
```
second-brain/docs/
second-brain/inbox/
```
 
**Rule:** If you paste content from the web, an email, or a client doc — it goes in
`second-brain/inbox/`, not in any trusted-zone file. The AI reads it as data, not
as instructions.
 
### File Permissions
After initial setup, lock trusted-zone files to prevent silent overwrites:
 
```bash
chmod 644 ~/clawd/MEMORY.md ~/clawd/SOUL.md ~/clawd/AGENTS.md ~/clawd/USER.md
chmod 644 ~/clawd/HEARTBEAT.md
chmod -R 644 ~/clawd/directives/
```
 
---
 
## File Structure
 
```
~/clawd/
├── MEMORY.md              # Curated long-term memory (distilled from logs)
├── USER.md                # Your context: role, preferences, working style
├── AGENTS.md              # How the AI should operate in this workspace
├── SOUL.md                # AI persona and tone
├── HEARTBEAT.md           # Drives proactive maintenance (see below)
│
├── memory/
│   └── YYYY-MM-DD.md      # Raw daily session logs
│
├── projects/
│   └── <project-name>.md  # Per-project context: stack, decisions, status, open Qs
│
├── directives/
│   └── *.md               # SOPs, workflows, coding standards, recurring processes
│
└── second-brain/
    ├── docs/              # Reference material (not injected as instructions)
    └── inbox/             # Untrusted / external content — read as data only
```
 
---
 
## Project Files (`projects/<name>.md`)
 
For dev and business work, per-project files are the core memory unit. Each should
contain:
 
```markdown
# Project: <name>
 
## Stack
- Language / framework / infra
 
## Key Decisions
- YYYY-MM-DD: <decision and rationale>
 
## Current Status
<one paragraph>
 
## Open Questions
- [ ] item
 
## Contacts / Stakeholders
<names and roles>
```
 
Reference project files from `MEMORY.md` with a wiki-link: `[[projects/my-app]]`.
The AI will be prompted to keep project files updated as work progresses.
 
---
 
## Obsidian Setup
 
1. **Create vault** pointing to `~/clawd` (File → Open Vault → Open Folder as Vault)
2. **Enable Graph View** (Ctrl/Cmd+G)
3. **Install plugins:**
   - **Dataview** — query project and memory files
   - **Templater** — daily note templates
4. **Daily note template** (Templater):
 
```
# <% tp.date.now("YYYY-MM-DD") %>
 
## Sessions / Projects Touched
 
## Decisions Made
 
## Things to Remember
 
## Blockers / Open Questions
```
 
5. **Useful Dataview queries:**
 
```dataview
LIST FROM "memory" SORT file.name DESC LIMIT 7
```
 
```dataview
TABLE status, last-updated FROM "projects"
```
 
---
 
## QMD Semantic Search Setup
 
```bash
# Index the workspace
qmd collection add ~/clawd --name clawd
 
# Generate embeddings (run after adding/changing files)
qmd embed
 
# Search from within OpenClaw
mcporter call qmd.vsearch query="what did we decide about the auth system"
mcporter call qmd.query query="project X current status"
```
 
**After adding significant content**, re-embed:
 
```bash
qmd embed
```
 
Consider adding a post-save hook or a nightly cron alongside the heartbeat so
embeddings stay current without manual runs.
 
**Note on context limits:** OpenClaw has a finite context window. If the workspace
grows large, QMD search is how the AI accesses older context — it will not silently
load everything. When uncertain whether context is complete, ask: "What project files
do you currently have loaded?"
 
---
 
## Heartbeat-Based Memory Maintenance
 
`HEARTBEAT.md` drives proactive AI behavior on a schedule. OpenClaw polls it and
acts on pending tasks autonomously.
 
**Because this runs autonomously**, keep `HEARTBEAT.md` in the trusted zone, author
it yourself, and don't let the AI rewrite it without your review. The AI should
*complete* tasks listed in HEARTBEAT, not *add new ones* to it unprompted.
 
### Example `HEARTBEAT.md`
 
```markdown
## Memory Maintenance
- [ ] Review memory/ logs from last 3 days
- [ ] Distill key decisions into MEMORY.md
- [ ] Update project status sections in projects/*.md
- [ ] Flag any open questions that have been unresolved > 1 week
- [ ] Remove stale or superseded entries from MEMORY.md
 
## Business / Work
- [ ] Summarize any client or stakeholder notes from second-brain/inbox/
- [ ] Check directives/ for any SOPs that need updating based on recent work
```
 
**Schedule:** Every 2–3 days for active projects. Weekly otherwise.
 
```bash
# Example cron (every 3 days at 8am)
0 8 */3 * * openclaw heartbeat ~/clawd
```
 
---
 
## MEMORY.md Guidelines
 
This is the single most important file. Keep it signal-dense, not a dump.
 
**Do:**
- Distilled decisions with dates
- Project status one-liners
- Key preferences and working patterns
- Wiki-links to detail: `[[projects/my-app]]`, `[[directives/deploy-sop]]`
 
**Don't:**
- Paste raw content from external sources
- Let it grow unbounded — prune stale entries regularly (heartbeat handles this)
- Duplicate what's already in a project file
 
---
 
## Best Practices
 
1. **Write it down** — if you want the AI to remember something next session, say
   "write this to memory" or update the relevant project file directly.
2. **One project file per project** — keep decisions and status there, not scattered
   in daily logs.
3. **Daily logs are raw** — `memory/YYYY-MM-DD.md` is for session notes; don't curate them.
4. **External content goes in inbox** — never paste client docs or web content into
   trusted-zone files.
5. **Re-embed after major additions** — run `qmd embed` after adding files to
   `second-brain/` or creating new project files.
6. **Link files in Obsidian** — use `[[file]]` wiki-links to build the graph and
   let Dataview queries work across the vault.
7. **Audit HEARTBEAT.md periodically** — make sure it only contains tasks you
   intentionally put there.
 
