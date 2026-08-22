# agentrc

Personal agent configuration: shared instructions plus a small set of skills.

## Layout

```
core/     Instructions loaded into every session (AGENTS.md, CLAUDE.md)
skills/   One directory per skill, each with a SKILL.md
```

## Skills

All skills are slash-command-only (`disable-model-invocation: true`) except `unslop`, which the agent applies to prose on its own.

| Skill                           | Use when                                                            |
| ------------------------------- | ------------------------------------------------------------------- |
| `arena`                         | Running N attempts at one task and grafting the best parts          |
| `babysit-pr`                    | Monitoring a pull request through review and CI                     |
| `blast-radius`                  | Checking what a change breaks outside its own diff                  |
| `bro`                           | Restating the last message in plain language                        |
| `code-review`                   | Reviewing a diff against repo standards and its originating spec    |
| `codebase-design`               | Designing a module's interface, seams, and depth                    |
| `diagnosing-bugs`               | Diagnosing a hard bug or performance regression                     |
| `domain-modeling`               | Sharpening the domain model, CONTEXT.md, or ADRs                    |
| `file-pr`                       | Filing, opening, or submitting a pull request                       |
| `grill-with-docs`               | Grilling a plan while capturing ADRs and glossary as you go         |
| `grilling`                      | Stress-testing a plan or decision through relentless questions      |
| `how`                           | Understanding how a subsystem works, or where code belongs          |
| `improve-codebase-architecture` | Scanning a codebase for deepening opportunities                     |
| `interrogate`                   | Adversarial multi-model review of a change                          |
| `prototype`                     | Building a throwaway prototype to answer a design question          |
| `read-postplan`                 | Reading a postplan dev URL the user provides                        |
| `teach`                         | Learning a topic across sessions with lessons and practice          |
| `technical-writing`             | Writing or reviewing docs, RFCs, readmes, PR descriptions           |
| `unslop`                        | Cutting AI tells from prose                                         |
| `wizard`                        | Generating a bash wizard for manual setup steps only a human can do |
| `write-html`                    | Communicating a plan, spec, or findings as an HTML document         |
