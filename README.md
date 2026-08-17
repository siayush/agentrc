# agentrc

Personal agent configuration: shared instructions plus a small set of skills.

## Layout

```
core/     Instructions loaded into every session (AGENTS.md, CLAUDE.md)
skills/   One directory per skill, each with a SKILL.md
```

## Skills

| Skill           | Use when                                                    |
| --------------- | ----------------------------------------------------------- |
| `file-pr`       | Filing, opening, or submitting a pull request               |
| `babysit-pr`    | Monitoring a pull request through review and CI             |
| `write-html`    | Communicating a plan, spec, or findings as an HTML document |
| `read-postplan` | Reading a postplan dev URL the user provides                |
