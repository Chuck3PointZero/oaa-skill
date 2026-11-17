---
name: org-agent-architecture
description: Design and review organizational agent hierarchies under the AGENT -> ROLE -> SKILL -> TOOL convention (one primitive, four kinds, bounded authority that composes down the chain). Use this skill whenever the user wants to create or edit an AGENT.md, ROLE.md, SKILL.md or TOOL.md; define what an agent owns, decides, escalates or must never do; set up or reorganize a repository of agents, roles, skills or tools; wrap an MCP server or API as a tool; split a role into sub-roles or extract a shared skill; or review an agent architecture for ownership conflicts or authority gaps - even if they don't name the convention and just say things like "give this agent a job description", "what is this agent allowed to do on its own" or "add a new tool for X".
allowed-tools: Read Grep Glob LS
---

# Organizational Agent Architecture

OAA structures multi-agent systems the way companies are structured: agents fill roles, roles
require skills, skills require tools, and bounded authority composes down the chain. This skill is
the guidance only. It has no scripts, so every check below is done by reading the files.

## Where things are

| File | What it is |
|---|---|
| `references/model.md` | The binding rules: the model, the fields by kind, how to create a node. Read it before you touch any node. |
| `references/templates.md` | A template for each kind. Copy one; never write frontmatter from memory. |
| `references/example-company.md` | A small valid company (three agents, roles, skills, one tool) to imitate. |
| `references/authority-model.md` | The complete authority algebra: how `never`, `decides` and `escalates` compose, and where each constraint belongs. |
| `references/validation.md` | The checklist for judging whether a structure is valid, and the report format. |

## Which job, which file

| The user wants to | Read |
|---|---|
| Create an agent, role, skill or tool | `model.md`, then `templates.md` |
| See what a finished structure looks like | `example-company.md` |
| Check a structure | `validation.md`, quoting the rule behind every finding |
| Audit or explain what an agent may do | `authority-model.md` (see Authority audit below) |

A **new system** is built bottom-up: tools, then skills, then roles, then agents. **Adding an agent**
means creating whatever is missing, checking that its roles' `owns` do not overlap existing roles,
and validating the result.

## Decomposing

Split only when a trigger fires. Do not decompose speculatively.

- **Split a role** when its `owns` mixes accountabilities whose escalations different humans should
  review (spending money, making creative work, watching the account). Afterwards the `owns` lists
  are disjoint across the whole graph.
- **Extract a shared skill** when two roles would compute or define the same thing (a metric, a
  parsing rule). The definition then exists in exactly one place.
- **A tool is one `TOOL.md`** per MCP server or API, holding its provenance, transport and every
  `never`.
- **Keep the agent singular** until roles need different schedules, models or reviewers. Then the
  split is a one-line move of a `fills` entry.

## Many agents, one role

When one accountability needs many isolated instances (territories, accounts, books, parallel
implementations), fill the role with many agents. Do not fork the role per instance. The role's
`owns`, `decides`, `escalates` and `never` are the shared contract every instance inherits
identically.

## Authority audit

When something went wrong, a role feels too broad, or someone asks "what can this agent do?": read
the agent, follow `fills` to its roles, `requires` to their skills and tools, and compose the
`never`, `decides` and `escalates` along the whole chain using `references/authority-model.md`.
Report what the agent can do autonomously, what triggers an escalation and what is forbidden. Flag any
action discussed in prose but absent from every authority field (a gap), any `never` repeated at a
second layer (drift), and any identifier spelled two ways across layers (an error).

## Common mistakes

- MUST NOT restate a tool's `never` in a skill or role "for safety". It creates drift and adds no
  safety. Delete the copy.
- MUST NOT put decision thresholds on the agent. They belong to the role: the agent is replaceable,
  the role is not.
- MUST NOT put organizational judgement in a skill body ("escalate purchases over $500"). A threshold
  or an ownership decision is a role's line.
- MUST NOT hardcode a model name in agent frontmatter. Model choice is host configuration.
- MUST NOT point a `requires` edge upward or sideways.
- MUST NOT store a credential value in any node file. Env var names only.
- MUST NOT leave a `{{placeholder}}` in a committed node.
- MUST NOT model an external autonomous system as a tool because it is "just an API". If it makes its
  own model calls, plans its own steps or has its own persona, it is an agent with `executor: remote`,
  filling the role its output accountability matches. A tool is dumb by convention.
