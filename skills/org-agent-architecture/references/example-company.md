# Example company

A small, valid company: three agents, three roles, three skills and one tool. Read it to see how the four kinds reference each other.

## agents/site-owner/AGENT.md

~~~~markdown
---
type: oaa/Agent
kind: agent
name: site-owner
description: The person who decides what to do when the site has a problem.
executor: human
fills:
  - ../../roles/site-owner
---

# Agent: Site Owner

A person. It has no schedule and is never run. Its compiled `AGENTS.md` is that person's handbook:
the role they hold, what they decide, and how they decide it.
~~~~

## agents/site-watcher/AGENT.md

~~~~markdown
---
type: oaa/Agent
kind: agent
name: site-watcher
description: Checks the site every five minutes with no model, and escalates when it has failed twice in a row.
executor: predicate
fills:
  - ../../roles/site-watcher
metadata:
  schedule: "*/5 * * * *"
---

# Agent: Site Watcher

A predicate agent: its skill is compiled into a script, so nothing here runs a model. It keeps the
previous check's result in `memory/predicate-state.json`.
~~~~

## agents/status-reporter/AGENT.md

~~~~markdown
---
type: oaa/Agent
kind: agent
name: status-reporter
description: Writes the daily account of how the site did, using a model.
executor: llm
models: [tiny]
fills:
  - ../../roles/status-reporter
metadata:
  schedule: "0 8 * * *"
---

# Agent: Status Reporter

Runs on a small model, so compiling it also produces a compact `AGENTS.md`.

## Memory

**Path:** `memory/state.json`. It holds the last 24 hours of check results, as written by the
site watcher.
~~~~

## roles/site-owner/ROLE.md

~~~~markdown
---
type: oaa/Role
kind: role
name: site-owner
description: Accountable for deciding what to do when the site has a problem.
requires:
  - ../../skills/decide-site-escalation
authority:
  owns:
    - site-decisions
  decides:
    - accept-downtime-for-a-window
  escalates:
    - downtime-longer-than-a-day
---

# Role: Site Owner

The site's decision point when something is wrong: it accepts a downtime window when that is the right trade-off, and escalates a problem too large for that - an outage lasting longer than a day.

| Escalation | Goes to |
|---|---|
| `downtime-longer-than-a-day` | a wider decision beyond this role |
~~~~

## roles/site-watcher/ROLE.md

~~~~markdown
---
type: oaa/Role
kind: role
name: site-watcher
description: Accountable for noticing when the site is down, and for telling the owner.
requires:
  - ../../skills/check-site
authority:
  owns:
    - site-health
  decides: []
  escalates:
    - site-down
    - site-probe-unavailable
  never:
    - site-decisions
---

# Role: Site Watcher

A pure watcher: it finds a problem and escalates it, and never acts on the site.

| Escalation | Goes to |
|---|---|
| `site-down` | the site owner |
| `site-probe-unavailable` | the site owner |
~~~~

## roles/status-reporter/ROLE.md

~~~~markdown
---
type: oaa/Role
kind: role
name: status-reporter
description: Accountable for a plain-language daily account of how the site did.
requires:
  - ../../skills/write-status-note
authority:
  owns:
    - status-notes
  decides:
    - write-the-daily-note
  escalates:
    - unclear-outage-cause
  never:
    - site-health
---

# Role: Status Reporter

| Escalation | Goes to |
|---|---|
| `unclear-outage-cause` | the site owner |
~~~~

## skills/check-site/SKILL.md

~~~~markdown
---
type: oaa/Skill
kind: skill
name: check-site
description: Probes the site once and escalates when it has failed on two checks in a row.
allowed-tools:
  - ../../tools/site-probe
authority:
  escalates:
    - site-probe-unavailable
---

# Skill: Check Site

## Workflow

1. Call `probe_site`. If it cannot run, escalate `site-probe-unavailable` and stop.
2. A check **fails** when `status` is not 200, or `ms` is over 5000.
3. Read the previous result from memory. If this check failed and the previous one failed too,
   escalate `site-down` once, and do not escalate again until a check passes.
4. Write this check's result to memory.
~~~~

## skills/decide-site-escalation/SKILL.md

~~~~markdown
---
type: oaa/Skill
kind: skill
name: decide-site-escalation
description: How the site owner handles an escalation by reading it, deciding, and recording the decision.
---

# Skill: Decide Site Escalation

## Workflow

1. Read the escalation and the latest status note.
2. Decide: act now, accept the downtime for a window, or ask for more information.
3. Record the decision with its reason and the date.
~~~~

## skills/write-status-note/SKILL.md

~~~~markdown
---
type: oaa/Skill
kind: skill
name: write-status-note
description: Writes a short plain-language note on how the site did over the last day, from the stored check results.
---

# Skill: Write Status Note

## Workflow

1. Read the last 24 hours of check results from memory.
2. Write at most five sentences: how many checks failed, when, and for how long. Say nothing about a
   cause you do not have evidence for.
3. If the results do not show whether an outage was one event or several, say so in the note instead
   of choosing.
~~~~

## tools/site-probe/TOOL.md

~~~~markdown
---
type: oaa/Tool
kind: tool
name: site-probe
description: Fetches one fixed web page and reports its status code and response time.
connector: local
authority:
  never:
    - request-any-address-except-the-configured-site
    - send-any-method-except-get
---

# Tool: Site Probe

A local script. It reads the address to check from the environment of whatever runs it, so the
address is never written into a node.

## Declared Functions

| Function | Description |
|---|---|
| `probe_site` | One GET of the configured page. Returns `status` (the HTTP status code) and `ms` (the response time). |

## Enforcement

| never | Enforced by |
|---|---|
| request-any-address-except-the-configured-site | `scripts/probe.py` takes no address argument; it reads the one configured address and does not follow a redirect to another host. |
| send-any-method-except-get | `scripts/probe.py` makes a single GET and has no code path for any other method. |
~~~~
