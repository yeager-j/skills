---
name: larp-agent
description: Delegate a bounded task to a larp Role, a named agent that runs on Claude Code or Codex, and continue it later, run a larp review of a diff with the default reviewer Roles, or run a larp swarm that splits a large task into chunks for parallel Agents. Use when the user mentions larp or a larp Role, or asks for a review, research, or second opinion from another model, or for a sweep or audit across the whole repository.
---

# larp agent

A larp Role is a named agent definition: its harness, model, permission, and instructions live in the user's Role file, so you only choose the Role and write the brief. Each `larp agent` command blocks until the Agent's Turn ends, then prints its reply on stdout.

## 1. Choose a Role

Run `larp agent roles`. Each line is `name`, `harness:model`, `permission`, `description`. Pick the Role whose description fits the task. If none fits, tell the user which Roles exist and ask; Role files belong to the user.

A `write` Role edits the same working tree as you. While it runs, keep your own edits away from the files it may touch.

## 2. Write the brief

The Agent sees only the repository and your message, never this conversation. Write a self-contained brief:

- the goal, and what to return (for example, "findings ordered by severity" or "only findings about these fixes");
- the scope: base branch, untracked files, paths, commands to run;
- the spec: decisions, docs, or constraints the Agent should hold the work to.

The brief is done when someone with only the repository and the message could do the task. Run the command from the repository root, since the Agent works in your current directory.

Pass long or quoted text through a quoted heredoc, so the shell leaves backticks and `$` alone:

```bash
larp agent start --role plan-reviewer --message "$(cat <<'EOF'
Review the plan in /path/to/plan.md ...
EOF
)"
```

You do *not* need to include instructions on how to behave—this is covered by the role prompts that are automatically included alongside your message.

Good: "Review the plan in /path/to/plan.md"

Unnecessary: "Act as an independent, thoughtful reviewer with an eye for detail [...] Review the plan in /path/to/plan.md"

## 3. Launch in the background

Turns take minutes. Run the command with the Bash tool and `run_in_background: true`, then continue other work or end your turn. You are notified when it exits; read the output file then.

The first stderr line is `[larp] agent <id> (<role>) started`. Record the ID from the output file; follow-ups need it.

## 4. Use the reply

stdout holds the reply, then a footer with the Agent ID. Treat the reply as evidence from another model: check each claim against the code before you act on it or report it, and tell the user which findings held up.

Exit code 1 means the Turn failed. stderr gives the cause. The message stays waiting, and the next `larp agent message` delivers it again.

## 5. Continue the Agent

```bash
larp agent message <id> --message "<text>"
```

This resumes the same session, so send only what changed: new fixes, answers, or the next question. Launch it in the background, as in step 3.

If you used larp to review a plan, you can use this to request a re-review.

If a Turn is already running, the command prints `[larp] Queued for agent <id>` and exits at once. The process that is already running delivers the message after its current Turn and prints that reply in its own output. When one output holds several replies, each starts with `[larp] Reply to: "<message>"`.

`larp agent show <id>` prints the Agent's full log.

# larp review

Use `larp review` for code and style reviews of a diff. It sends one message to the default reviewer Roles from the user's larp config and saves one report per Role. Do not pass `--role`; the user owns the reviewer list. Use `larp agent` with the `plan-reviewer` Role for plan reviews.

## 1. Start the review

Run from the repository root. Name the scope in the message: the base branch or the commits, and any untracked files.

```bash
larp review --message "Review the diff of the current branch against main"
```

Launch it in the background, as in "Launch in the background" above. stderr shows the review ID and each reviewer's Agent ID.

## 2. Use the reports

stdout holds every report in order. Each one ends with its follow-up command. Check each finding against the code, as in "Use the reply" above.

## 3. Request a re-review

Only request re-reviews if a critical finding was found and fixed. Send only what changed to the same reviewer, and launch it in the background:

```bash
larp review message <agent-id> --message "<what changed>"
```

`larp review resume <review-id>` retries failed or unfinished reviewers. `larp review show <review-id>` prints the saved review.

# larp swarm

A swarm runs one Role over many chunks of a large task, in parallel, one fresh session per chunk. Use it for work that splits into independent parts, such as a style sweep or audit of the whole repository. For one bounded task, use `larp agent`.

## 1. Choose the execution Role

Pick the Role that will do each chunk, as in "Choose a Role" above. `init` always splits the task with the `swarm-planner` Role. If `larp swarm init` says that Role is missing, tell the user to run `larp config`.

Chunks run at the same time with no coordination of file edits. With a `write` Role, make sure the chunk paths do not overlap, and keep your own edits away from them.

## 2. Split the task

Run from the repository root, since chunk paths are relative to that directory:

```bash
larp swarm init --message "<task>" --role <execution-role>
```

`--role` only gives the splitter the execution criteria as context. The only stdout line is the absolute path to the editable `chunks.json`. Launch it in the background, as in "Launch in the background" above.

## 3. Check the chunks

Read `chunks.json` and fix it before you start. Each chunk has `id` (lowercase letters, digits, and hyphens; unique), `paths` (literal relative files or directories, no globs; `.` is the whole directory), and `focus` (what to do in that chunk). Paths set where findings belong; Participants may still read other files. Look for gaps and overlaps. Tell the user the chunk count, and ask before a very large swarm.

## 4. Start the swarm

Run from the same directory as `init`:

```bash
larp swarm start --chunks <path>/chunks.json --role <execution-role> [--parallel 1-8]
```

`--parallel` defaults to 3. The first stderr line is `[larp] swarm <id> · ...`; record the swarm ID. Chunks take minutes each, so launch it in the background and continue other work or end your turn.

stdout holds the final summary. Each chunk's report is at `~/.larp/swarms/<id>/results/<chunk-id>.md`, or in `--out <new-directory>` if you set it.

## 5. Use the reports

larp does not merge the reports. Read each one, check its claims against the code as in "Use the reply" above, and give the user one combined result.

## 6. Recover

Exit code 1 means validation, a chunk, or larp itself failed. A failed chunk does not stop the others. `larp swarm show <id>` prints inputs, outcomes, errors, and report paths as JSON. `larp swarm resume <id>` retries only the unfinished chunks; launch it the same way as `start`. Do not run `start` again on the same `chunks.json`; larp refuses it. For a new run, copy the file outside its swarm directory first.

`larp swarm list` lists all swarms with their progress.
