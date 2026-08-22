# agentrc

Personal agent configuration: shared instructions plus a small set of skills.

## Layout

```
core/     Instructions loaded into every session (AGENTS.md, CLAUDE.md)
skills/   One directory per skill, each with a SKILL.md
```

## Skills

| Skill               | Use when                                                        |
| ------------------- | --------------------------------------------------------------- |
| `architect`         | Settling types and module shape before writing code               |
| `arena`             | Running N attempts at one task and grafting the best parts        |
| `babysit-pr`        | Monitoring a pull request through review and CI                   |
| `blast-radius`      | Checking what a change breaks outside its own diff                |
| `bro`               | Restating the last message in plain language                      |
| `file-pr`           | Filing, opening, or submitting a pull request                     |
| `how`               | Understanding how a subsystem works, or where code belongs        |
| `interrogate`       | Adversarial multi-model review of a change                        |
| `read-postplan`     | Reading a postplan dev URL the user provides                      |
| `teach`             | Explaining work so someone actually understands it                |
| `technical-writing` | Writing or reviewing docs, RFCs, readmes, PR descriptions         |
| `unslop`            | Cutting AI tells from prose                                       |
| `why`               | Digging up why code is shaped the way it is                       |
| `write-html`        | Communicating a plan, spec, or findings as an HTML document       |

## Vendored skills

`architect`, `arena`, `blast-radius`, `bro`, `how`, `interrogate`, `teach`,
`technical-writing`, and `why` are copied from
[cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills).
They were written for Cursor, so the harness-specific parts are rewritten here.
Model slugs map to `opus`, `fable`, and `sonnet`. The
`~/.cursor/rules/pstack-models.mdc` lookups are gone in favor of fixed defaults.
Subagent config uses the Agent tool's parameters. Everything else is upstream
text, so re-pulling will clobber those edits.

Several of them name pstack's `principle-*` skills, which aren't copied here.
Those references read as principles, not as skills you can invoke.
