# Source playbooks

The why skill runs on what this machine can reach with no credentials: the git object store, the GitHub API through `gh`, and the files in the tree. Two playbooks cover that ground, one per investigator.

| Category | Playbook | What it reaches |
|---|---|---|
| Source control history | [`code-archaeology.md`](./sources/code-archaeology.md) | Commits, diffs, blame, PR bodies and review threads via `gh` |
| In-repo written records | [`in-repo-records.md`](./sources/in-repo-records.md) | ADRs, design docs, postmortems, runbooks, CHANGELOG, config as code, comment blocks |

Cross-cutting:

- [`incident-postmortem.md`](./sources/incident-postmortem.md). Give this to both investigators when the target code looks defensive (null checks, retry, timeout, rate limit, feature flag, egress guard, OOM handler).

## What this cannot reach

Say this plainly in every output rather than letting the reader assume coverage. Nothing here searches a ticket tracker, a wiki, team chat, dashboards, error tracking, or an analytics warehouse. No MCP servers, no API tokens, no network calls beyond `gh`. If a decision was argued in a chat thread and never written into a commit or a doc, this skill will not find it, and no amount of searching git will change that.

What survives is what someone committed. That biases the record toward decisions made during code review and away from decisions made before anyone opened an editor. Weigh findings accordingly, and when the trail goes cold, name the likely place it went instead.
