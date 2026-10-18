# yagni-debate

Agent skill that decides what, if anything, a GitHub issue should change, by
debating it with codex instead of accepting either side's first answer.

1. **Read the issue**: body, comments, labels and linked PRs; separate the
   problem from the proposed solution, and check it still applies.
2. **Own position first**: read the code, reproduce the problem, check the
   premise, weigh the cost of building against the cost of not building.
3. **Independent verification**: `codex exec --yolo` checks the issue against
   the current code and reproduces it in a scratch directory.
4. **Debate rounds**: each side rebuts with evidence (file:line, a realistic
   scenario, a reproduction) or concedes. Every round also asks for a smaller
   change, ideally zero added lines or reuse of existing code. At most three
   rounds.
5. **Verdict**: implement, narrower scope, zero-line alternative, close, or
   keep open, with line counts, a minimal plan or tested patch, and a draft
   comment for the issue.

It covers the debate only: implementing, opening a PR, and commenting on or
closing the issue are left to the caller.

## Install

```bash
git clone https://github.com/rmens/yagni-debate ~/.claude/skills/yagni-debate
```

## Use

```
/yagni-debate https://github.com/owner/repo/issues/123
/yagni-debate #123
/yagni-debate owner/repo#123 ../repo net zero lines
```

Requires `gh` (authenticated) and `codex`.
