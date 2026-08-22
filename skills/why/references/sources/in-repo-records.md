# In-repo written records

## What this source contains

Everything a team wrote down and committed. No credential, no network, no API. It is the only long-form rationale you can reach, and on most repos it is thinner than people expect, so read what exists carefully.

- ADRs and RFCs, usually under `docs/adr/`, `docs/rfc/`, `rfcs/`, or `architecture/`
- Design docs and specs checked into the tree
- Postmortems and incident writeups, often in `docs/incidents/` or `postmortems/`
- Runbooks, which frequently explain defensive code better than the code does
- `CHANGELOG.md` and release notes, which carry the user-visible framing of a change
- Directory-level READMEs, where a module's constraints often live
- Issue and PR templates under `.github/`, which reveal what the team asks people to record
- Config as code: Terraform monitor definitions, alert rules, feature flag configs, dbt models. These hold thresholds, and a threshold in config matching a constant in code is a real finding
- Deprecation notes, TODOs, FIXMEs, and long comment blocks near the target

The distinguishing property of this source is that it is version controlled. A doc's own git history tells you when the rationale was written relative to the code, a question you cannot ask of a hosted wiki page.

## How to reach it

Nothing to authenticate. Work from the repo root:

```bash
git rev-parse --show-toplevel
```

## How to search it

1. **Inventory before searching.** Know what kinds of documents this repo keeps, because their absence is itself a finding.

   ```bash
   ls -d docs docs/adr docs/rfc rfcs architecture postmortems .github 2>/dev/null
   find docs -name '*.md' -maxdepth 3 2>/dev/null | head -40
   ```

2. **Keyword sweep across all prose in the tree.** Search the feature name, the key symbols, the threshold values, and any user-visible strings.

   ```bash
   rg -n -i -C3 'upload_retry|UploadRetry|backoff' --glob '*.md' --glob '*.txt' --glob '*.rst'
   rg -n -i 'alternatives considered|rejected|we decided|tradeoff|instead of' --glob '*.md'
   ```

   The second query is the high-yield one. Those phrases mark the paragraph where someone explained a choice, and they show up in design docs regardless of what the doc is named.

3. **Search the numbers.** If the target has a magic constant, look for it everywhere, not only in prose. A threshold repeated in a Terraform alert or a flag config usually came from the same decision.

   ```bash
   rg -n '\b131072\b|128 \* 1024|"128KB"' --glob '!node_modules'
   rg -n 'monitor|alert|threshold' --glob '*.tf' --glob '*.yaml' --glob '*.yml'
   ```

4. **Date the document against the code.** This is the step that turns a doc into evidence. A spec written three months after the code shipped is a description, not a motivation.

   ```bash
   git log --diff-filter=A --format='%h %ad %an %s' --date=short -- docs/adr/0012-retry-policy.md
   git log --format='%h %ad %an %s' --date=short -- docs/adr/0012-retry-policy.md
   git show <hash> -- docs/adr/0012-retry-policy.md
   ```

   Compare the doc's first-commit date with the target's merge date and state which came first. The synthesizer needs that ordering to weigh the claim.

5. **Read deleted history.** Docs get removed. A rationale deleted two years ago is still in the object store, and it explains code that outlived it.

   ```bash
   git log --diff-filter=D --format='%h %ad %s' --date=short -- 'docs/**'
   git show <hash>^:docs/adr/0007-old-approach.md
   ```

6. **Follow the commit that added the doc back to its PR.** The doc's own PR discussion is often where the disagreement happened. Hand that PR number to the source control investigator rather than opening it yourself.

7. **Read comment blocks near the target.** A long comment is a written record even though it lives in a source file.

   ```bash
   rg -n -B2 -A8 '(TODO|FIXME|HACK|XXX|NOTE|WARNING)' <target_file>
   ```

## What good evidence looks like here

- An ADR whose "Context" or "Decision" section names the target's constraint, committed before the target shipped
- A postmortem in the tree with an action item that maps onto the target's diff
- A runbook step explaining what breaks when the target's guard is removed
- A CHANGELOG entry giving the user-facing reason for the change
- A Terraform alert threshold matching a constant in the target code
- A deleted design doc describing the approach the code still follows

## Common pitfalls

- **Docs written after the fact.** Plenty of ADRs are backfilled. Always check the doc's git date against the code's. If the doc came second, it records someone's later understanding, worth citing but not as motivation.
- **Aspirational specs.** A doc may describe a plan that changed during implementation. If the doc says X and the code does Y, that contradiction is the finding. Don't smooth it over.
- **Template boilerplate.** A "Motivation" heading filled with "improve user experience" is not evidence. Look for specificity.
- **Vendored and generated files.** `node_modules`, `vendor/`, and generated docs will drown a keyword sweep. Exclude them.
- **Absence read as decision.** No ADR doesn't mean the team decided informally. It may mean the team keeps ADRs somewhere this repo cannot see. Report the absence as an absence.
- **Treating a comment as a citation for intent.** A comment saying "retry three times" is mechanics. A comment saying "three retries because the upstream drops the first connection after a cold start" is motivation. Only the second is evidence.

## What to return

For each relevant document or comment block:
- Path and the specific section or line range
- The rationale text, quoted verbatim
- When it entered the repo (first commit hash and date) and who wrote it
- How that date sits relative to the target's ship date, stated explicitly
- Whether it is current, superseded, or deleted
- Any PR numbers or ticket IDs it references, handed off as leads rather than chased
