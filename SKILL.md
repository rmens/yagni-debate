---
name: yagni-debate
description: "Decide whether one proposed change earns its lines by debating it with codex --yolo: take your own position first, let codex verify and reproduce, then argue in rounds where each side must rebut with evidence or concede, until the smallest justified outcome is clear (change, smaller change, zero-line alternative, or no change). Use for a single review finding (CodeRabbit inline or outside-diff, an agent, codex itself) or a design choice, when the user wants YAGNI/KISS/DRY challenged, prefers net zero added lines, or says not to just accept codex's verdict."
user-invocable: true
argument-hint: "<finding text, link, or decision> [repo path] [line budget]"
---

# YAGNI debate

One finding or decision in, one justified verdict out. Codex is a sparring
partner, not an authority: neither side wins by repeating itself, only by
evidence.

Scope: the debate and its verdict. Applying a fix, pushing, or replying on
GitHub is up to the caller; this skill hands back the verdict, the minimal
patch if any, and a reply text.

## Inputs

- The finding (full text, or a link to read it from), and where it applies
  (repo path, branch, file:line).
- The owner's line budget. Default: net zero added lines unless the change is
  genuinely useful; tests count, readability counts more than one saved line.

## Step 1 — Your own position first

Before asking codex anything:

1. Read the code the finding touches and trace the real flow. A finding raised
   on an older commit (outside-diff comments often are) may no longer apply.
2. Write down, in a few lines:
   - **Claim**: what breaks, for whom.
   - **Trigger**: the realistic sequence that causes it, and how likely it is.
     Check the premise (for example *when* an error can actually occur).
   - **Cost of fixing**: production lines, test lines, new branches, new
     fixtures, new concepts.
   - **Cost of not fixing**: worst realistic outcome, and whether the user
     already gets a clear signal (an error, guidance, an alert).
   - **Position**: CHANGE, SMALLER CHANGE, ZERO-LINE ALTERNATIVE, or NO CHANGE.

Holding a position is what makes the debate useful. Without one you will
drift to whatever codex says.

## Step 2 — Round 1: independent verification

Run codex with the finding, but not your position, so its first answer is
independent. It must verify against the current code, reproduce where
possible in a scratch directory, and answer AGREE / DISAGREE / FOLLOW-UP
with the minimal fix, its line count, and the test that fails without it.
Template: [references/prompts.md](references/prompts.md#round-1).

## Step 3 — Debate rounds

Compare codex's answer with your position. Where you differ, send your
argument and ask codex to **rebut with evidence or concede**
([challenge template](references/prompts.md#challenge)). Push on all of these:

- **Necessity**: is the trigger realistic, or only constructible? Is the
  outcome already signalled to the user?
- **Premise**: do both sides agree on when the problem occurs? A rebutted
  premise does not automatically flip the conclusion.
- **Size**: is this the smallest fix? Ask explicitly for a zero-line
  alternative (moving a statement, reordering a check, reusing an existing
  helper) and for reuse of existing machinery instead of a new reader, type
  or hook.
- **Counterexamples**: accept them only when concrete (file:line, a realistic
  operator sequence, a reproduction). A pathological case that is really
  another bug already tracked elsewhere does not justify extra code here;
  name it as a known limit instead.

Rules:

- Change your own position when the evidence is better, and say so.
- Do not accept a concession that comes without reasons either.
- When you propose a smaller version, implement it in a scratch copy, run its
  test (including a mutation that removes the key line), and give it back to
  codex for the next round.
- At most three rounds. When a round brings no new evidence, decide yourself
  and record the remaining disagreement and why you overruled it.

## Step 4 — Verdict

Return:

```
Verdict:   CHANGE | SMALLER CHANGE | ZERO-LINE ALTERNATIVE | NO CHANGE | FOLLOW-UP
Lines:     production +a/-b, tests +c/-d
Why:       two or three sentences with the decisive evidence
Patch:     the minimal diff (if any), tested
Test:      name, and the mutation it catches
Open:      remaining disagreement with codex, if any, and why it was overruled
Reply:     a short reply text for the reviewer (English), stating what changed
           or why nothing did
```

## Guardrails

- Codex never edits the repository: `codex exec --yolo -C <repo>`, experiments
  only in a scratch directory, and verify afterwards with `git status --short`
  and `git diff` against a saved patch.
- Prompts go to a file and are fed on stdin; run codex in the background and
  read its final message from `-o`.
- Never present codex's verdict as yours; quote the evidence, not the label.
- A test that only passes because of how it is written proves nothing: every
  test you keep must fail with the fix reverted.

## Lessons this skill encodes

- An existing filter or helper often makes a new mechanism unnecessary
  (filtering before a read limit beat writing a second log reader).
- The best answer can be one neither side proposed at first (moving an
  increment into an existing loop: zero lines, no new test).
- "Rare" claims need checking: an error described as hardware failure also
  occurred on a permission problem, yet the existing guidance still sufficed.
- Readability beats a saved line: no clever constructs in tests to hit a
  line budget.
