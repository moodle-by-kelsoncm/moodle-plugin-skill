# Changelog

## 1.3.0 - 2026-10-31

Lessons from `tool_categorycustomfields` 1.0.0 and `tiny_toolbar` 1.0.0, and a review against the Moodle 5.3 developer documentation (moodledev.io and `UPGRADING.md`).

- `SKILL.md`, versions: `$plugin->component` and the optional `$plugin->incompatible`.
- `SKILL.md`, JavaScript: `core/modal` classes instead of the removed `core/modal_factory` and of jQuery modals, Bootstrap 5 markup and classes, colours through Bootstrap custom properties for the 5.3 colour modes.
- `SKILL.md`, External Services: no `require_login()` in `execute()`, advisory `capabilities`, `classname` only, deprecation with `execute_is_deprecated()`.
- `SKILL.md`, namespaced output, URL and exception classes available since 4.5.
- `SKILL.md`, custom fields: a custom field area of your own (handler, management page, cleanup on delete, test generator).
- `SKILL.md`, new item **Pages and hooks**: own pages linked from navigation callbacks, category secondary navigation, `db/hooks.php` rules, `after_standard_main_region_html_generation` to add content to core pages.
- `SKILL.md`, new item **TinyMCE plugins**: only `tiny_` plugins change the editor, `tiny/<x>:use` checked in `is_enabled()` for 4.5, toolbar and menus applied in the TinyMCE plugin callback, button registry, selection toolbar, `editor_styles.css`, 5.0-only interfaces.
- `SKILL.md`, new items **Tests** (PHPUnit 9.6 and 11, annotations, static providers, Behat escaping), **Activity modules on 5.x**, **Deprecating your own code** and **Subplugins**.
- `SKILL.md`, repository: renaming a repository or changing the plugin type; Pages on private repositories; new reference implementations.
- `references/act.md`: iterating on PHP in the job container (phpcbf, a single test suite), RequireJS developer mode and source maps, seed scripts, screenshots.
- `references/release-marketplace.md`: normal release for a non-stable maturity; editing a published release.

## 1.2.0 - 2026-10-30

Lessons from `profilefield_json` 1.1.0 and `customfield_json` 1.0.0.

- `SKILL.md`, JavaScript: third-party UMD libraries in `thirdparty/` loaded with `window.require`, their stylesheet added from JavaScript, page language for libraries, stale `getString()` answers.
- `SKILL.md`, new item **Forms**: `data-*` markers instead of classes, `.felement` flex, JavaScript of elements in dynamic (modal) forms required at render time through a registered element subclass, random id suffixes.
- `SKILL.md`, new item **Profile fields and custom fields**: validation on save paths that skip the form, custom field privacy export of the stored value, escaped storage decoded for display.
- `SKILL.md`, repository: first commit of an empty repository and GitHub Pages before the first documentation deploy; new reference implementations.
- `references/release-marketplace.md`: first submission of a new plugin.
- `references/act.md`: scripted edit pitfalls (form feeds and Unicode escapes), `pgrep` self-match, remounting the throwaway Moodle.

## 1.1.0 - 2026-10-30

- `SKILL.md`: new item **Database (`db/`)**: required columns of an activity main table (`course` with index, `name`, `intro`/`introformat`, `timemodified`), upgrade steps with `<` and savepoints, guarded DDL, backfill of new relation columns, and the schema check after upgrade. Found on an activity module whose missing `course` column broke the deletion of every course on Moodle 5.3.
- `references/review-checklist.md`: matching blocker entry.

## 1.0.0 - 2026-10-30

First public version of the `moodle-plugin` skill, generalised from the skill used on the EduPlay Moodle Suite plugins and on `mod_imagemap`.

- `SKILL.md`: plugin standard (versions, language, boilerplate, CSS, templates, ES6 AMD, External Services, input cleaning, backup/restore for activities, blocks and other plugin types, Privacy API, admin pages, availability messages), repository layout, local validation, documentation and release.
- `references/review-checklist.md`: Moodle Marketplace review checklist.
- `references/act.md`: moodle-plugin-ci with act, job container shortcuts, legacy JavaScript conversion, throwaway Moodle for browser tests.
- `references/release-marketplace.md`: draft releases, verification, Marketplace upload (manual and automatic), plugin page and resubmission.
- Packaged as a Claude Code plugin marketplace (`moodle-plugin@moodle-by-kelsoncm`).
