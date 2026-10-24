---
name: planning
description: Use when planning the implementation of a task. This Skill provides guidelines on what the plan must contain.
---

Before planning, invoke the `/engineering-principles` Skill.

1. Read the request and repository instructions; inspect the branch, worktree,
   and relevant issue/project context. Branch before editing when required.
2. Read known sources directly. When broad discovery is needed, you may use subagents
   for exploration. Inspect sources that determine the design before planning.
3. Consider the amount of work. Sometimes a ticket that is assumed to be L may end up
   being XXL in reality. Is the ticket actually multiple independent tickets wearing
   a trench coat? Is it a single ticket, but should be broken down into chunks? The
   user has Skills to help with these levels of work: `develop-lite`, which is for
   very difficult tasks that should go through a few review rounds, and
   `develop-with-subagents`, which is for breaking down huge tickets into chunks. If the
   ticket is really multiple tickets, suggest a split. If it's a single ticket that should
   be done in chunks, suggest a breakdown of the task.
4. Write the implementation plan: user-visible and technical outcomes, seams
   and APIs, affected files/modules and reasons, ordered steps, invariants,
   exclusions, and tests/checks/UI verification. Resolve ambiguity before presenting
   the plan. Repair the plan yourself when evidence invalidates it.
5. Use `/larp-agent` to request a review from the `plan-reviewer` Role. If any P1s surface,
   request a re-review until there are no more P1s.
6. Before presenting the plan, verbally state what you are least confident about
   regarding the task at hand. If these are product questions, ask the user or the agent
   that requested the plan. Otherwise, continue researching and iterating on the plan.
   Present the plan when you have no remaining concerns.

Important: The plan must include a brief sketch of any API changes or additions. The plan should detail how the changes are consumed. Skip if the task only changes internals.