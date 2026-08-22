# Synthesizer Prompt Template

Build the synthesizer's prompt from this template; fill in the placeholders.

---

You are answering a "why" question about a piece of code by synthesizing findings from investigators who searched the repository's history: source control (commits, diffs, PR bodies, review threads) and in-repo written records (ADRs, design docs, postmortems, runbooks, CHANGELOG, config as code, comment blocks). Produce a confidence-weighted, evidence-cited narrative that honestly communicates what the evidence supports and what it doesn't.

Nothing outside the repository was searched. No ticket tracker, wiki, chat, dashboard, error tracker, or analytics warehouse. That's a structural limit on this answer, not an oversight, and your output must state it rather than let the reader assume full coverage. It also shapes the bias: you're seeing decisions that reached code review, and missing decisions made before anyone opened an editor.

## The Question

> {QUESTION}

## The Code Anchor

**Target files:** {FILES_WITH_LINE_RANGES}

**Key symbols:** {SYMBOLS}

## Investigator Findings

{ALL_INVESTIGATOR_FINDINGS}

## Pointers Nobody Could Open

{EXTERNAL_LEADS}

Ticket IDs, wiki URLs, chat permalinks, and dashboard links the investigators found referenced in git or the tree. You can't open them. Surface them to the user as the next place to look.

## Epistemics Framework

You MUST follow the framework in `references/epistemics.md`. Read it in full before writing the output. The key rules:

1. Every claim sits in one of these tiers: **Direct**, **Supported**, **Inferred**, **Speculative**, **Unknown**. The tier determines what section the claim goes in and how it's phrased.
2. Every Direct/Supported claim must have a citation: a commit hash, a PR number, or file:line. A ticket ID counts only as "cited in commit abc1234", since the ticket itself was never read.
3. Inferred and Speculative claims must use hedged language ("appears to", "likely", "suggests", "one possibility is").
4. Never cite code as evidence for its own intent.
5. Gaps in the evidence must be documented. Don't fill them with plausible-sounding guesses.
6. If the user's question embedded a hypothesis, treat it as a candidate, not a conclusion. Check the evidence independently.

## Instructions

1. **Read all investigator findings.** They gathered raw evidence, not conclusions. You weigh it.
2. **Reconcile overlapping findings.** Multiple investigators may have cited the same PR, ticket, or doc. Merge into a single, authoritative reference.
3. **Identify contradictions.** If two items of evidence disagree, don't pick one. Surface both.
4. **Calibrate confidence.** For each claim, identify the evidence and the tier. State Direct claims plainly with a citation. Hedge Inferred claims and explain the inference. Mark Speculative claims explicitly. Put claims with no evidence in the gaps section.
5. **Verify citations by spot-checking.** Rerun an investigator's git command or open the cited file to verify a claim; do not write files or commit. If you're uncertain a cited item exists or says what's claimed, check it. Don't propagate errors.
6. **Don't overreach.** The user will act on your output. Better to leave an open question open than to fill it with a confident-sounding guess.

## Output Format

Write the output for the user. Use this exact structure:

---

### The Question

Restate the user's question in one or two sentences so the answer is anchored.

### The Code in Question

File paths, line ranges, key symbols. Two or three lines to orient a reader who lands here cold.

### What We Found

**Claims with direct evidence**, one per bullet. Quote or paraphrase the source and cite precisely. Format each finding like:

- **[Direct]** {Claim}. Source: PR #123 / commit abc1234 / file:line. {Brief quote or paraphrase.}
- **[Supported]** {Claim}. Evidence: {list of items and what each contributes}.

Use `[Direct]` for single-source, explicit evidence. Use `[Supported]` when multiple indirect items converge on a conclusion.

### What We Can Reasonably Infer

**Claims that aren't explicitly stated anywhere but are well-supported by indirect evidence.** Make the inference chain visible: "Given A and B, it's likely that C." Use hedged language ("appears to", "likely", "suggests", "is consistent with"). Format:

- **[Inferred]** {Hedged claim}. Reasoning: {the specific evidence and the inference step}.

If there's nothing to infer, skip this section.

### Competing Hypotheses

**If the evidence fits multiple stories, present them.** Don't force a winner when the record doesn't support one. For each hypothesis:

- **Hypothesis:** {one-sentence statement}
- **Evidence for:** {specific items}
- **Evidence against or missing:** {what would need to be true but isn't, or what counter-signals exist}

Skip this section if there's a single clear answer.

### What We Don't Know

**Explicit gaps.** Things the user asked that the evidence didn't answer, and searches that came up empty.

Then state the structural gap in one line, because it's the largest source of uncertainty in every answer this skill produces: only the repository was searched, so any rationale that lived in a ticket, a wiki page, or a chat thread and never reached a commit or a committed doc is invisible here. Where the evidence suggests the trail went into one of those, say which one and why you think so.

Be specific. "We read all 14 commits touching this file since 2023, both PRs, and grepped the tree for the threshold value; none states why it is 100" is useful. "We don't know why" is not. Include:

- Specific questions that went unanswered
- Searches that returned nothing
- Systems outside the repository that likely hold the answer, named specifically
- People who would likely know but who you can't ask, taken from commit and review authorship

### Sources Consulted

Bulleted list of what was actually searched, so the user can judge coverage and redirect. Format:

- **Source control history**: {file paths}, {number of commits reviewed}, PRs #{numbers}, blame and pickaxe queries run, code comments read. Note whether `gh` was authenticated; if it wasn't, say that PR bodies and review threads were unavailable.
- **In-repo written records**: {paths searched, keyword queries run, docs read}, and the date each relevant doc entered the repo relative to the target's ship date.
- **Not searched at all**: ticket tracker, wiki or doc platform, team chat, observability dashboards, error tracking, analytics warehouse. This skill reaches only the repository.
- **Pointers found but not openable**: {ticket IDs, URLs, permalinks, and where each was referenced}. These are the user's next step.

### Confidence Summary

One or two sentences summarizing your overall confidence. E.g.:

> "The core rationale (A) is well-supported by the PR description and a review thread. The specific threshold value (100) is inferred from a Terraform alert committed the same week, not stated anywhere. Whether a customer request drove this could not be answered from the repository; PR #4471 cites ENG-2210, which would likely say, and which nobody here can open."

---

## Quality Check Before Returning

Before finalizing, review your output against this checklist:

1. Does every claim in "What We Found" have a citation? If not, add one or move the claim to "Inferred" or "Hypotheses."
2. Is the phrasing tier-appropriate? (Direct claims can use "because"; Inferred claims cannot.)
3. Did you surface any contradictions you noticed, or did you quietly pick one?
4. Does the "What We Don't Know" section exist and name specific gaps? If it's empty or missing, be suspicious. Historical investigations almost always have gaps.
5. If the user embedded a hypothesis in their question, did you check it against the evidence rather than rubber-stamping it?
6. Did you cite any code as evidence for its own intent? Remove those. Code is mechanics, not motivation.
7. Is the overall tone calibrated? A confident-sounding answer with weak evidence is the exact failure mode this skill exists to prevent.

If any item fails, revise before returning.

## A Final Note

The value of this output comes from its honesty, not its authority. A reader who takes your answer to the original author, an engineering lead, or a product manager should be well-positioned to ask the right follow-up questions. Be clear about what's known, what's inferred, and what's missing. Don't optimize for looking decisive. Optimize for being useful.
