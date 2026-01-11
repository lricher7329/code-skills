# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is the **coding skills** source repository. Skills created here sync to `~/.claude/skills/coding/` for global use across projects.

## Repository Structure

```
code-skills/
├── skills/       # Reusable coding skills (each skill has SKILL.md)
├── patterns/     # Domain-specific reference patterns
├── agents/       # Coding agents
└── commands/     # Coding commands
```

## Skill Format

Each skill lives in its own directory with a `SKILL.md` file that defines:
- Skill purpose and when to use it
- Required inputs and context
- Step-by-step instructions for Claude Code
- Expected outputs

## Syncing

Skills sync to the global working copy via:
```bash
~/.claude/skills/sync-skills.sh coding
```

## Creating New Skills

Use the meta skill-creator:
```
~/.claude/skills/meta/skill-creator/SKILL.md
```

## Available Skills

| Skill | Purpose | Invocation |
|-------|---------|------------|
| `plan` | Full implementation planning with PR breakdown, input gathering, and testing | "use plan skill" |
| `plan-lite` | Lightweight focused planning for single features, bug fixes | "use plan-lite skill" |
| `audit` | Read-only codebase quality and consistency review | "use audit skill" |
| `debug` | Investigate bugs, trace root causes, produce minimal fixes | "use debug skill" |
| `refactor` | Behavior-preserving structural improvements | "use refactor skill" |
| `review` | Review PRs/diffs with structured feedback | "use review skill" |

## Domain Patterns

Domain-specific patterns are in `patterns/`:
- `clinical-research-patterns.md` - Healthcare, clinical trials, research data (truth vs documentation, extraction pipelines, provenance, visit scheduling)

## Skill Components

Skills may include additional files beyond `SKILL.md`:
- `TEMPLATE.md` - Handlebars template for programmatic/orchestrator use (e.g., `plan/TEMPLATE.md`)

## Relationship to Writing Skills

This repo is the coding counterpart to `~/writing/skills-repo/`. Both sync to `~/.claude/skills/` but serve different domains:
- **This repo**: Software development, tooling, automation
- **Writing repo**: Grants, manuscripts, protocols, clinical documents
