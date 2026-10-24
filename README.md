# Filament Kanban Pro

> **Documentation only.** Filament Kanban Pro is a commercial plugin: this public repository holds its
> documentation, changelog and licence terms. The package itself is installed from the private Composer
> repository you get with a licence — [buy one on Anystack](https://checkout.anystack.sh/filament-kanban-pro).
> Questions: support@asign.in.ua.

[![Stand With Ukraine](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/badges/StandWithUkraine.svg)](https://stand-with-ukraine.pp.ua)

<img class="filament-hidden" src="https://raw.githubusercontent.com/asignua/filament-kanban-pro-docs/main/art/cover.jpg" alt="Filament Kanban Pro">


Kanban Pro turns the free [filament-kanban](https://github.com/asignua/filament-kanban) board into a pipeline tool:
**swimlanes**, **WIP limits**, **transition rules**, **column totals per currency**, **stale and overdue badges**,
**several boards as tabs**, **Mine / All scope toggles** and **Filament table filters** on the board. It builds on the
base package's documented extension points and never forks its views or its move pipeline, so the base package keeps
receiving fixes on its own.

## Free or Pro?

| | [Kanban](https://github.com/asignua/filament-kanban) (free, MIT) | [Kanban Pro](https://github.com/asignua/filament-kanban-pro-docs) (€69 / €169) |
| --- | :---: | :---: |
| Board from an enum or from database rows, any Eloquent model (int / UUID / ULID) | ✓ | ✓ |
| Drag and drop that keeps the places of cards hidden by search or filters | ✓ | ✓ |
| Ask before a move (modal fields), policy and `canMove()` authorization | ✓ | ✓ |
| Lazy columns: load more cards per column (opt-in) | ✓ | ✓ |
| Search box, foldable columns, edit modal, events, mokhosh-compatible API | ✓ | ✓ |
| Swimlanes (by owner, priority, any attribute) | | ✓ |
| WIP limits, soft or hard | | ✓ |
| Transition rules: allow-list, required fields on entering a column | | ✓ |
| Column totals per currency, stale and overdue badges | | ✓ |
| Several boards as tabs, Mine / All scope toggles | | ✓ |
| Any Filament table filter on the board | | ✓ |
| Workflow designer: steps, transitions and automations edited in the panel, journal, webhooks | | ✓ |

Kanban Pro is an add-on: it requires the free package and builds on its documented extension points, so a board
upgrades without rewriting.

## Screenshots

Swimlanes by owner, WIP limits (soft and hard), column totals per currency, stale and overdue badges, boards as tabs and a Mine / All toggle on one page:

![A pipeline board with owner swimlanes, WIP limit warnings, totals per currency and overdue badges](https://raw.githubusercontent.com/asignua/filament-kanban-pro-docs/main/art/board.jpg)

Several boards as tabs, each with its own scope and limits (here Support with a hard WIP limit on "In progress"):

![The Support tab of the pipeline board](https://raw.githubusercontent.com/asignua/filament-kanban-pro-docs/main/art/support.jpg)

Dark mode:

![The pipeline board in dark mode](https://raw.githubusercontent.com/asignua/filament-kanban-pro-docs/main/art/board-dark.jpg)

## Purchase

Filament Kanban Pro is a commercial plugin. Both tiers include one year of updates; after that the
plugin keeps working on the last release you received, and you can renew for further updates.

| Tier | Projects | Activations | Price | Renewal |
| --- | --- | --- | --- | --- |
| Single Project | 1 | up to 3 (production, staging, local) | €69 | €35 / year |
| Unlimited | any number, SaaS included | unlimited | €169 | €85 / year |

Refunds are available within 14 days of purchase.

**[Buy a licence on Anystack](https://checkout.anystack.sh/filament-kanban-pro)** — after the purchase you receive a licence key and
access to the private Composer repository. See [LICENSE.md](https://github.com/asignua/filament-kanban-pro-docs/blob/main/LICENSE.md) for the licence terms.

## Requirements

- PHP 8.3+
- Laravel 12 or 13
- Filament 5
- [`asignua/filament-kanban`](https://github.com/asignua/filament-kanban) ^1.0 (installed automatically)

## Installation

1. **Add the private repository** (the same for every customer):

   ```bash
   composer config repositories.filament-kanban-pro composer https://filament-kanban-pro.composer.sh
   ```

2. **Add your credentials.** The username is the e-mail address you bought the licence with; the
   password is your licence key. A **Single Project** licence is bound to a fingerprint, so the password
   is the key followed by a colon and the fingerprint you activated — usually the production domain:

   ```bash
   # Single Project
   composer config --auth http-basic.filament-kanban-pro.composer.sh you@example.com "LICENCE-KEY:shop.example.com"

   # Unlimited
   composer config --auth http-basic.filament-kanban-pro.composer.sh you@example.com "LICENCE-KEY"
   ```

   This writes `auth.json` next to your `composer.json` — keep it out of git (add `auth.json` to `.gitignore`).
   On CI and servers pass the same JSON through the `COMPOSER_AUTH` environment variable instead.
   A Single Project licence allows three activations (for example production, staging and local); manage
   them in your Anystack account.

3. **Require the package:**

   ```bash
   composer require asignua/filament-kanban-pro
   ```

4. **Register the plugin** on your panel (optional - the assets and translations load without it):

   ```php
   use Asignua\FilamentKanbanPro\KanbanProPlugin;

   $panel->plugin(KanbanProPlugin::make());
   ```

## Usage

Extend `KanbanProBoard` (or `KanbanProResourcePage` on a resource) instead of the base `KanbanBoard` and override the
hooks you need. All of them are `protected` and off by default. The base board's own properties and hooks
(`$model`, `$statusEnum`, `$recordSortAttribute`, `validateMove()`, `recordBadges()`...) work exactly as before.

```php
use Asignua\FilamentKanbanPro\Pages\KanbanProBoard;
use Asignua\FilamentKanbanPro\Support\{BoardDefinition, BoardScope, ColumnSummary, OverdueBadge, StaleBadge, Swimlanes, TransitionRules, WipLimit};
use Filament\Forms\Components\Textarea;
use Filament\Tables\Filters\SelectFilter;
use Illuminate\Database\Eloquent\Builder;

class DealsBoard extends KanbanProBoard
{
    protected static string $model = Deal::class;
    protected static string $statusEnum = DealStatus::class;
    protected static ?string $recordSortAttribute = 'position';

    protected function swimlanes(): Swimlanes
    {
        return Swimlanes::by('owner_id')->options(fn () => User::pluck('name', 'id')->all());
    }

    protected function wipLimits(): array
    {
        return ['doing' => 5, 'review' => WipLimit::hard(3)];
    }

    protected function transitionRules(): TransitionRules
    {
        return TransitionRules::make()
            ->allow('todo', ['doing'])
            ->allow('doing', ['todo', 'review'])
            ->allow('review', '*')
            ->ask('done', [Textarea::make('resolution')->required()]);
    }

    protected function columnSummary(): ColumnSummary
    {
        return ColumnSummary::sum('amount')->currency('currency');
    }

    protected function staleBadge(): StaleBadge
    {
        return StaleBadge::make()->sinceLatest('interactions', 'happened_at')->after(7)->exceptColumns(['done']);
    }

    protected function overdueBadge(): OverdueBadge
    {
        return OverdueBadge::relation('tasks', fn (Builder $q) => $q->whereNull('done_at')->where('due_at', '<', now()));
    }

    protected function boards(): array
    {
        return [
            BoardDefinition::make('sales')->label('Sales')->scope(fn (Builder $q) => $q->where('pipeline', 'sales')),
            BoardDefinition::make('support')->label('Support')->scope(fn (Builder $q) => $q->where('pipeline', 'support'))
                ->statuses(fn () => Stage::where('pipeline', 'support')->get()->map(fn ($s) => ['id' => (string) $s->id, 'title' => $s->name, 'value' => $s->id])->all())
                ->wipLimits(['3' => WipLimit::hard(2)]),
        ];
    }

    protected function scopes(): array
    {
        return [
            BoardScope::make('mine')->label('Mine')->icon('heroicon-o-user')->default()->query(fn (Builder $q) => $q->where('owner_id', auth()->id())),
            BoardScope::make('all')->label('All')->icon('heroicon-o-users')->visible(fn () => auth()->user()->isManager()),
        ];
    }

    protected function boardFilters(): array
    {
        return [SelectFilter::make('priority')->options(['low' => 'Low', 'high' => 'High'])];
    }
}
```

### Swimlanes

`Swimlanes::by('owner_id')` groups the visible cards by the attribute. Without `options()` the lanes are the distinct
values on the cards (title from `relation('owner')`, from a `HasLabel` enum, or from `titleUsing()`); with
`options([value => title])` the lanes are fixed and empty lanes stay drawn as drop targets (`showEmpty(false)` hides them).
Cards with an empty attribute go to an "Unassigned" lane (`unassigned('Nobody')` renames it, `unassigned(false)` removes it
and forbids dropping into it). A drop into a lane is validated: a lane that is not on the board is refused with a
notification and nothing is written. Columns are not collapsible while lanes are on.

### Lazy columns

The free package can load a column page by page (`protected ?int $recordsPerColumn = 25;`, or `recordsPerColumn(KanbanColumn)` for
some columns only). Kanban Pro follows it: the column totals are summed by a query over **all** cards of the column that match the
search and the table filters (not over the loaded page), WIP limits already count the whole column, and the table filters
narrow the pages and the counters alike. Two things to know:

- **Swimlanes switch lazy columns off.** A lane is cut from the loaded cards, so a board with `swimlanes()` loads every card
  (`recordsPerColumn()` returns null while lanes are on).
- The attribute of `ColumnSummary::sum()` (and its `currency()`) must be a real database column on a lazy column: it is read
  with a query, not from the models, so an accessor-only attribute is not summed there.

### WIP limits

`['doing' => 5]` is a soft limit: the header shows `count/limit` in warning colour at the limit and danger colour above it.
`WipLimit::hard(3)` also refuses a drop that would exceed it (and greys the column out while a card is dragged). The count
is the real number of cards in the column, so cards hidden by search or filters still count. A board of a tab brings its own
limits through `BoardDefinition::wipLimits()`.

### Transition rules

`allow(from, to)` builds an allow-list; once there is one, anything not listed is refused (`'*'` is "any column" on either
side; a column nobody can leave simply has no entry). `ask(to, fields)` demands fields when entering a column - optionally
only `from:` one source column, or a closure `fn (Model $record, string $from, string $to): array` - and
`askIfEmpty('doing', 'owner_id', Select::make('owner_id')...)` asks only while the card's attribute is empty. The answers are
saved with the move by the base `onTransitionConfirmed()` (fill the record), so name fields after attributes or override it.
Your own `validateMove()` still runs first.

### Totals and badges

`ColumnSummary::sum('amount')->currency('currency')->scale(2)` prints one line per currency in the column header; cards
without an amount are skipped, cards without a currency are summed under no currency. The totals cover the cards the board
shows. `StaleBadge` reads the last activity from an attribute (`since('last_activity_at')`) or from the latest related row
(`sinceLatest('interactions', 'happened_at')`, one `withMax` for the board), falling back to `created_at`. `OverdueBadge`
reads a count attribute (`OverdueBadge::attribute('overdue_tasks_count')`) or counts a relation with your constraint
(`OverdueBadge::relation(...)`, one `withCount`). Both take `inColumns([...])` / `exceptColumns([...])`, and a badged card
gets the classes `fi-kanban-card--stale` / `fi-kanban-card--overdue`.

### Boards, scopes, filters

With two or more visible boards a tab row appears; the picked board is remembered in the session, an unknown or no longer
visible key falls back to the first board. A board's `scope()` closure is part of the board's own query (it defines which
cards belong to it), so reordering and WIP counts respect it. Scope toggles narrow the visible cards only, and are
remembered too.

> **Scopes are view filters, not an authorization boundary.** They only narrow what is drawn; a crafted request can still
> move or edit a card hidden by the active scope if the policy allows it. Restrict access with a policy or `canMove()`, or
> put the restriction into `getEloquentQuery()` / `BoardDefinition::scope()`.

`boardFilters()` takes any Filament table filter (`SelectFilter`, `TernaryFilter`, `Filter::make()->schema()`,
`QueryBuilder`...), applied live and remembered in the session.

## Workflow designer (Pro)

An opt-in module: steps, allowed transitions, reasons, required fields and automations are designed in the panel (a
git-graph style diagram) instead of being hard-coded, and every change of a record's step goes through one engine that
checks the rules, writes a journal row and runs the automations.

**1. Enable and migrate.** Set `KANBAN_PRO_WORKFLOW=true` (config key `filament-kanban-pro.workflow.enabled`) and run
`php artisan migrate`. The tables are not loaded while the flag is off. Register `KanbanProPlugin::make()` on the panel to
get the "Workflows" resource.

**2. Add the step column** to the table of the model that moves through a workflow:

```bash
php artisan kanban-pro:workflow-column "App\Models\Deal"   # --column=workflow_step_id by default
php artisan migrate
```

It writes a nullable foreign key to the steps table with `restrictOnDelete()`. The command refuses a class that is not
an Eloquent model or does not use `HasWorkflow`.

**3. Use the trait, register the model** and decide who may design:

```php
use Asignua\FilamentKanbanPro\Workflow\Concerns\HasWorkflow;

class Deal extends Model
{
    use HasWorkflow;
}

Workflow::for(Deal::class);   // required: in a service provider's boot(); the designer offers only registered models

Gate::define('manageWorkflows', fn ($user) => $user->isAdmin());   // deny by default
// or: KanbanProPlugin::make()->authorizeWorkflowsUsing(fn (?Authenticatable $user) => ...)
// menu: KanbanProPlugin::make()->workflowNavigationGroup('Admin')->workflowNavigationLabel('Pipelines')->workflowNavigationSort(90)
```

**4. Move records through the engine.** The model guard throws `WorkflowGuardViolation` when the step column is written
directly:

```php
use Asignua\FilamentKanbanPro\Workflow\Workflow;

Workflow::engine()->move($deal, $step, ['comment' => '...'], by: $user);   // throws TransitionRefused
Workflow::engine()->assign($deal, $step);            // the first placement of a record that has no step yet
Workflow::engine()->refusalFor($deal, $step);        // ?string reason, null = allowed
Workflow::engine()->allowedTargets($deal);           // steps the record may move to
Workflow::engine()->requirements($from, $to);        // comment / reason / fields demanded by the target

Workflow::withoutGuard(fn () => Deal::query()->update([...]));   // seeders, imports, migrations
```

`HasWorkflow` also gives the record thin wrappers: `$deal->workflow()` (the process it sits in), `$deal->allowedSteps()`
and `$deal->moveTo($step, $data, $by)`.

**A record with no step: create it empty, then `assign()`.** Creating a record already in a step is not guarded (seeders,
imports), but writes no journal row; `assign()` is the documented entry that places a step-less record with the same
requirements, a journal row (from nothing) and on-enter actions as a move. It refuses a record that already has a step.

If a policy is registered for the model, a move needs an authenticated user allowed to `update` it. `withoutGuard()` only
lifts the model guard, it does **not** bypass that check. For the console, queue jobs and imports use the system mode:

```php
Workflow::asSystem(fn () => Workflow::engine()->move($deal, $step, [], $user));   // $user is optional
```

Inside the callback the policy and `canTransition` are skipped, but the allowed transitions, the requirements and hard
WIP limits still apply; the journal row is written with `is_system = true` and `user_id` = `$user` (null if none — the logged-in user is
not borrowed).

**A system placement is not a user decision.** When a record that has no step yet is placed in system mode
(`Workflow::asSystem(fn () => Workflow::engine()->assign($lead, $step))` — a website lead, an import), the entry
requirements of the step (comment / reason, also a required reason with no reason configured) and its hard WIP limit are
**not** applied: nobody is there to answer the modal, and refusing would lose the record itself (an owner tightening the
first step in the designer would silently break lead intake). A comment or reason passed in is still validated and
journalled. Soft limits are a board hint and never block. A system **move** of a record that already has a step keeps
every requirement and the hard limit.

**The guard covers Eloquent `updating` only.** A query-builder `update()`, `saveQuietly()` or `withoutEvents()` bypass it
(they fire no model event), so keep step writes on the engine.

**5. Show it.**

```php
// A board: one tab per workflow of the model, one column per step, edges, WIP limits and the requirements modal.
class DealsBoard extends KanbanProBoard
{
    use InteractsWithWorkflow;   // Asignua\FilamentKanbanPro\Workflow\Board\InteractsWithWorkflow

    protected static string $model = Deal::class;
    protected static string $recordStatusAttribute = 'workflow_step_id';
}

// A table / infolist
MoveToStepAction::make();        // row action
MoveToStepBulkAction::make();    // bulk action
WorkflowStepEntry::make();       // read-only badge with a "Change step" action
```

**6. Per-model hooks** (all optional):

```php
Workflow::for(Deal::class)
    ->canTransition(fn (Deal $deal, ?Step $from, Step $to, ?Authenticatable $by) => $deal->owner_id === $by?->id ?: 'Only the owner.')
    ->fields(fn (Deal $deal, Step $to) => [TextInput::make('amount')->required()])   // extra form fields in the modal
    ->rules(fn (Deal $deal, Step $to) => ['amount' => ['required', 'numeric']])
    ->afterTransition(fn (StepChange $change) => ...)                                // inside the move's transaction
    ->recipients(fn (Deal $deal) => [$deal->owner])                                   // the "responsible" of notification / mail
    ->title(fn (Deal $deal) => $deal->name)                                           // used in notifications and mail
    ->url(fn (Deal $deal) => DealResource::getUrl('view', ['record' => $deal]));
```

**Keep personal data out of the journal.** Every validated app field lands in the journal row (`StepChange::$data`)
for good. Fields that belong on the record only (tax numbers, addresses) go in `journalExcept()`: they are still
validated and handed to `afterTransition` through `$change->submitted()`, but the journal row never stores them.

```php
Workflow::for(Deal::class)
    ->journalExcept(['tax_number', 'registration_address'])
    ->afterTransition(fn (StepChange $change) => $change->submitted());   // the full set; $change->data has no excluded keys
```

`afterTransition` runs **inside** the move's transaction: an exception rolls the move back. Put side effects (mail, HTTP,
dispatching jobs) in `DB::afterCommit(fn () => ...)`, or use an automation, which runs on the queue after the commit.

`canTransition` returns `true`, or a string (the refusal reason); anything else refuses with a generic message.

**Host step fields.** Declare your own fields for the step slide-over (a SLA, a queue, a flag your code reads) with
`stepFields()`; they are stored in `steps.settings` (JSON) and read with `Step::setting()`:

```php
Workflow::for(Deal::class)->stepFields(fn () => [
    TextInput::make('sla_hours')->numeric(),
    Section::make('Routing')->schema([TextInput::make('queue')]),
]);

$step->setting('sla_hours', 24);   // key, default
```

Only keys that belong to the declared components are saved; anything else sent to `WorkflowDesigner::createStep()` /
`updateStep()` in `settings` is dropped.

**Journal name snapshots.** Every journal row (`StepChange`) also stores `from_step_name` and `to_step_name`, the step
labels at the time of the change (current locale, with the usual fallback). When a step is deleted its journal rows lose
`from_step_id` / `to_step_id`, but the names stay readable.

**Automations** are configured per step in the designer. Built in: `notification` (panel), `mail` and `webhook`. Write your
own by implementing `Actions\StepAction` (`key()`, `label()`, `configSchema()`, `handle(StepChange $change, array $config)`)
and registering it, for example in a service provider:

```php
Workflow::registerAction(SlackAction::class);
```

**The queue is required.** Automations run on the queue after the commit; without a running queue worker nothing is sent.
A failed action does not undo the move.

**Webhooks.** `https` only, no redirects followed, the host is resolved once and pinned, and private, loopback and CGNAT
addresses are refused. The body is signed: `X-Workflow-Signature` is `sha256=` plus the HMAC SHA-256 of the raw body, keyed with the secret set on the automation. Only a connection
error, 5xx, 408 and 429 are retried. Config (`filament-kanban-pro.workflow.webhooks`): `allow_http` and `allow_private`
(both `false`; for local development only), `timeout` (10 s), `tries` (3).

**Other config keys:** `table_prefix` (`kanban_`, set before the first migrate), `colors` (the palette offered for steps),
`locales` (languages the designer offers for names; empty = the current locale).

### Limitations

- The journal uses `morphs()`, so the primary keys of workflow models must be integers; ULID / UUID keys are not supported yet.
- `role` recipients for notification and mail are not implemented yet: pick users, the actor, or the record's responsible
  people (`recipients()`).
- Deleting a step relocates all its records first, including those hidden by global scopes (soft deletes, tenants), and
  journals each as a system change; the target must be a step of the **same kind** (the designer offers only those), and
  the kind of a step that holds records cannot be changed. Hard WIP limits are enforced by the engine for every writer; soft limits are a board
  hint only.

## Configuration

The boards need no config; everything is declared on the board class. The only config is the opt-in workflow designer
(`config/filament-kanban-pro.php`, see above). Translations can be published with
`php artisan vendor:publish --tag=filament-kanban-pro-translations`. Registering `KanbanProPlugin` on the panel is optional.

## Gotchas

- **The page must `implements HasTable`.** `KanbanProBoard` and `KanbanProResourcePage` already do; if you put the trait
  `InteractsWithKanbanPro` on your own page, add the interface, `protected string $view = 'filament-kanban-pro::kanban-board'`
  and `protected static string $toolbarView = 'filament-kanban-pro::kanban-toolbar'` to the class itself (PHP does not let a
  trait redeclare a property the parent class already has). The page then also carries Filament's table plumbing, with no
  columns and no records of its own.
- **Drop targets in lane mode carry the lane**: `data-status-id="column|lane"`. If you published or replaced the board views,
  keep that attribute, otherwise lane drops are read as plain column drops.
- **A hard WIP limit is checked just before the move, not locked.** Two people dropping the last free slot at the same moment
  can both succeed; the next render shows the column as over its limit.
- **Put the model's own scopes in `getEloquentQuery()`, UI filters in `modifyRecordsQuery()`** - the base package finds a whole
  column through the first when it saves positions. A `BoardDefinition::scope()` counts as the first kind.
- **Aggregates need the relation**: `sinceLatest('interactions', ...)` / `OverdueBadge::relation('tasks', ...)` call
  `withMax` / `withCount` on the board query, so the relation must exist on the model. A Lane title from `relation()` is
  eager loaded for you.
- **Lane ids are strings.** An integer owner id `5` is the lane `'5'`; `options()` keys keep their original type when written.
- The "forbidden column" painting and the early refusal of a drop are JavaScript conveniences. The server validates every
  move with `validateMove()` regardless.

## Translations

The interface ships in English, Ukrainian, German, Spanish, French, Italian, Dutch, Polish, Brazilian Portuguese and Turkish
under the `filament-kanban-pro::filament-kanban-pro` namespace. A test keeps every language in step with the English keys. Override a
string by publishing the translations (`--tag=filament-kanban-pro-translations`) and editing the copy in
`lang/vendor/filament-kanban-pro`.

## AI agents

The package ships [Laravel Boost](https://laravel.com/docs/boost) guidelines (`resources/boost/guidelines/core.blade.php`) so a
coding agent wires it up correctly.

## Testing

```bash
composer install
vendor/bin/phpunit
npm test            # the small drag-rule helpers in resources/dist/kanban-pro.js
vendor/bin/phpstan analyse --memory-limit=1G
vendor/bin/pint --test
```

`composer.json` has a development-only path repository for `../filament-kanban`; remove it before tagging a release
(see the CHANGELOG).

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## License

Filament Kanban Pro is commercial software. See [LICENSE.md](https://github.com/asignua/filament-kanban-pro-docs/blob/main/LICENSE.md).
