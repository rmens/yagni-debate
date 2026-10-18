# yagni-debate

Agent skill that decides whether one proposed change earns its lines, by
debating it with codex instead of accepting either side's first answer.

1. **Own position first**: read the code, check the premise, weigh the cost
   of fixing against the cost of not fixing.
2. **Independent verification**: `codex exec --yolo` checks the finding
   against the current code and reproduces it in a scratch directory.
3. **Debate rounds**: each side rebuts with evidence (file:line, a realistic
   scenario, a reproduction) or concedes. Every round also asks for a smaller
   fix, ideally zero added lines or reuse of existing code. At most three
   rounds.
4. **Verdict**: change, smaller change, zero-line alternative, no change or
   follow-up, with line counts, the tested patch, the mutation its test
   catches, and a reply text for the reviewer.

It covers the debate only: applying the fix, pushing and replying on GitHub
are left to the caller (for example
[razor-review](https://github.com/rmens/razor-review)).

## Install

```bash
git clone https://github.com/rmens/yagni-debate ~/.claude/skills/yagni-debate
```

## Use

```
/yagni-debate https://github.com/owner/repo/pull/123#discussion_r456
/yagni-debate "Apply runtime effects when the save is committed" ./repo
/yagni-debate <finding text> . net zero lines
```

Requires `codex`; `gh` only to read a finding from a link.
