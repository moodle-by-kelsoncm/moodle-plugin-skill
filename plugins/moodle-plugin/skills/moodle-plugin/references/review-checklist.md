# Moodle Marketplace review checklist

Findings from real Marketplace reviews (four approved EduPlay plugins and the `mod_imagemap` review), with the fix that passed.

## Blockers
- [ ] **Backup/restore** (activity modules, blocks and any plugin with course-related data). Restore classes live in `backup/moodle2/`. A `restore/moodle2/` folder is never read, and a test that `require`s the classes by hand hides the problem. Test with a real course backup and restore.
- [ ] **Marketplace page.** The description belongs to this plugin, with correct dependencies and install path. One plugin was flagged because its description was copied from a sibling plugin.

## Security
- [ ] No `PARAM_RAW`. Free text goes through a dedicated sanitiser with an allow list and tests.
- [ ] Every write endpoint: `require_sesskey` or an External Service, `require_capability` in the right context, and a check that the record belongs to the instance of the course module.
- [ ] No declared capability without use (the reviewer asked to remove an unused `manage` capability).
- [ ] Every declared setting is used (a cache duration setting had no effect).

## Deprecated APIs and standards
- [ ] `print_error()` becomes `throw new moodle_exception()`.
- [ ] `AJAX_SCRIPT` scripts become External Services (`classes/external`, `db/services.php`, `core/ajax`).
- [ ] HTML in `echo` becomes a template; inline JavaScript becomes AMD (`js_call_amd`).
- [ ] JavaScript in ES6 (`import`/`export`), not legacy `define([], function () {...})`.
- [ ] `amd/build/*.min.js` present and equal to the grunt build.
- [ ] No `$PAGE->requires->css()` for the plugin's `styles.css`.
- [ ] Every string used exists in `lang/en`. Cross-check `get_string`, `{{#str}}`, `new lang_string`, `moodle_exception` and `core/str`.

## Boilerplate
- [ ] PHP: GPL header `// This file is part of Moodle ...` and a docblock with `@package`, `@copyright`, `@license`.
- [ ] JavaScript (`amd/src`): the same GPL header and a docblock with `@module <component>/<name>`.
- [ ] CSS: GPL header in a `/* ... */` comment.
- [ ] Mustache: a `{{! GPL }}` block and a block with `@template <component>/<name>`, a description and `Example context (json):`.

## Only found by testing
- [ ] Mustache has no `{{else}}`: use `{{^var}}...{{/var}}`.
- [ ] `availableinfo` without `format_info()` loses the activity names.
- [ ] Editor actions that change data (drag, double click, delete) call the save service.
- [ ] An editor that runs inside an iframe or a third-party editor (TinyMCE, H5P) may not even start on the reviewer's site. Explain on the plugin page how to test it.

## Marketplace description
- [ ] Plain text or HTML, no raw Markdown.
- [ ] Link to the documentation.
- [ ] What the plugin does, dependencies, supported Moodle versions and where to configure it.
- [ ] Privacy: what leaves Moodle and where it goes.
- [ ] "Unofficial project" notice when the plugin integrates a third-party service.
