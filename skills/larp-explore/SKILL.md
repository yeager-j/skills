---
name: larp-explore
description: Explore a codebase with parallel LARP researcher Agents and saved Markdown reports. Use when the user requests larp explore, when independent repository questions need delegated discovery, or when continuing a saved researcher's findings. For a known file or symbol, read it directly; for reviews or automatic task splitting, use larp-agent.
---

# LARP exploration

`larp explore start` runs one `researcher` Agent per query. Each Agent gets its own saved conversation and uses the command's working directory. Researchers return Markdown; LARP saves the reports and an index. The caller reads the results and decides what they mean for the task.

**Inside a LARP Turn:** if `LARP_TURN` is set, use [the structured research protocol](references/inside-larp.md) before attempting any launch. Nested shell commands are refused even when the Role has `can-larp`.

## 1. Decide what needs discovery

- If you know the file, symbol or command that holds the answer, inspect it directly. A researcher round trip costs more than a local read.
- If the missing answer requires tracing an unknown path through the repository, make it a research query.
- Group independent queries into one invocation so they can run together. If question B needs A's answer, run A first and use its result to write B or send a follow-up.
- Use `larp-agent` for a review or second opinion, and its Swarm workflow when the task needs automatic splitting. Exploration expects the caller to supply the questions.

Finish this step with one bounded query per unknown. Avoid assigning the same discovery to yourself and a researcher at the same time.

## 2. Write self-contained queries

Each researcher sees the repository and its query, not this conversation or the other queries. Include the task context, the unknown, known scope and relevant constraints. Ask for locating evidence: paths, symbols, control or data flow, governing instructions, affected callers, tests and exact verification commands. Include only the categories the question needs.

Give the problem rather than a proposed implementation, so the researcher does not search only for evidence that supports your guess. Role instructions already cover research behavior; spend query space on the task.

For example, these questions can run independently:

```bash
larp explore start \
  --query 'For adding request cancellation, trace where requests start, how cancellation reaches workers, and who owns cleanup. Return paths, symbols and line references.' \
  --query 'For adding request cancellation, locate tests for interrupted or failed requests and the exact commands that run them. Identify missing coverage from repository evidence.' \
  --parallel 3
```

Run from the repository root. For multiline queries containing shell syntax, use a quoted heredoc; this preserves literal backticks and dollar signs:

```bash
larp explore start --query "$(cat <<'QUERY'
Trace how `SessionStore` is used and where `$sessionId` is validated. Name the governing source and affected callers.
QUERY
)"
```

Keep `--parallel` at its default of 3 unless the task needs a different limit; allowed values are 1 through 8. Each query creates a separate researcher even when its text duplicates another query.

Before the first launch, check `larp --help` for `explore start` and `larp agent roles` for `researcher`. If the command is missing, report that the installed CLI needs an update. If the Role is missing, tell the user to run `larp config`; do not silently choose a model or rewrite their Role file. Exploration saves read-only permissions and Markdown replies even if the Role specifies write access or a custom reply schema. Model, instructions and other Role settings come from the user's configuration.

## 3. Launch and await completion

Research takes minutes. Run `start`, `message` and `resume` with the Bash tool using `run_in_background: true`. Retain the returned task ID and output-file path. Continue independent work or end the turn; read the output file when the completion notification arrives. Use that notification rather than polling the process. The Bash output file contains command output, not the Markdown report bodies.

Record the exploration ID from `[larp] exploration <id>` and retain the process output location. Use the returned paths rather than guessing the newest session, since other explorations may run concurrently. Progress goes to stderr: live rows in normal terminals, plain lines and an idle heartbeat when redirected. stdout contains the final counts, report paths and researcher follow-up commands, not the report bodies.

## 4. Read and use the results

1. Read the full command summary, then `index.md`. Map each assigned question to its status, report path and researcher Agent ID. A failed or pending query is an unanswered question, not evidence that the code does not exist.
2. Read each successful report needed for the task. Start with the index and reports; use Agent logs only to inspect follow-ups, failures or missing context. This avoids loading raw execution output into the caller's context.
3. Open the cited source that controls your next design or implementation decision. Confirm that the evidence supports the claim and that the files have not changed. Trust the report as a map; do not repeat the same broad search unless the evidence is missing or conflicts.
4. Combine the findings into the current task. Name the sources that support decisions, distinguish observed facts from inference, and keep unresolved questions visible. LARP does not merge or verify the reports for you.
5. Ask the existing researcher to close a specific gap when it already has the relevant context. Start a new query for an unrelated unknown.

Research and resume read the current working tree; there is no repository snapshot. Do independent work while research runs, and disclose any intervening edits when consuming a report or asking a follow-up.

## 5. Continue a researcher

Use the **researcher Agent ID** from the index, not the exploration ID:

```text
larp explore message <agent-id> --message "Trace the cleanup path you found through its callers."
```

Launch it as in step 3. It resumes the saved conversation, so send the new question, changed files or correction rather than the whole original brief. `larp agent message` also works for these Agents.

The answer appears in command stdout and the Agent log. It does **not** update the original Markdown report or index. Preserve the follow-up evidence in your own task notes or final answer; keep generated reports unchanged because resume can overwrite them.

If the command says the message was queued, the process already delivering that researcher will print the answer. Read that process's output when it finishes; do not resend the same question. Use `larp agent show <agent-id>` if you need the saved conversation.

## 6. Recover incomplete work

Exit 0 means all initial reports were saved and exported. Exit 1 can mean invalid input, a failed query, an interruption or an infrastructure failure; inspect the error and saved status first.

```text
larp explore show <exploration-id>
larp explore resume <exploration-id>
larp explore list
```

`show` and `list` are short inspection commands. Launch `resume` as in step 3. It retries unfinished queries with their saved Agents, skips completed ones and rebuilds reports from committed replies without repeating successful research. Fix a reported setup problem before retrying; if it recurs, report it rather than starting a retry loop. A live process lock or surviving Harness must clear before another Turn can start; do not delete its lock to force a retry.

## Where information goes

| Location | Contents and use |
| --- | --- |
| `~/.config/larp/roles/researcher.md` | User-owned model, Harness and research instructions; new researchers snapshot these settings. |
| `~/.larp/exploration/<exploration-id>/index.md` | Queries, initial completion status, report links, Agent IDs and follow-up commands. Read this first. |
| Same directory: `001.md`, `002.md`, ... | Initial answers in query order, written as each researcher finishes. Follow-ups do not replace them. |
| Same directory: `exploration.json`, `messages.jsonl` | Saved session inputs, researcher links and initial-result records. Inspect with `explore show`. |
| `~/.larp/agents/<agent-id>/` | Agent metadata, initial and follow-up messages, saved Harness session references and raw Turn output under `turns/`. Inspect with `agent show` when needed. |

Reports are saved outside the repository. They do not enter this conversation automatically: read the files and bring the relevant, verified findings back to the user.
