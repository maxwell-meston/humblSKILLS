---
title: "Sync Handoff Tasks For The Next Pickup"
context: handoff
category: tasks
concept: task-source-sync
description: "Create and complete relevant tasks using the same private configuration that pickup reads."
tags: tasks, pickup, configuration, continuity
sources:
  - "references/raw/Task sync request.txt"
  - "references/raw/Actions not logs request.txt"
last_ingested: 2026-11-10
---

# Sync Handoff Tasks For The Next Pickup

Handoff writes agreed task changes; pickup reads the current list.
Both use smart-pickup's external personal profile and the same starting-folder scope.
An invoked handoff includes this sync when configured, unless the user requests a read-only handoff or no task changes.
This is agent-mediated work through the current client's connected tool, not background synchronization.

**The task list holds actions; the handoff holds the log.**
A task is one next action a human owns, with a clear done-when.
Progress, evidence, version numbers, run IDs and what was opened or closed belong in the handoff, which a task can link by path.
Most Next Steps stay in the handoff only: the default for a handoff is few or no new tasks.
A list that mirrors every handoff's Next Steps grows into a session log nobody can act on.

1. Resolve the installed smart-pickup directory from the client skill catalog or preflight inventory.
   Read its personal-task-source concept and run its `scripts/pickup.py context --cwd <session-start-folder>` by absolute path.
   Honor its config-path handling, including `XDG_CONFIG_HOME`; do not keep a second task-source configuration in smart-handoff.
   If the skill or profile is missing, save the handoff and report task sync unavailable with a configure suggestion.
   Explicit `todos: null` disables sync without falling back to another list.
2. Use the returned source, field mappings, closed states, and `todo_scope` for both reads and writes.
   If `todo_scope_unmapped` is true, report the missing mapping, offer smart-pickup's add-category flow once, and skip task writes until it resolves.
   Never replace a scoped lookup with an all-project lookup.
   Discover provider tools from the current catalog; connector names vary between clients.
3. Fetch the live schema before writing and inspect existing tasks in that scope, including closed tasks when checking duplicates or completion.
   For Notion, use its returned data-source ID and exact writable property names and status values.
   For a configured local list, preserve unrelated text and headings while adding or checking only matching scoped items.
   Missing write access or unsupported mappings leave sync unavailable; never create a replacement database automatically.
4. Add a task only for an agreed next action a human owns that is not already tracked.
   Start the title with a verb and make "done" obvious from it; if you cannot say what done looks like, it stays in the handoff.
   Do not add "watch", "monitor" or "confirm it works" tasks unless they name a concrete check and when to run it.
   Do not add a task per sub-step: put sub-steps as a checklist in the body of the matching parent task.
   A drafted message gets a task only if the human must send it themselves.
   Reuse a matching task by its link or clearly matching intent and scope; repeated handoffs must not create duplicates.
   If a matching row is already closed, do not reopen it merely because an older handoff repeats it.
   Keep titles short; put the handoff link in the task body and leave file/run evidence in the handoff.
   Set the resolved project value and a verified open state; leave unknown owners, priorities, and dates unset.
5. Complete only matching tasks this session actually finished, with verified evidence.
   For status-based sources, use the first configured `closed_statuses` entry only after verifying it is a completion state in the live schema.
   For a configured `done` checkbox, verify the property is a checkbox in the live schema, then check it.
   If neither is configured or the live schema does not match, report the missing mapping and leave the row unchanged.
   Assumed progress, proposed work, and partially completed tasks stay open.
   Keep a task's status note to one line of current state and who it waits on, and overwrite it rather than appending dated progress notes.
   On completion, that line becomes one clause of evidence plus the handoff link; the detailed evidence lives in the handoff.
   Preserve other task content, such as the body and checklist, when you change the status line.
   When a task's date or deadline has passed, ask the user whether it slipped or is dead; never close it on the date alone, because plans get pushed back.
6. Verify the resulting rows and record their links under the handoff's Done or Next Steps section, with a short task-sync result.
   After an uncertain write, inspect current state before retrying to avoid duplicates or repeated changes.
   Partial failure is reported as partial; pickup can still read the handoff and recheck live task state.
   Keep source IDs, task rows, and personal paths out of the shareable skill brain.

**Incorrect:** every handoff creates another copy of its next steps, adds a "watch the release" task, appends a dated progress paragraph to a task's notes, or closes tasks described only as assumed.
**Correct:** the handoff completes a verified task with a one-clause evidence line and links one new open action; its other next steps stay in the handoff; pickup reads that same scoped source and shows the open action.
