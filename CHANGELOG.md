# Changelog

All notable changes to `asignua/filament-kanban-pro` are documented here.

## 1.0.2 - 2026-10-12

- Licence terms (`LICENSE.md`): the commercial licence that comes with an Anystack licence key; prices in the README.
- Docs: `scripts/sync-docs.sh` points the screenshots at the public docs repository.

## 1.0.1 - 2026-10-12

- Fix: the WIP-limit badge showed its screen-reader sentence next to `4/3` (Tailwind's `sr-only` is not in the panel CSS) and truncated on narrow columns; the visible text is now just `4/3`, the full sentence stays the tooltip and accessible name.
- Docs: screenshots, cover and a workbench demo.

## 1.0.0 - 2026-10-09

- Fixed: `migrate` failed on MySQL / MariaDB (SQLSTATE 1059) — an auto-generated index name of the step journal was longer than 64 characters (SQLite does not check). All composite indexes now have short explicit names derived from the table prefix; a test checks every identifier against the MySQL limit with a long prefix.

Workflow designer (opt-in, `KANBAN_PRO_WORKFLOW=true`):

- Designer resource "Workflows" with a git-graph diagram: steps (open / won / lost kinds, colours, WIP limits), allowed
  transitions, reasons, required fields per target step, automations. Access through the `manageWorkflows` gate.
- One engine for every step change: `Workflow::engine()->move()` with refusals, requirements, a journal and the
  `HasWorkflow` model guard (`Workflow::withoutGuard()` for seeders and imports).
- Automations on the queue: panel notification, mail, webhook (https only, private hosts refused, HMAC signature,
  selective retries) and custom actions through `Workflow::registerAction()` and `StepAction`.
- Host step fields: `Workflow::for(Model::class)->stepFields([...])` renders your Filament components in the step form; values live in `steps.settings` and are read with `Step::setting($key, $default)`.
- `Workflow::asSystem()` journals only the user passed to `move()` / `assign()`; it no longer falls back to the logged-in user.
- `journalExcept([...keys])` on `Workflow::for(Model::class)`: app fields that are validated and handed to `afterTransition`
  (`StepChange::submitted()`) but never written to the journal's `data` — for personal data that belongs on the record only.
- Journal name snapshots: `from_step_name` / `to_step_name` on every `StepChange`, kept after a step is deleted.
- Per-model hooks: `Workflow::for(Model::class)` with `canTransition`, `fields`, `rules`, `afterTransition`, `recipients`,
  `title`, `url`.
- `MoveToStepAction`, `MoveToStepBulkAction`, `WorkflowStepEntry` and the board trait `InteractsWithWorkflow` (tabs,
  columns, transition map, WIP limits and the requirements modal come from the designed workflow).
- `KanbanProPlugin::workflowNavigationLabel()` / `workflowNavigationSort()` next to `workflowNavigationGroup()`: where the
  "Workflows" resource sits in the menu.
- `php artisan kanban-pro:workflow-column` writes the migration for the step column.
- A system placement (`asSystem` + a record with no step yet) skips the step's comment / reason requirements and its hard
  WIP limit: a website lead or an import is never lost to a requirement meant for people.
- The designer refuses to change the kind of a step that holds records, and relocates the records of a deleted step only
  into a step of the same kind (the select lists only those).

- `Workflow::asSystem()` for console and queue moves (skips the policy and `canTransition`, journals `is_system`),
  `WorkflowEngine::assign()` for the first placement of a step-less record, and `HasWorkflow::workflow()` / `allowedSteps()` /
  `moveTo()`.
- Hard WIP limits are enforced by the engine (not only the board); a required reason with no active reasons is refused
  before the modal and warned about in the designer; a trashed record is refused as a missing record.
- The designer edits a process (name, default, order; one default per model), counts and relocates records through global
  scopes, checks webhook URLs on save and explains how to register a model with `Workflow::for()`.
- Bulk move asks for the strongest arrow requirement over the selection; an unplaced record sees process-prefixed steps.

Decisions that differ from the design spec:

- The current step in forms is a read-only `WorkflowStepEntry` plus a "Change step" action, not an editable select that opens
  the requirements modal on save: same rules, no hidden dirty state in the edit form.
- The journal has an `is_system` flag: relocations of records when the designer deletes a step are recorded as system changes.
- Designer access is the `manageWorkflows` gate (or `KanbanProPlugin::authorizeWorkflowsUsing()`), denied by default.

Known limits: integer record keys only (the journal uses `morphs()`); no `role` recipients yet; the model guard covers
Eloquent `updating` only; `afterTransition` runs inside the move's transaction.

### Board

Manual test in a real app: the board is a HasTable page (for the filter form) that renders no table, and Filament leaves the action modals of a HasTable page to the table view, so the transition and edit modals never reached the browser (the drop looked like it did nothing). The board view now renders the action modals itself.

Review round 1: a refused or same-column lane drop no longer leaks its lane into the next move; overflow-safe decimal sums and formatting; a lane drop fetches the board once less; new `canChangeLane()` hook; plain-board JS no longer splits column ids; Htmlable lane labels; plural stale tooltip; README licence and scope notes.

- Workflow: "Move to step" row and bulk actions (comment/reason/own fields asked per target step) and a read-only `WorkflowStepEntry` with a "Change step" action. Decision vs the spec: a read-only entry plus an action instead of a form select that opens the requirements modal on save - same rules, no hidden dirty state in the edit form.
- Swimlanes by an attribute or relation; dropping into another lane writes the lane attribute.
- WIP limits per column, soft (warning colours) and hard (drop refused, forbidden column painted while dragging).
- Transition rules: allowed from -> to pairs with wildcards, fields demanded by the target column, forbidden targets painted.
- Column summaries: one sum per currency, bcmath with an integer fallback.
- Stale ("quiet N days") and overdue card badges.
- Several boards as tabs (own columns, query slice, limits and rules each) and declarative scope toggles, both remembered
  in the session.
- Filament table filters in a dropdown on the board.
- Translations: en, uk, de, es, fr, it, nl, pl, pt_BR, tr.

### Before publishing (development only)

`composer.json` carries a LOCAL path repository (`../filament-kanban`, version pinned to 1.0.0) so the add-on can be
developed before `asignua/filament-kanban` is on Packagist. **Remove the `repositories` entry before tagging**, and tag
the base package first (the add-on requires `asignua/filament-kanban: ^1.0`). The committed `composer.lock` is ignored by
git on purpose, so nothing else needs cleaning.
