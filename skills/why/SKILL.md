---
name: why
description: "Use for 'why does X work this way', 'why we picked Y', design rationale, regressions, or where a threshold came from. Searches the two records that live on this machine, git history (commits, diffs, blame, PRs and review threads via gh) and written records committed to the repo (ADRs, design docs, postmortems, runbooks, CHANGELOG, config as code), then returns a cited, confidence-calibrated read on the decision. Needs no credentials or external services, and says where the trail leaves the repo. Use how for runtime behavior."
metadata:
  source: pstack
---

# Why

Investigate the motivation and intent behind code. Why was it built this way? What edge cases were considered? What product, business, or operational constraints shaped the design? What alternatives were rejected, and why?

Companion to the `how` skill. `how` answers what the code does and how it works. `why` answers what forces led to its shape.

## How this skill works

Two records survive on this machine, and they hold different things:

1. **Source control history.** Commits, diffs, blame, and, when `gh` is authenticated, PR bodies and review threads. Implementation-time reasoning lives here, captured while people argued about a diff.
2. **In-repo written records.** ADRs, design docs, postmortems, runbooks, CHANGELOG, config as code, long comment blocks. Reasoning gets written out at length here, sometimes before the code and sometimes long after.

One investigator goes into each, in parallel, then a synthesizer weighs both sets of raw findings and calibrates confidence. Splitting gathering from judging is the mechanism that stops a tidy story from forming early and recruiting evidence to fit.

### What this skill cannot see

No ticket tracker, wiki, chat, dashboard, error tracker, or analytics warehouse. Nothing outside the repository, and no credentials are involved anywhere. Say that up front, because it biases the record in a specific direction: you will see decisions that reached code review and miss decisions made before anyone opened an editor. A choice settled in a meeting and implemented without comment leaves no trace here.

Two habits follow. Report null results as findings, since "no commit, PR, or doc explains this threshold" tells the user something real. And collect every pointer that leaves the repo, the ticket IDs in commit messages, the doc URLs in PR bodies, and hand them back as the next place to look. That list is often the most useful part of the output.

## Operating Posture

Operate as a careful, cautious, precise investigator. Think like a detective piecing together a historical case from fragmentary records. When the record is thin, say so.

Concretely:

- **Evidence before narrative.** Collect the pieces first, then see what story they support. Never pick a story and recruit the evidence that fits it.
- **Precision over polish.** Prefer the exact quote and citation over a smooth paraphrase. A reader should be able to follow any claim back to its source and verify it in under a minute.
- **Consider what you haven't seen.** The evidence you find is a sample, not the whole truth. Before concluding, ask what you would expect to see if an alternative explanation were true, and whether you looked for it.
- **Name the gaps.** If a thread goes cold, a source isn't searchable, or a question has no answer, document the gap. Don't paper it over with an authoritative-sounding guess.
- **Hedge on purpose.** When evidence is indirect, your language should signal it ("appears to", "likely", "suggests"). Confidence-matching phrasing is a feature of the output, not a stylistic choice the synthesizer may override.
- **No shortcut by code-reading.** The code tells you what it does, rarely why it exists. Resist inferring intent from code shape.

This posture is the working method, not a disclaimer.

## Core Epistemics

This skill builds a **patchwork understanding** from fragmented historical evidence. Commit messages lie. Design docs get backfilled months later and read as though they came first. People change their minds between the PR description and the implementation. The original author may have left the company. And the largest fragment is missing outright, since anything argued outside the repository never reached you.

Be ruthlessly honest about what you know versus what you're inferring. The goal is not a satisfying story; it is to surface evidence, calibrate confidence, and let the user decide.

Principles:

- **Cite everything.** Every claim about intent should reference a specific commit hash, PR number, doc path with a line range, or code comment. A ticket ID is citable only as "referenced in commit abc1234", never as though the ticket itself was read. If you can't cite it, it's inference, not fact, and must be labeled as such.
- **Prefer "appears to" over "because".** Hedge when evidence is indirect. Reserve confident language for direct, explicit evidence.
- **Surface contradictions.** If two sources disagree, show both. Don't quietly pick the one that fits your narrative.
- **Acknowledge gaps.** If a question has no answer in any source you searched, say so. An honest "we couldn't find out why" beats a confident guess.
- **Multiple hypotheses are valid.** When the evidence fits several stories, present them all with the evidence for each. Let the user triangulate.
- **Beware rationalization.** Code that makes sense today may have been written for reasons that no longer apply, or for no good reason at all. Don't retrofit intent.

Read `references/epistemics.md` for the full confidence framework and phrasing guide. The synthesizer must follow it.

## Step 1. Understand the Target and the Question

Parse what the user is asking. The **target** is usually a chunk of code, a pattern, a feature, or a named design decision. The **question** is usually one of:

- "Why was X designed this way?" Design rationale.
- "Why do we do X instead of Y?" Tradeoff or alternatives.
- "What edge cases motivated this?" Defensive reasoning.
- "What business or product constraint led to this?" External forcing function.
- "Why does this code still exist?" Dead-code territory.
- "What's the history of X?" Broad archaeological sweep.

If the target is vague ("why do we do it this way?" with no clear referent), make your best guess from conversation context (open files, recent edits, cursor location, what was just discussed). State your interpretation briefly so the user can redirect if you're off, then proceed.

## Step 2. Establish the Code Anchor

Before spawning investigators, anchor the investigation in concrete code. You need:

- The relevant file path(s) and line range(s)
- The key symbols (function names, class names, constants)
- An initial commit list. The last few commits touching the target.
- PR numbers from merge commits (pattern `(#1234)` in the subject line)

Build this inline. It's cheap, and every investigator needs it.

```bash
# Blame target lines for last-touch commits
git blame -L <start>,<end> <file>

# Full file history, with patches, through renames
git log --follow -p -- <file>

# Last N commits touching the file, PR numbers visible
git log --oneline -20 -- <file>

# Extract PR numbers from a commit message
git log -1 --format=%B <commit>
```

Pull PR bodies and discussion via `gh` for any substantive commits:

```bash
gh pr view <number> --json title,body,author,createdAt,mergedAt,labels,closingIssuesReferences,comments,reviews
```

Capture this as seed context (file paths, symbols, commits, PR numbers, linked ticket IDs). Pass it to the investigators so they don't rediscover it.

Run `gh auth status` here too. Without it, PR bodies and review threads are gone, which removes the richest part of source control, and both the source control investigator and the final output need to know that.

## Step 3. Spawn the investigators

Two investigators, launched in a single message so they run concurrently. Each owns one record and goes deep rather than wide.

Subagent config (each):

- `subagent_type`: `general-purpose`
- `model`: `sonnet`
- Read-only posture. Tell each one it must not commit, write files, or modify state. Reading historical objects out of the object store is fine; checking anything out is not.

Each investigator gets:

1. The base prompt from `references/investigator-prompt.md`
2. Its playbook: `references/sources/code-archaeology.md` or `references/sources/in-repo-records.md`
3. `references/sources/incident-postmortem.md` **if the target code looks defensive** (null checks, retry logic, timeout handling, rate limiting, feature flags, egress guards, OOM handlers). Both investigators get it, since incident traces show up in commit patterns and in committed postmortems
4. The code anchor from Step 2, and whether `gh` is authenticated
5. The user's original question

### The roster

1. **Source control investigator.** Git history, `gh` for PRs, code comments, tests. Best at _implementation-time rationale captured during review_: PR descriptions stating the problem, review threads debating alternatives, inline comments encoding a non-obvious constraint, test names that encode the motivating edge case, commit messages linking a ticket or an incident. The most trustworthy record available, because it attaches to the diff that actually shipped.

2. **In-repo records investigator.** ADRs, RFCs, design docs, postmortems, runbooks, CHANGELOG, directory READMEs, Terraform alert definitions, feature flag configs, dbt models, long comment blocks. Best at _rationale written out at length_: problem statements, "alternatives considered" sections, action items that map onto a diff, thresholds duplicated in config. Its distinguishing move is dating a document against the code. A spec committed three months after the target shipped is a description, not a motivation, and only git can tell you which came first.

The two overlap, and that's fine. A doc's own PR discussion belongs to the first investigator; the doc belongs to the second. Each records cross-references as leads rather than chasing them.

### When to do it inline instead

Two subagents is the right shape for a target with real history: several commits, more than one PR, or a question about a tradeoff. It's more ceremony than a small target needs.

Answer inline, without spawning, when the target is a single commit whose PR description already contains the complete answer, or when the whole file has three commits and you read them during Step 2. Say that you did so. Then still run the in-repo keyword sweep, because it costs one `rg` and it's where a surprising ADR turns up.

What you may never do is skip a search and report the result as though nothing exists. A null result is a finding; an unrun search is a blind spot, and the two must not look alike in the output.

## Step 4. Synthesize

Spawn one synthesizer subagent:

- `subagent_type`: `general-purpose`
- `model`: `fable`
- Its quality check reruns a git command or opens a cited file to spot-verify a claim, so it needs Bash. Read and verify only, never write.

The synthesizer gets:

1. Both investigators' findings, including null results
2. The code anchor from Step 2, and whether `gh` was authenticated
3. The user's original question
4. The epistemics framework from `references/epistemics.md`
5. The synthesizer prompt template from `references/synthesizer-prompt.md`
6. The consolidated list of pointers that leave the repository: ticket IDs, doc URLs, chat permalinks, dashboard links, and where each was referenced

Its job is the final output: a confidence-weighted, evidence-cited narrative with "what we know" and "what we're inferring" kept apart, an honest account of the gaps, and the standing note that only the repository was searched.

## Step 5. Present

Take the synthesizer's output and present it to the user. You may lightly edit for clarity or add context from the conversation, but **do not rewrite the confidence language**. The epistemic framing is the product. Dropping the hedges to sound more authoritative is the exact failure mode this skill exists to prevent.

## Output Format

The final output uses this structure. Adapt as needed, but keep the confidence separation intact.

**The Question**. Restate what the user asked, concisely.

**The Code in Question**. File paths, line ranges, and key symbols. One or two lines so the reader is anchored.

**What We Found (direct evidence)**. Claims with explicit citations: commit hash, PR number, doc path with line range, or code comment at file:line. Each bullet is something with textual evidence behind it. Quote or paraphrase the source.

**What We Can Reasonably Infer**. Claims well-supported by indirect evidence or converging signals, but not stated anywhere. Each bullet must show the inference chain: "Given A and B, it's likely that C." Use hedged language ("appears to", "likely", "suggests").

**Competing Hypotheses**. If the evidence fits multiple stories, list them. For each, give the hypothesis, the evidence for it, and the evidence against it. Don't force a winner when the record doesn't support one. (Skip this section if there's a clear answer.)

**What We Don't Know**. Explicit gaps, stated specifically. "We read all 14 commits on this file since 2023 and both PRs; none states why the limit is 100" beats "we don't know why." Close with the standing limit in one line: only the repository was searched, so rationale that lived in a ticket, a wiki, or a chat thread and never reached a commit or a committed doc is invisible from here.

**Sources Consulted**. What was actually searched, so the user can judge coverage and redirect.

Example:

- Source control (git/gh): `git log --follow backend/retry.ts` across 14 commits since 2023, blame on lines 40-88, PRs #49074 and #47812 with review threads. PR #49074 introduced exponential backoff and its description cites ENG-4421.
- In-repo records: swept `docs/` and the tree for "retry", "backoff", "ENG-4421", and the literal `5000`. Found `docs/adr/0019-upstream-retries.md`, first committed 2024-08-09, five days before PR #49074 merged. Its "Alternatives considered" section rejects a fixed delay. The same 5000ms appears in `infra/alerts.tf`, committed in the same PR.
- Not searched: ticket tracker, wiki, chat, dashboards, error tracking, analytics warehouse. This skill reaches only the repository.
- Pointers found but not openable: ENG-4421 (cited in PR #49074 and in the ADR), a Slack permalink in the review thread on PR #49074, a dashboard URL in `docs/runbooks/upstream.md`. Opening ENG-4421 would most likely settle whether a customer drove the deadline.

**Next Place to Look**. When the answer is incomplete, name the specific system and item, not a generic suggestion. "ENG-4421, cited in PR #49074" is actionable. "Check the ticket tracker" is not. Where authorship points at a person who would know, name them from the commit and review record.

After the Sources Consulted block, if the user's `why` question is a precursor to actually changing this code, convert the lineage findings into a Preserve / Change / Avoid / Risk constraint set suitable for planning the change.

## Common Failure Modes to Avoid

- **Confident storytelling**. A plausible narrative built from thin evidence. A bullet with no citation goes in "inferred" or "hypotheses," not "what we found."
- **Citing the code as evidence for its own intent**. "Handles the null case because it checks for null" is mechanics, not motivation. Motivation comes from a commit message, PR discussion, doc, or comment, or it's labeled as inference.
- **Letting silence imply coverage**. The output must say that only the repository was searched. A reader who assumes the ticket tracker was checked will trust the answer more than it deserves.
- **Speculating about what the unreachable sources hold**. "The ticket probably says this was a customer request" is not a finding, and calling it a hypothesis doesn't fix it. Name the pointer, say it couldn't be opened, stop.
- **Recency bias**. Assuming the most recent commit is authoritative. The current shape is usually the accretion of many earlier decisions. Trace back.
- **Trusting a doc's date**. An ADR that reads like it preceded the code may have been committed months after. Check with git before treating it as motivation.
- **Sycophantic agreement**. If the user suggests a reason ("I assume this is for performance?"), treat it as a hypothesis and check the evidence independently.
- **Skipping the gaps section**. An honest accounting of what you couldn't find out is part of the value.
- **Skipping a search by anticipation**. Deciding up front that "there's probably no design doc for this" without running the sweep. A null result is a data point; an unrun search is a blind spot, and they must not look alike in the output.

## Reference Files

- `references/epistemics.md`. Confidence tiers and phrasing guide. The synthesizer must follow it.
- `references/investigator-prompt.md`. Base prompt template for the investigator subagents.
- `references/source-playbook.md`. Index of the two playbooks, plus a statement of what this skill cannot reach.
- `references/sources/code-archaeology.md`. Git and `gh` playbook.
- `references/sources/in-repo-records.md`. Committed documents, config as code, and comment blocks.
- `references/sources/incident-postmortem.md`. Cross-cutting angle for defensive code. Give it to both investigators.
- `references/synthesizer-prompt.md`. Prompt template for the synthesizer, including the output format.
