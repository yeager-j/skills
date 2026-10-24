---
name: develop-lite
description: Plan a task, delegate the implementation to one subagent or larp Role, review it, and submit one PR. A lightweight version of /develop-with-subagents with no stack.
disable-model-invocation: true
---

# Develop Lite

You are the coordinator. You plan, brief, review, and submit. The implementor writes all code. Do not edit code yourself, because your context is for coordination. Send fixes back to the implementor instead.

The work goes into one branch and one PR. If the plan does not fit in one PR, stop and tell the user to use `/develop-with-subagents`.

## 1. Plan

1. Write the plan with the `/planning` skill.
2. Save the plan to a file outside the working tree: the Plan Mode plan file, or the session scratchpad directory. A plan file in the working tree gets committed by accident.
3. Add a `Status` section to the plan file. Record the branch, the state (`planned`, `in progress`, `reviewed`, `submitted`), the implementor's ID, the plan reviewer's larp Agent ID, and the PR number. Update it at each state change. After your context is summarized, read the plan file again before you do anything else.
4. Use `/larp-agent` to request a plan review from the `plan-reviewer` Role. Verify each finding against the repository, revise the plan, and request a re-review from the same Agent. Repeat until the review has no blocking findings.
5. If the session is in Plan Mode, present the plan and wait for approval. Otherwise, continue.

## 2. Implement

1. Run `git status` and make sure the working tree is clean. Then create the branch from the base branch.
2. Start the implementor with the brief below. Use the method that the user asked for:
   - **Subagent (default):** an `implementor` subagent on `opus`, unless the user names a different model. No worktree.
   - **larp:** the `/larp-agent` skill with the `implementor-codex` Role. Use `implementor-codex-extreme` only if the task is extraordinarily difficult.
3. If the implementor returns a decision, make it. If the decision changes an API or a seam, update the plan file and send the change to the plan reviewer before you answer the implementor.

### Implementor brief

The implementor sees only the repository, its agent definition, and your message. Include all of these:

- The path to the plan file, with an instruction to read the full plan.
- The facts from planning that the implementor needs.
- The focused verification commands and the command for the full test suite.
- "The branch is checked out. Do not change branches, push, or open PRs."
- "When the verification commands pass, commit. Then use `/larp-agent` to run `larp review --message "Review the diff <or what changed>"` on the diff against the base branch. Verify each finding, fix the valid ones, commit, and request a re-review. Repeat until the review has no P1 blocking findings. Skip the review if there are no code changes."
- "If you change code later for review feedback, request a re-review from the same reviewer Agents with `larp review message`."
- "Report the commits, the test results, each review finding you rejected and why, and each deviation from the plan."

## 3. Review

1. Run the verification commands and the full test suite yourself. Do not trust the report.
2. Read the full diff against the base branch. Check each acceptance criterion and look for defects.
3. Check each style finding that the implementor rejected. If a rejection is wrong, add the finding to your own findings.
4. Send your findings to the same implementor. For a subagent, continue it with `SendMessage`. For larp, continue the same Agent.
5. After the implementor commits the fixes, verify again. Repeat until the tests pass and you have no blocking findings.

## 4. Submit

1. Make sure `git status` is clean and the full test suite passes.
2. Push the branch and open a ready PR (not a draft) with `gh pr create`, because Codex reviews only ready PRs.
3. Watch the PR with `/watch-prs`.

## 5. Fix findings

1. Verify each Codex finding against the code. Reject the invalid ones with a reply on the inline comment.
2. Send the valid findings to the same implementor. If it cannot be resumed, start a new one with the plan file and the findings.
3. After the implementor commits, run the full test suite, then push.
4. Reply to and resolve each inline comment. Do not watch for a second review: Codex reviews a PR one time.
