---
name: recall
description: "Use for 'recall my work on X', 'catch me up', 'what have I been working on', 'where did I leave off'. Rebuilds your recent working context from past chats, live git and gh state, and the shared record (user reports, prior fixes, incidents), then returns a tight current-state brief."
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Agent
---

# Recall

Rebuild the user's recent working context and hand back a tight brief of where
things stand now and what to do next. Keep it on topic. Read only what the
in-scope threads need, then stop.

The context lives in two records. Your own chat history holds what you did and
decided. The shared record holds what happened around the same code under other
names. That means the symptoms users keep reporting, the fixes that shipped and
got reverted, and the errors still firing in prod. The `why` skill searches that
second record. A feature with a long bug tail keeps most of its story there, so
don't rebuild it from transcripts alone.

Transcripts live at `~/.claude/projects/<slug>/<session-id>.jsonl`. The
`<slug>` is the absolute workspace path with every "/" turned into "-", leading
dash kept, so `/Users/you/proj` becomes `-Users-you-proj`. Each line is one
JSON record, and not every record is a chat message. Chat text sits under
`message.content` in records whose `type` is `user` or `assistant`. Other types
such as `mode` or `file-history-snapshot` are noise. Each session also has a
`<session-id>/` directory beside its file.

## Steps

1. **Classify, then route.** If the user wants one specific prior chat back,
   that is a different task. Tell them to resume it with `claude --resume` or
   `/resume`. A human-readable summary of their work is also a different task.
   Recall loads context across recent chats before you act. If the user already
   gave you a full state capsule (paths, branch, the change), use it and skip
   the mining.
2. **Lock the scope before searching.** Pin the window. "Recent" is a real
   range, so default to the last 7 days. Pin the topic if one is named. Pin the
   workspace, defaulting to the active one. Never read another project's
   transcripts unless asked. State the scope back. Never quietly turn "all" into
   "recent N".
3. **Fan out over chat history.** Spawn parallel `Explore` agents, each taking a
   slice of the transcripts. Use `model: "haiku"` or `"sonnet"` as the cheap,
   fast choice, and set effort per the Subagents section of
   `../minion/SKILL.md`. Give every agent these instructions.
   - Order candidates by real modification time (`ls -t`), never by file name.
   - Grep the topic first. Read only the matching chats and only their relevant
     regions.
   - Skip the current chat and noise chats (subagent, eval and test chats).
   - Read only `user` and `assistant` records. Skip records with
     `isSidechain: true` and skip the per-session subdirectories, unless the
     answer hinges on what a subagent did.

   Each agent returns one block per chat with these fields, citing the chat
   session id: topic, the user's goal, decisions, open threads, struggles and
   corrections, and artifacts (PRs, tickets, branches). For one or two chats,
   skip the fan-out and search directly. The raw transcripts stay in the
   subagents. The main thread gets only their findings.
4. **Sweep the shared record.** Do this whenever the topic names a feature,
   file, subsystem, area or bug. It is the default, and "my work on X" does not
   exempt it. Hand it to the source investigators in `../why/SKILL.md`, using
   `../why/references/source-playbook.md` and its per-source guides. Steer their
   question away from "why was this built this way". Ask what the current state
   is, what was tried and did not hold, and what users are still reporting. Run
   them in parallel with the chat mining. Keep why's posture. One investigator
   owns one source, null results are findings, and an unavailable MCP is skipped
   with a note. Fold what comes back into the brief. Skip this step only for
   pure activity recall with no named target ("what did I do this week"). There
   your history and live state are the whole answer.
5. **Verify against live state.** Take the PRs, branches and tickets that the
   mining and the sweep surfaced and check them with `git` and `gh`. If the
   answer hinges on what an agent actually did (tools run, files read, errors
   hit), read that session's full transcript, subdirectory included.
6. **Write the brief** to the contract below. Group by thread. Stay on the
   named topic.

## Output contract

Lead with the capsule, then thread status, then problems, then the next move.
Deeper detail goes below or gets cut.

- **Capsule.** At most 5 bullets. What this work is and where it stands overall.
- **Threads.** One line each, prefixed with exactly one status tag:
  `[merged #N]`, `[open PR #N]`, `[in flight <branch>]`,
  `[verified, uncommitted]`, `[reverted #N]` or `[planned, not started]`. A
  thread with no tag is not done yet, so tag it.
- **Problems.** At most 5, the recurring ones. Include the symptoms users keep
  reporting and any fix that shipped and was reverted, so the next attempt
  starts where the last one failed.
- **Next move.** The single most useful next action, concrete.

An adjacent feature or ticket stays out unless it blocks this one. When the
capsule and thread lines outgrow a screen, cut detail before you cut threads.
Write the brief through the `../unslop/SKILL.md` skill. Cite chat findings by
session id and shared-record findings by their source (PR number, ticket ID,
chat permalink, error-tracker issue). Sanitize private context before any
public output.

**Reply:** the brief, to the contract above.
