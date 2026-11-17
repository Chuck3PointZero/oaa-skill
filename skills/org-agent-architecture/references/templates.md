# Templates

Copy the template for the kind you are creating, then fill every field. Never write node frontmatter from memory.

## AGENT.md

~~~~markdown
---
type: oaa/Agent
kind: agent
name: {{agent-name}}
description: {{One sentence: what this runner does.}}
executor: {{llm | remote | predicate | human}}   # omit for llm. predicate: every skill step must be deterministic. human: never scheduled.
fills:                               # agent-kind spelling of `requires`
  - ../../roles/{{role-name}}        # one line per role; the agent inherits their skills
metadata:
  schedule: "{{cron-or-trigger}}"    # e.g., "0 8 * * *"; omit if event-driven
# NO authority block. The agent inherits the union of its chain's grants,
# bounded by every `never` declared anywhere in it.
# Declaring authority here is almost always a mistake - it belongs on a role.
---

# Agent: {{Agent Name}}

{{Only what's non-obvious from the frontmatter and chain - a known operational
constraint, a deliberate disabled-on-ship reason, an env var that has no obvious
default. If there's nothing to add, delete this section entirely.}}

{{Delete the Memory section below if this agent has no memory file.}}

## Memory

**Path:** `memory/state.json`

Schema and field definitions are in the role's `## Memory` section.

{{Delete the two sections below if they do not apply to this agent.}}

## Startup Credential Check

Required env vars: `{{VAR_A}}`, `{{VAR_B}}`.
If any is missing, escalate immediately and exit - do not run.

## Trigger Model

{{How this agent is woken: its `metadata.schedule`, a human force-run
(`python heartbeat.py --force {{agent-name}}`), a webhook, or an inbound event.
Omit if the schedule alone is the whole story.}}
~~~~

## ROLE.md

~~~~markdown
---
type: oaa/Role
kind: role
name: {{role-name}}                  # lowercase-hyphenated; must match directory name
description: {{One sentence: the accountability this role carries.}}
requires:
  - ../../skills/{{skill-name}}      # the procedures this job needs; one line per skill
watches:
  - {{signal-1}}                     # signals/events this role monitors (e.g., cpa-spikes)
authority:
  owns:
    - {{domain-noun}}                # MUST be disjoint with every other role's owns
  decides:
    - {{bounded-action}}             # autonomous actions, with bounds in the name where possible
                                     # e.g., budget-change-within-20pct-per-24h
                                     # decides: [] is legal - a pure watcher role
  escalates:
    - {{human-threshold}}            # the line where a human decides
  never:
    - {{other-roles-domain}}         # explicit non-ownership; omit field entirely if empty
---

# Role: {{Role Name}}

{{Operating prose - only write this if an authority identifier needs a definition
("stable volume means >= 3 consecutive hours"), or good performance has a shape
a maintainer couldn't derive from the frontmatter. Never restate owns/decides/
escalates/never in prose. If the frontmatter is self-explanatory, one line or nothing.

A ROLE.md with no body beyond frontmatter and this one-line mission is fully
valid. Section headings you might add below - Memory, Decisions Log, or
anything else - are examples of what operating prose CAN contain, not a
required-sections checklist. Add a section only because this specific role
needs it, never because a template or another role's ROLE.md has one.}}

## Memory

`memory/state.json` persists between activations - the runtime state for this role's owned domain.
Omit this section if the role makes no decisions based on prior state.

```json
{{state-schema-example}}
```

{{Field-by-field notes: what each field tracks, when it resets, what a human learns from it.}}

**For `executor: remote` agents:** the runner injects this schema into the system prompt. Without
documented field names, the model invents them. Every remote-executor agent filling this role must
have this section populated.

## Decisions Log

Escalation outcomes accumulate in `decisions/` (one dated file per decision).
Every DEFAULT escalation logged there is a missing line in this file's authority block.
~~~~

## SKILL.md

~~~~markdown
---
type: oaa/Skill
kind: skill
name: {{skill-name}}
description: {{One sentence: the procedure. Written for catalog-level discovery (~100 tokens).}}
allowed-tools:                       # skill-kind spelling of `requires` (agentskills.io compat)
  - ../../tools/{{tool-name}}        # the tool(s) this skill calls
authority:                           # ONLY if this procedure adds its own constraint;
  escalates:                         # otherwise omit the whole block
    - {{procedure-specific-trigger}} # e.g., repeated-tool-failure
---

# Skill: {{Skill Name}}

{{Only write this if the skill's scope or preconditions aren't obvious from the
description. Usually omit - the ## Workflow below is the body.}}

## Workflow

1. {{Step - name the tool function used}}
2. {{Decision point - thresholds as numbers; note which verdicts belong to the
    ROLE (the skill detects the threshold; the role owns it)}}
3. {{Output/handoff - what is written to memory, dispatched, or escalated}}

## Supporting Files

- `scripts/` - deterministic helpers (optional)
- `references/` - loaded on demand
- `assets/` - output templates this procedure fills
~~~~

## TOOL.md

~~~~markdown
---
type: oaa/Tool
kind: tool
name: {{tool-name}}
description: {{One sentence: what this tool does and why it exists.}}
connector: {{api | mcp | local}}
auth: {{api-key | oauth | bearer}}   # connector: api only; omit otherwise
layer: {{sqlalchemy | filesystem | internal-service}}   # connector: local only; omit otherwise
concept-map: {{ontology-map-name}}   # optional: the ontology map block that translates this tool's vocabulary
env: {{ENV_VAR_NAME}}                # the primary credential env var; name only, never the value
                                     # omit entirely if this tool needs no credentials
provenance:                          # required for connector: api and connector: mcp
  source: {{upstream-url-or-internal}}
  status: {{first-party | third-party}}
  vendored: server/vendor/{{package}}
authority:
  never:
    - {{hard-ceiling-or-safety-rule}} # what this tool must never do, regardless of who calls it
---

# Tool: {{Tool Name}}

{{Auth model, known quirks, rate limits, enforcement gaps. Omit anything already
captured in the frontmatter (type, env, provenance, never). If the tool is
straightforward, write only what would surprise a maintainer.}}

## Data Models

{{Optional. Only add this section if a skill or role elsewhere in the graph
defers a schema to this tool ("schema owned by {{tool-name}}"). Document the
field names and types a maintainer would need - this is what that deferral
resolves to. Omit entirely if nothing defers to this tool.}}

## Declared Functions

| Function | Description |
|---|---|
| `{{function_name}}` | {{what it does}} |

## server/ - the implementation

- `server/mcp.json` (or client config) - transport configuration
- `server/proxy/` - enforcement layer applying every `never`
- `server/vendor/{{package}}/` - vendored upstream code
~~~~
