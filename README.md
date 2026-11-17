# oaa-skill

A single Claude skill for Organizational Agent Architecture (OAA): structuring a multi-agent system
the way a company is structured. Agents fill roles, roles require skills, skills require tools, and
bounded authority composes down that chain, so every agent has a written answer to what it owns, what
it may decide on its own, and what it must escalate to a human.

This is the guidance-only edition. It is plain Markdown with no scripts, no hooks and no MCP server.
The full toolkit, with a compile and lock utility and an ontology server, lives in
[Chuck3PointZero/OAA](https://github.com/Chuck3PointZero/OAA).

## What it does

Ask Claude to create an `AGENT.md`, `ROLE.md`, `SKILL.md` or `TOOL.md`, to split a role, to wrap an
API as a tool, or to say what an agent is allowed to do on its own. The skill supplies the rules, the
templates, a worked example company, the authority algebra and a validation checklist, and Claude
applies them by reading your files.

## Install

```bash
npx skills add Chuck3PointZero/oaa-skill
```

## Contents

- `skills/org-agent-architecture/SKILL.md` - the entry point
- `references/model.md` - the binding rules and fields by kind
- `references/templates.md` - one template per kind
- `references/example-company.md` - a small valid company
- `references/authority-model.md` - how authority composes
- `references/validation.md` - the review checklist

## License

MIT
