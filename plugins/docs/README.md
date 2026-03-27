# Documentation Decision Records Skills

> [한국어](README.ko.md)

A collection of skills for Claude Code to create and track Architecture Decision Records (ADR) and Markdown Architectural Decision Records (MADR).

## Skills

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| adr | `/docs:adr` | Create an Architecture Decision Record in Nygard format |
| madr | `/docs:madr` | Create a Markdown Architectural Decision Record in MADR 3.x format |

## Storage Paths

| Skill | Path | Format |
|-------|------|--------|
| `docs:adr` | `docs/adr/NNNN-<title>.md` | Nygard ADR |
| `docs:madr` | `docs/decisions/NNNN-<title>.md` | MADR 3.x |

## Integration with Other Skills

Both skills are designed to work standalone or as part of a larger workflow:

```
superpowers:brainstorming   →  spec document (docs/superpowers/specs/)
superpowers:writing-plans   →  implementation plan
/docs:adr path=<spec>       →  formal ADR document
/docs:madr path=<spec>      →  formal MADR document
```

Pass the spec document path via `path=` argument to automatically extract context, decisions, and options from the spec.

## MADR Variants

`docs:madr` supports three templates selectable via `variant=`:

| Variant | Sections | When to use |
|---------|----------|-------------|
| `minimal` | Title, Status, Context, Decision Outcome | Quick records |
| `standard` | + Decision Drivers, Considered Options, Consequences | **Default** |
| `full` | + Pros and Cons per option | Multiple options comparison |

When `variant=` is not specified, Claude automatically selects based on context.

## Installation

```bash
claude plugin marketplace add https://github.com/ppzxc/engineering-guidelines.git
```
