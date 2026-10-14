<p align="center">
    <img src="art/banner.png" alt="Filament Changelog" width="100%" class="filament-hidden">
</p>

<h1 align="center">Filament Changelog</h1>

<p align="center">
    Manage, read and generate <a href="https://keepachangelog.com/">Keep a Changelog</a> files inside Filament.
</p>

<p align="center">
    <img src="https://img.shields.io/badge/Filament-v5-FDAE4B?style=flat-square" alt="Filament v5">
    <img src="https://img.shields.io/badge/PHP-8.2+-777BB4?style=flat-square&logo=php&logoColor=white" alt="PHP 8.2+">
    <img src="https://img.shields.io/badge/Laravel-12-FF2D20?style=flat-square&logo=laravel&logoColor=white" alt="Laravel 12">
    <img src="https://img.shields.io/badge/tests-41%20passing-brightgreen?style=flat-square" alt="Tests">
    <img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="MIT License">
</p>

---

A **hybrid** changelog for Filament: entries live in your database (fully
editable via a Filament resource, with grouping, filters and badges) **and** can
be imported from / exported to a `CHANGELOG.md` file at any time — via the UI or
artisan. Point the reader at your own project's file, a remote GitHub URL, or the
database, and get a clean, read-only changelog page out of the box.

## Screenshots

#### Reader page — grouped by version, typed sections, date badges

<img src="art/screenshots/reader.png" alt="Changelog reader page" width="100%">

#### Management resource — colored type badges, grouping, filters

<img src="art/screenshots/manage.png" alt="Changelog management table" width="100%">

<table>
<tr>
<td width="50%"><img src="art/screenshots/edit-form.png" alt="Edit a changelog entry"><br><em>Create / edit an entry</em></td>
<td width="50%"><img src="art/screenshots/import-modal.png" alt="Import a CHANGELOG.md"><br><em>Import a <code>CHANGELOG.md</code> — upload or paste</em></td>
</tr>
</table>

## Features

- 📖 **Reader page** — a polished, read-only changelog grouped by version with
  date badges and typed sections (built entirely with the Filament schema API,
  no custom Blade)
- 🔎 **Search, version filter & infinite scroll** on the reader — find entries as
  you type, jump to a single version, and load version cards as you scroll
- 📝 Management resource to create/read/edit entries, grouped by version
- 🏷️ Typed changes (Added, Changed, Deprecated, Removed, Fixed, Security) with colored badges
- ⬆️ **Import** an existing `CHANGELOG.md` (upload, paste, or `changelog:import`) —
  also when it is written in Portuguese (`### Adicionado`, `## [Não lançado]`)
- ⬇️ **Export** the database back to a `CHANGELOG.md` (download, or `changelog:export`)
- 🌍 **Translations** — English, European & Brazilian Portuguese (extendable)
- 🔗 **Live file / remote source** — read your own `CHANGELOG.md` or a GitHub URL, re-parsed on every visit
- 🛡️ **Shield-ready** — auto-discovered permissions, no hard dependency
- 🗂️ Optional multi-project mode (one app, many changelogs)

## Installation

```bash
composer require anselmokossa/filament-changelog
php artisan vendor:publish --tag="changelog-migrations"
php artisan migrate
php artisan vendor:publish --tag="changelog-config"
```

Register the plugin on your panel:

```php
use Filament\Changelog\ChangelogPlugin;

public function panel(Panel $panel): Panel
{
    return $panel->plugin(ChangelogPlugin::make());
}
```

### Customizing the plugin

Every option is available as a fluent method (each accepts a value or a
`Closure`), mirroring other Filament plugins. **Anything you don't set falls back
to the translations / `config/changelog.php`**, so you only call what you need.
This is the full list:

```php
ChangelogPlugin::make()
    // Navigation
    ->navigationLabel('What\'s new')            // string|Closure|null
    ->navigationIcon('heroicon-o-sparkles')     // string|BackedEnum|Closure|null
    ->activeNavigationIcon('heroicon-s-sparkles')
    ->navigationGroup('Docs')                   // string|UnitEnum|Closure|null
    ->navigationSort(1)                         // int|Closure|null
    ->registerNavigation(true)                  // bool|Closure — show a nav item at all

    // Labels
    ->modelLabel('release note')                // string|Closure|null
    ->pluralModelLabel('release notes')         // string|Closure|null

    // Reader page
    ->perPage(8)                                // int|Closure|null — cards per infinite-scroll step
    ->searchable(true)                          // bool|Closure|null — show the search box
    ->filterableByVersion(true)                 // bool|Closure|null — show the version filter

    // Formatting
    ->dateFormat('d M Y')                       // string|Closure|null — release-date badge format
    ->changeTypes(['added', 'fixed'])           // array|Closure|null — types + render order

    // Source (see "Reader source & file path" below)
    ->source('database')                        // 'database' | 'file'
    ->file('CHANGELOG.md')                      // string|Closure|null — local path or remote URL
    ->fromFile('CHANGELOG.md')                  // shortcut: source('file') + file(...)
    ->fromUrl('https://…/CHANGELOG.md')         // shortcut for a remote file
    ->remoteCacheTtl(300)                       // int|Closure|null — cache seconds for a remote URL (0 = off)

    // Authorization (see "Authorization" below)
    ->canManage(fn ($user) => $user?->hasRole('admin')) // bool|Closure|null — null defers to the policy
    ->policy(\App\Policies\ChangelogEntryPolicy::class)  // string|Closure|null — policy bound to the model

    // Routing & data
    ->slug('changelog')                         // string|Closure|null — reader page route slug
    ->resourceSlug('changelog/manage')          // string|Closure|null — management route slug
    ->multiProject(false);                      // bool|Closure|null — scope entries by a "project" column
```

### Authorization (who can manage)

The read-only **Changelog** page is always visible; the **Manage** area
(create / edit / delete) is guarded. Control it in two ways:

```php
// 1) A simple gate — block everyone, or decide per user:
ChangelogPlugin::make()
    ->canManage(fn ($user) => $user?->hasRole('admin'));

// 2) Do nothing and it defers to the model's policy / Gate —
//    so Filament Shield permissions work out of the box:
//    php artisan shield:generate  → ViewAny/Create/Update/Delete are enforced.
```

When management is denied, the routes return `403` and the **Manage** button on
the reader page is hidden automatically.

### Filament Shield

The plugin is **Shield-ready with no hard dependency** — it works with or
without Shield installed:

- **Detected automatically.** The resource and the reader page are registered on
  the panel, so `php artisan shield:generate` discovers them and creates their
  permissions (nothing to configure).
- **Enforced automatically.** Shield generates the policy under `App\Policies`;
  because the model lives in the package namespace, the plugin binds that policy
  to the model for you (override with `config('changelog.policy')`). The reader
  page also honours its generated Shield permission at runtime.
- **Without Shield**, everything stays open/functional and no Shield classes are
  referenced.

Prefer explicit wiring? Point `canManage()` at a Shield permission:

```php
ChangelogPlugin::make()
    ->canManage(fn ($user) => $user?->can('Update:ChangelogEntry'));
```

## Usage

### In the panel

A single **Changelog** item appears in the navigation (the read-only reader
page). From there, the **Manage** button opens the management table to
create/edit entries and **Import** / **Export** a `CHANGELOG.md` file; a
**Changelog** button takes you back.

### From the CLI

```bash
# Import CHANGELOG.md into the database
php artisan changelog:import

# Import a specific file, wiping existing entries first
php artisan changelog:import path/to/CHANGELOG.md --fresh

# Generate CHANGELOG.md from the database
php artisan changelog:export

# Print instead of writing
php artisan changelog:export --print
```

## Translations

Ships with **English (`en`)**, **European Portuguese (`pt_PT`)** and **Brazilian
Portuguese (`pt_BR`)**. The UI follows the app locale (`config('app.locale')`).
Add more languages by publishing the translations:

```bash
php artisan vendor:publish --tag="changelog-translations"
```

> The generated `CHANGELOG.md` always uses the canonical English section
> headings (`### Added`, `### Fixed`, …) regardless of locale, so the file stays
> valid Keep a Changelog and round-trips cleanly.

### Changelogs written in other languages

Import and the live file reader also understand a `CHANGELOG.md` written in any
bundled language. Section headings are matched against the label the package
renders in each language, ignoring accents, case and extra spaces, so
`### Adicionado` reads as *Added* and `### Corrigido` as *Fixed*. The unreleased
section is recognised however it is spelled — `## [Unreleased]`,
`## [Não lançado]`, `## [Não publicado]`, `## [Sem lançamento]` or the
translated label — and always sorts first.

Adding a translation to the package also teaches the reader to parse files
written in that language. The matching lives in `Support\Headings`, public so an
application can reuse it when it extends the parser.

## Reader source & file path

The reader page can read from two sources, set in `config/changelog.php` (or via
env), so you point it at **your own project** and it never shows a stale copy:

```env
# 'database' (default) — entries edited in the panel
# 'file' — parses your CHANGELOG.md live on every visit (never "dead")
CHANGELOG_SOURCE=file

# Your project's changelog. Relative → from base_path(); absolute → used as-is.
CHANGELOG_FILE=CHANGELOG.md
# CHANGELOG_FILE=/srv/my-other-project/CHANGELOG.md
```

Or configure it fluently on the plugin (this wins over config/env):

```php
ChangelogPlugin::make()
    ->fromFile(base_path('CHANGELOG.md'));   // = ->source('file')->file(...)
// or, granularly:
ChangelogPlugin::make()
    ->source('file')                          // 'database' | 'file'
    ->file('/srv/my-project/CHANGELOG.md');
```

The file can also be a **remote URL** — e.g. read your `CHANGELOG.md` straight
from GitHub. Remote content is cached (`CHANGELOG_REMOTE_CACHE_TTL`, default
300s; `0` disables it):

```php
ChangelogPlugin::make()
    ->fromUrl('https://raw.githubusercontent.com/acme/app/main/CHANGELOG.md');
```

You can also paste the **normal GitHub file URL** — `github.com/.../blob/...`
links are auto-converted to their raw form, so this works too:

```php
ChangelogPlugin::make()
    ->fromUrl('https://github.com/acme/app/blob/main/CHANGELOG.md');
```

In `file` mode the panel's edit/export buttons are hidden — the file is the
single source of truth, and every page load re-parses it, so what you see always
matches the file on disk.

## Configuration

See `config/changelog.php` — reader source, file path, table name, change-type
order, date format, multi-project toggle and navigation placement.

## License

MIT © Anselmo Kossa
