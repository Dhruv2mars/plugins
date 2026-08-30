---
name: interrogate
description: "Use for \"interrogate\", \"adversarial review\", \"multi-model review\", \"challenge this\", \"stress test this code\", \"find blind spots\", or \"tear this apart\". Multiple reviewers challenge changes from independent lenses."
disable-model-invocation: true
---

# Interrogate

Spawn one reviewer per lens to adversarially review code changes. Every reviewer runs on the session model (GLM-5.3-Flash); the adversarial signal comes from differentiated lenses, not model diversity. Each lens gets the same intent, diff, and rubric, plus its own lens addendum that decides what it hunts for. Agreement across lenses is high-confidence signal; single-lens findings are worth reading but lower confidence. For extra pressure on maintainability-heavy diffs, route a harsher pass through the **thermo-nuclear-code-quality-review** skill instead of adding a fifth reviewer.

The deliverable is a synthesized verdict. Do NOT auto-apply changes.

## Step 1, Determine Scope

Identify what to review from context:

- If the user points at specific files or a diff, use that
- If on a feature branch, run `git diff main...HEAD` (or the appropriate base branch) for the full changeset
- If the user's message references recent work, gather the relevant files

Package the diff (or file contents) plus any surrounding context files the reviewers need to understand the code.

## Step 2, State the Intent

Before spawning reviewers, state the intent explicitly. What is this code trying to accomplish? Derive this from:

- The user's message
- Commit messages
- PR description if one exists
- The code itself

Write one clear paragraph. Reviewers challenge whether the work achieves the intent well, not whether the intent itself is correct. If you're unsure about the intent, ask the user before proceeding.

## Step 3, Spawn Reviewers

Launch all reviewers in a single message using the Agent tool. Four lenses by default:

| Reviewer | Lens |
|----------|------|
| Reviewer A | Correctness — logic errors, edge cases, race conditions, error paths, broken invariants |
| Reviewer B | Security — injection surfaces, auth gaps, unsafe deserialization, secret handling, trust boundaries |
| Reviewer C | Maintainability — abstraction quality, reader load, hidden state, test seams, dead weight |
| Reviewer D | Spec / verification — does the diff do what the intent says, and is behavior proven at runtime (tests, control skill, receipts), not just claimed |

For each reviewer:
- `subagent_type`: `"general-purpose"`
- `model`: omit — all reviewers inherit the session model (GLM-5.3-Flash)
- `readonly`: `true`
- Isolated context: no shared scratchpads, no cross-reviewer communication. The independence that model diversity used to provide now comes from isolated contexts and differentiated prompts; protect it.

Read `references/reviewer-prompt.md` and fill in the template with:
1. The stated intent
2. The diff or file contents
3. The review rubric from `references/rubric.md`
4. The code-quality lens from `references/code-quality-review.md`
5. The reviewer's lens addendum from the table above, as an opening paragraph: "Your lens for this review: <lens text>. Prioritize findings inside your lens; report out-of-lens findings only when severe."

Each reviewer produces structured findings as described in the prompt template.

## Step 4, Synthesize

As results come back, build a unified picture:

1. **Parse all findings** from the reviewers
2. **Identify consensus**. Findings raised by 2+ reviewers, from different lenses, are highest signal.
3. **Identify single-lens findings**. Still worth reading, but weight accordingly.
4. **Deduplicate**. Different lenses may describe the same issue differently. Merge these and note which reviewers raised it.
5. **Note disagreements**. If one lens flags something and another explicitly says the opposite, that's useful context for the verdict.

Because all reviewers share one model, correlated blind spots are likelier than in a multi-model panel. Weight findings that cite concrete evidence (a failing path, a repro, a spec line) above taste-based ones, and say in the verdict that the panel is single-model.

## Step 5, Lead Judgment

You are the lead reviewer, a pragmatic senior engineer, not a neutral aggregator.

Read `references/lead-judgment.md` for the full framework. Reviewers only see a slice of the codebase. You have the full context (the goal, the constraints, the timeline, which tradeoffs were already considered). Use that context aggressively.

Categorize every finding using these buckets:

- **Act on**. Real issues affecting correctness, security, or maintainability given the actual goals. These would block a real PR.
- **Consider**. Legitimate points, but you're not sure they outweigh the cost of addressing them right now. Worth the user's attention.
- **Noted**. Technically valid but not actionable. Context-dependent, premature optimization, or low-impact given the current stage.
- **Dismissed**. Wrong, nitpicky, or missing context. Brief explanation why.

For each finding, include:
- Which reviewer(s) raised it, and their lens
- The category (act on / consider / noted / dismissed)
- A one-line rationale for the categorization

## Output Format

Present the verdict in this structure:

### Intent
> [The stated intent paragraph from Step 2]

### Reviewers
- Reviewer [label]: [lens], [N findings] (one bullet per reviewer)

### Act On
[Findings that should be addressed. For each: description, which lenses raised it, why it matters.]

### Consider
[Findings worth thinking about. For each: description, which lenses raised it, tradeoff involved.]

### Noted
[Valid but low-priority. Brief list.]

### Dismissed
[Rejected findings with brief rationale. This shows the user what was filtered out and why, so they can override your judgment if they disagree.]

### Lens Divergence Map
[Where did lenses agree, where did they diverge, and what does the pattern of agreement/disagreement tell us? Note that this panel ran on a single model; correlated blind spots are possible.]
