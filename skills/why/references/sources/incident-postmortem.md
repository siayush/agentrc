# Incident & postmortem context

Not a separate source, a **cross-cutting angle**. Incidents motivate a lot of defensive code ("we added this check after the X outage"), so when the target looks defensive (null checks, retry logic, timeout handling, rate limiting, feature flags), hunt specifically for incident history in git and in the tree.

In git:

```bash
git log --all -i --oneline --grep='incident\|postmortem\|sev[- ]\?[0-9]\|outage\|hotfix\|mitigat'
git log --all -i --oneline --grep='revert'      # a revert followed by a re-apply is a fire-drill signature
git log -S'<the guard string>' --oneline -- <file>
```

A revert, then a re-apply with extra defenses a few days later, is the strongest incident signature available without an incident tracker. Pull both commits and their PRs.

In the tree:

```bash
rg -l -i 'postmortem|incident report|root cause|action item' --glob '*.md'
git log --diff-filter=A --format='%h %ad %s' --date=short -- 'docs/**' | rg -i 'incident|postmortem'
```

Postmortems committed to the repo usually carry an "Action Items" section that maps onto code changes. When an action item and a commit diff line up, that is close to direct evidence.

Also worth checking: hotfix branches and tags (`git tag --list '*hotfix*'`), release notes mentioning an urgent fix, and runbook edits committed in the same window as the target.

## The limit

Formal incident records live in tools this skill cannot reach. If the git and in-repo trail suggests an incident (a hotfix, a revert-and-reapply, a guard added on a weekend) but nothing explains it, say exactly that. "The commit pattern suggests an incident-driven fix, but no incident record is visible from the repo" is honest and useful. Inventing the incident is not.
