---
name: develop-with-subagents
description: Plan a large ticket as a coordinator, split it into chunks, and have subagents implement each chunk as one layer of a PR stack.
disable-model-invocation: true
---

# Develop With Subagents

You are the coordinator. You plan, brief, verify, and submit. Implementors write all code. Never edit code yourself, because your context is for coordination and the implementor has the context of the chunk. Send feedback back to the implementor instead.

Implementors are `implementor` subagents. They run on `opus` unless the user names a different model. Never use worktrees unless the user requests them: the stack is in one working tree and each chunk builds on the chunk below it. For the same reason, only one implementor runs at a time.

The user may ask you to implement via the `/larp-agent` Skill. If so, use the `implementor-codex` Role by default. The `implementor-codex-extreme` Role is available if the task is extraordinarily difficult and requires frontier model intelligence.

## 1. Understand

1. Read the ticket, every comment, and the ticket's Linear project. Then read the architecture docs in the repository, and the spec if there is one.
2. Read each file that the ticket cites yourself. Do not delegate these.
3. Spawn `Explore` subagents for fact finding. Give each one a specific question. Continue until you can state the seams, APIs, and invariants that the task touches without a guess.

## 2. Plan

1. Write the plan with the `/planning` skill.
2. Split the plan into chunks. Each chunk:
   - is one stack layer and becomes one PR;
   - depends only on the chunks below it;
   - has its own acceptance criteria and verification commands;
   - leaves the full test suite passing, because each PR in the stack must be green by itself.

   For chunk boundaries and branch names, read `references/stack-design.md` in the `/gh-stack` skill.
3. Save the plan to a file outside the working tree: the Plan Mode plan file, or the session scratchpad directory. A plan file in the working tree gets committed by accident.
4. Add a `Status` section to the plan file. For each chunk, record the branch, the state (`pending`, `in progress`, `accepted`, `submitted`), the implementor's agent ID, and the PR number. Record the plan reviewer's larp Agent ID too. Update this section at each state change. After your context is summarized, read the plan file again before you do anything else.
5. Request a plan review from the larp `plan-reviewer` Role with `/larp-agent`. Verify each finding against the repository, revise the plan, and request a re-review from the same Agent. Repeat until the review has no blocking findings.
6. If the session is in Plan Mode, present the plan and wait for approval. Otherwise, continue.

## 3. Implement

Do this loop for each chunk, bottom to top:

1. Create the branch. Use `gh stack init <branch>` for the first chunk and `gh stack add <branch>` for each subsequent chunk. Run `git status` first and make sure the working tree is clean, because uncommitted changes carry over to the new branch.
2. Start the implementor with the brief below.
3. If the implementor returns a decision, make it. If the decision changes an API, a seam, or a chunk boundary, update the plan file and send the change to the plan reviewer before you answer the implementor.
4. When the implementor reports completion, verify the chunk:
   - run the chunk's verification commands and the full test suite yourself. Do not trust the report;
   - check each acceptance criterion against the code;
   - make sure the work is committed on the correct branch and `git status` is clean.

   This is an acceptance check, not a code review. The style review and the Codex review find code defects.
5. If a check fails, send the failure with its output to the same implementor and verify again. Mark the chunk `accepted` only when all checks pass.

### Implementor brief

The implementor sees only the repository, its agent definition, and your message. The agent definition already covers code style, open decisions, and the basic report, so do not repeat those. Include all of these:

- The path to the plan file, with an instruction to read the full plan.
- "You implement chunk N: <name>. The chunks below it are complete. The chunks above it come later: make the seams that they need, but do not implement them."
- The facts from exploration that apply to the chunk.
- The chunk's focused verification commands and the command for the full test suite.
- "The branch is checked out. Do not change branches, push, or open PRs."
- "When the verification commands pass, commit. Then use `/larp-agent` to run `larp review --message "<what changed>"` with the default reviewer Roles on the chunk's diff against its parent branch. Verify each finding, fix the valid ones, commit, and request a re-review. Repeat until the review has no blocking findings. Skip the review if the chunk has no code changes."
- "If you change code later for review feedback, request a re-review from the same reviewer Agents with `larp review message`."
- "Add to your report: the commits, each review finding you rejected and why, and each deviation from the plan."

## 4. Submit

Submit only after every chunk is `accepted`. Review feedback on a lower chunk that arrives while an upper chunk is in work causes conflicts in the shared working tree.

1. Run the full test suite at the top of the stack.
2. Run `gh stack submit --auto --open`. Codex reviews only ready PRs, not drafts.
3. Set each PR's title and body with `gh pr edit`. Put ticket-closing words (`Closes <ticket id>`) only in the top PR, because a stack merges bottom to top and a lower PR would close the ticket before the work is complete.
4. Watch all PRs in the stack with one `/watch-prs` command.

## 5. Review findings

1. Verify each Codex finding against the code. Reject the invalid ones with a reply on the inline comment.
2. Group the valid findings by the chunk that owns the code. Fix the lowest chunk first, because each fix rebases all the chunks above it.
3. For each chunk with valid findings:
   - check out its branch with `gh stack checkout <branch>`;
   - send the findings to that chunk's implementor. If the implementor cannot be resumed, start a new one with the plan file, the chunk, and the findings;
   - after the implementor commits, run `gh stack rebase --upstack`, then `gh stack top`, then the full test suite.
4. Run `gh stack push` after all fixes are in and the suite passes.
5. Reply to and resolve each inline comment. Do not watch for a second review: Codex reviews a PR one time.
