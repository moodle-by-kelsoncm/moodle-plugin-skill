---
name: moodle-plugin
description: Use whenever you create, review, fix, document, version or publish a Moodle plugin (mod_, local_, block_, tool_, tiny_, media_, repository_, filter_, format_, theme_, qtype_ and others), including answering the Moodle Marketplace reviewer (MMR-* or CONTRIB-* tickets), preparing a release, filling in the plugin page on marketplace.moodle.com, writing the plugin documentation or running moodle-plugin-ci locally with act. Encodes what passes the Marketplace review and what it rejects.
license: MIT
metadata:
  version: "1.3.0"
  repository: "https://github.com/moodle-by-kelsoncm/moodle-plugin-skill"
---

# Moodle plugins: development, review and publishing

## Why
Plugins built this way passed the Moodle Marketplace review on the first resubmission. The reviewer's findings on another plugin (backup/restore, `PARAM_RAW`, legacy AJAX, inline JavaScript, missing boilerplate...) cost a full round of rework. Follow this skill so a new plugin is approvable from the start.

Reference implementations (public, follow them when in doubt):
- `eduplay-moodle-suite/moodle-local_eduplay`, plus `moodle-media_eduplay`, `moodle-tiny_eduplay` and `moodle-repository_eduplay`: approved plugins with the CI, release and documentation setup described here.
- `moodle-by-kelsoncm/mod_imagemap` 1.3.0: an activity module fixed after a Marketplace review (backup/restore, External Services, AMD ES6).
- `moodle-by-kelsoncm/profilefield_json` 1.1.0 and `moodle-by-kelsoncm/customfield_json` 1.0.0: a profile field and a custom field with a bundled third-party JavaScript library, validation on every save path and an editor that also starts inside modal forms.
- `moodle-by-kelsoncm/tool_categorycustomfields` 1.0.0: a custom field area of its own (course categories), an output hook that adds content to a core page, a web service and an event observer.
- `moodle-by-kelsoncm/tiny_toolbar` 1.0.0: a TinyMCE plugin that replaces the toolbar and menus of every editor, with a visual builder fed by the editor's button registry.

## Before you start
- Follow the project's own conventions for issues, branches, commit messages and time tracking. If it has none, use one issue per reviewer finding and one branch per change.
- When the reviewer sends many findings, group them into a few stacked pull requests. Close the issues only after CI is green on the default branch. Merge only with the maintainer's approval.
- Before you add or edit workflows, check the latest major version of every action (`gh api repos/<owner>/<action>/releases/latest --jq .tag_name`); never write action versions from memory.

## Plugin standard
1. **Versions:** Moodle 4.5 LTS and 5.3 LTS.
   - `$plugin->component` (required: install and upgrade check the code location against it).
   - `$plugin->requires = 2024100700;`
   - `$plugin->supported = [405, 503];`
   - `$plugin->release` in semver.
   - `$plugin->maturity` must match the release: a non-stable maturity is published as a pre-release.
   - `supported` does not block installs on later branches; add `$plugin->incompatible = <first unsupported branch>` only when you know it breaks.
2. **Language:** ship only `lang/en`; other languages come through AMOS. Keep strings in alphabetical order and without section comments (the code checker warns about both).
3. **GPL boilerplate in every file**, including `amd/src` JavaScript, CSS and Mustache. See [references/review-checklist.md](references/review-checklist.md).
4. **CSS:** only `styles.css`, which Moodle loads on its own. Never `$PAGE->requires->css()` for it. Scope rules with the page class (`.path-mod-<x>`) or plugin-specific classes. The only other stylesheet is the one shipped with a third-party library (item 6).
5. **HTML** comes from templates (`$OUTPUT->render_from_template`). No `echo '<div>'` in pages.
6. **JavaScript:**
   - ES6 modules in `amd/src`, started with `$PAGE->requires->js_call_amd()`.
   - No `js_init_code`, no `onclick`, no `confirm()`, no `console`.
   - Large data goes in a `data-config` (JSON) attribute of the template; `js_call_amd` complains about arguments over 1024 characters.
   - Confirm with `core/notification` (`saveCancelPromise`).
   - Modals with the `core/modal` classes (`Modal`, `ModalSaveCancel.create()`, `ModalDeleteCancel`...), available since 4.3. `core/modal_factory` and `core/modal_registry` were removed in 5.2. A Bootstrap modal already in the page markup may still be opened with jQuery (`$(el).modal('show')`) on 4.5 and 5.x, but only through the Bootstrap 4 compatibility layer, which goes in 6.0; prefer `core/modal`. Since 5.3 the modal title is an `<h2>`: headings in the body start at `<h3>`.
   - Bootstrap 5 markup and classes (`data-bs-*`, `ms-*`/`me-*`, `visually-hidden`, `form-select`, `fw-*`, `text-bg-*`); the compatibility layer rewrites `data-toggle`/`data-target` only in markup printed by the server, never in markup built by JavaScript. On 5.3, AMD modules must not import `theme_boost/bootstrap/*`.
   - Colours in `styles.css` through Bootstrap custom properties (`var(--bs-body-bg)`, `var(--bs-body-color)`, `var(--bs-border-color)`...), not literal values: Boost 5.3 has colour modes (`data-bs-theme` on `<html>`), and the properties also exist on 4.5.
   - **Third-party libraries** (UMD builds): keep them out of `amd/src`, where grunt warns "File ignored because of a matching ignore pattern" even when `thirdpartylibs.xml` lists them. Put them in `thirdparty/<lib>/`, list that folder in `thirdpartylibs.xml` and load them from your module with `window.require([Config.wwwroot + '/<plugin path>/thirdparty/<lib>/<lib>.min.js'], resolve, reject)`. Remove the `sourceMappingURL` line when the `.map` is not shipped. Add the library stylesheet from JavaScript (a `<link>` appended once to `document.head`): it also works in forms loaded in modals, after the page head was printed.
   - Libraries that pick their language from the browser must get the page language (`document.documentElement.lang`).
   - `getString()` is asynchronous: when it runs on every keystroke, ignore answers to older calls (a counter), or an old message comes back.
7. **`amd/build`:** commit the `*.min.js` and `.map` files built by Moodle's grunt. Never commit builds without `.min`.
8. **AJAX:** External Services in `classes/external/*` (`core_external\external_api`) and `db/services.php` (`'ajax' => true`, `capabilities`).
   - In `execute`: `validate_parameters`, `validate_context`, `require_capability`. Never `require_login()` or `$PAGE->set_context()` there.
   - `capabilities` in `db/services.php` is only advisory: still call `require_capability()`. Use `classname` only (no `methodname` or `classpath`). Top-level parameters cannot be optional.
   - Check that every received id belongs to the instance of the course module (IDOR).
   - JavaScript calls them through `core/ajax`.
   - To deprecate a function, add a static `execute_is_deprecated()` returning `true` (not the deprecation attribute).
9. **Input:** never `PARAM_RAW`. Use the specific type; free text that becomes style or HTML goes through a dedicated sanitiser with an allow list, covered by PHPUnit.
10. **Errors:** `throw new moodle_exception(...)`; `print_error()` is gone. Since 4.5, the namespaced names (`\core\exception\moodle_exception`, `\core\url`, `\core\output\renderer_base`, `plugin_renderer_base`, `renderable`, `templatable`, `html_writer`) exist and can be used directly; navigation classes moved to `\core\navigation\*` only in 5.1, so keep the old names while 4.5 is supported.
11. **Backup/restore:** every plugin that stores course-related data needs it.
    - All classes live in `backup/moodle2/`, never in `restore/moodle2/`.
    - **Activity module** (`mod_`): `backup_<x>_activity_task.class.php`, `backup_<x>_stepslib.php`, `restore_<x>_activity_task.class.php` and `restore_<x>_stepslib.php`. Declare `FEATURE_BACKUP_MOODLE2` in `<x>_supports()`.
    - **Block** (`block_`): `backup_<x>_block_task.class.php` and `restore_<x>_block_task.class.php` (plus a stepslib for own tables).
      - Core already copies the instance `configdata`.
      - The task implements `get_fileareas()` and `get_configdata_encoded_attributes()`, which are abstract.
      - If `configdata` keeps course or activity ids, remap them by overriding `decode_configdata()` or in `after_restore()`.
    - **Other types** (`qtype_`, `format_`, `local_`, `report_`, `enrol_`...): `backup_<type>_<x>_plugin` and `restore_<type>_<x>_plugin` classes, attached to the course, section, activity or question structure (`define_course_plugin_structure()` and friends).
    - Annotate every file area, including `intro`.
    - Ids of other activities or sections: remap them in the task's `after_restore()`, once the whole course is restored. Without a mapping, keep the id if it belongs to the target course (duplication); otherwise clear it.
    - Test a real course backup and restore, an activity placed after the plugin, and duplication (`cmactions::duplicate` on Moodle 5.2+, `duplicate_module` before). Do not `require` the backup classes by hand in tests: it hides a wrong location.
12. **Privacy API** always present (null provider if no personal data is stored).
13. **Admin pages:** the first call after `config.php` must be visible to the code checker (`require_admin()`, `require_login()` or `admin_externalpage_setup()`). Do not hide it in your own function.
14. **Availability:** pass `availableinfo` through `\core_availability\info::format_info($info, $course)`; otherwise the activity names disappear from the message.
15. **Database (`db/`):**
    - **Activity module main table** (`<x>`) always has `course` (int 10, not null, default 0, indexed) and `name`; add `intro` and `introformat` when `FEATURE_MOD_INTRO` is on (the default) and `timemodified`. Core reads `{<x>}.course` for every installed module when it deletes or resets *any* course (`remove_course_contents()`); without it every course deletion fails with "column m.course does not exist" and leaves the course half-deleted, even in courses that never used the activity.
    - Edit `install.xml` with the XMLDB editor and keep it equal to what the upgrade steps produce.
    - `upgrade.php`: one `if ($oldversion < X)` block per change, ending with `upgrade_mod_savepoint(true, X, '<x>')` (or the plugin type's savepoint), with `X` no greater than `version.php`. Never `<=`: the block runs again on the next version bump.
    - Guard DDL with `field_exists()`, `index_exists()` and `table_exists()`, so the step also works on sites installed from an old `install.xml`.
    - A new relation column on existing data needs a backfill in the same step (for `course`: from `{course_modules}` joined on `module` and `instance`).
    - After the upgrade, run `php admin/cli/check_database_schema.php`; it must report nothing for the plugin. Then delete a course on the test site.
16. **Forms:**
    - The form templates replace the `class` attribute of elements such as `textarea`: mark them with a `data-*` attribute instead, and use a selector more specific than `.form-control` to restyle or hide them.
    - `.felement` is a flex container: a component injected next to an element needs `flex: 1 1 100%; width: 100%`.
    - **Dynamic forms** (`core_form\dynamic_form`, the modals of custom field settings and others) build the form definition before they start collecting the page JavaScript: `js_call_amd()` called in `definition()` is lost. Require the module while the element is rendered, in a subclass of the element registered with `MoodleQuickForm::registerElementType()` that overrides `export_for_template()` (and `toHtml()`), skipping frozen elements.
    - Element ids get a random suffix in dynamic forms: pass the id read from the element at render time, never a fixed `id_<name>`.
17. **Profile fields and custom fields:**
    - User upload, course upload and web services save values without running the form validation (`profile_save_data()`, `instance_form_save()`). Validate again where the value is saved (`edit_save_data()` for profile fields, `instance_form_save()` for custom fields), keep the stored value when the new one is invalid, and cover it with PHPUnit (`assertDebuggingCalled()` if you report it with `debugging()`).
    - A custom field privacy provider implements `customfield_provider`; the `$exportdata->value` it receives is the displayed value (`export_value()`, often HTML). Export the stored value (`$data->get_value()`) when the display differs from the data.
    - Data stored escaped (for example JSON with `JSON_HEX_TAG`) is decoded for display and escaped by the template, never printed as stored.
    - **A custom field area of your own** (for an entity core gives none, such as course categories): a handler `<component>\customfield\<area>_handler` extending `core_customfield\handler` (the class name is parsed: no other namespace or suffix), with `can_configure()`, `can_edit()`, `can_view()`, `get_configuration_context()`, `get_configuration_url()`, `get_instance_context()` and the area settings in `config_form_definition()`. The management page is `core_customfield\output\management` rendered with the `core_customfield` renderer, behind an `admin_externalpage`. Delete the values when the entity is deleted (event observer calling `delete_instance()`); core does not. In tests, create the field category with `create_category(['component' => ..., 'area' => ...])` of the `core_customfield` generator.
18. **Pages and hooks:**
    - Core pages without a hook for your need (for example the category edit form): add your own page and link it from the navigation callback (`<plugin>_extend_navigation_category_settings()`, `_extend_settings_navigation()` and friends).
    - A page of a category gets the category secondary navigation with `$PAGE->set_category_by_id()` (not only `set_context()`); highlight your node with `set_secondary_active_tab('<node key>')`. Narrow forms look right with `$PAGE->add_body_class('limitedwidth')`.
    - Hooks: callbacks in `db/hooks.php` (`hook`, `callback` as `[class, 'method']`, optional `priority`, default 100, higher first), static methods. Registrations are cached: bump the version after changing them. When a hook callback exists, Moodle ignores the matching legacy `lib.php` callback. Callbacks may run during install or upgrade: guard them with `during_initial_install()` or `$CFG->upgraderunning` when they need the database. Test with `\core\di::set(\core\hook\manager::class, \core\hook\manager::phpunit_get_instance([...]))`.
    - To add content to a core page, use `db/hooks.php` with `core\hook\output\after_standard_main_region_html_generation` (available on 4.5), filter by `$PAGE->pagetype` and context, render a template and, if the content belongs higher up the page, move it with an AMD module (`js_call_amd()` from the callback works). Without JavaScript it stays at the end of the main region.
19. **TinyMCE plugins (`tiny_`):**
    - Only `tiny_` plugins can change the editor; a `local_` or `tool_` callback such as `<plugin>_editor_tiny_get_config_data()` is never called. Search Moodle for any callback before relying on it.
    - `classes/plugininfo.php` extends `editor_tiny\plugin`; options for the JavaScript come from `plugin_with_configuration::get_plugin_configuration_for_context()` and are read with `editor.options.get(getPluginOptionName(pluginName, '<option>'))` after `editor.options.register()` in the plugin.
    - Moodle 5.x expects `tiny/<x>:use` in `db/access.php` (otherwise a debugging message: "does not define the standard capability") and checks it; Moodle 4.5 does not. Define it and check it yourself in `is_enabled()` so both versions behave the same.
    - The `configure()` functions run in plugin order and later plugins add to the toolbar after yours. To have the final word on `toolbar`, `menubar` or `quickbars_selection_toolbar`, call `editor.options.set()` in the TinyMCE plugin callback (`tinyMCE.PluginManager.add(name, (editor) => ...)`): it runs after every plugin registered its buttons and before the interface is drawn. A `setup` returned by `configure()` is too early: those options are not registered yet and `options.set()` returns `false`.
    - `editor.ui.registry.getAll().buttons` lists the buttons of the page with their Moodle names (`tiny_link_link`, `tiny_media_image`...), labels (`editor.translate(spec.tooltip)`) and icons (`registry.icons[spec.icon]`). Items TinyMCE draws itself (`blocks`, `styles`, `fontfamily`, `fontsize`, `align`, `lineheight`) are not in the registry. Some Moodle buttons, such as `tiny_link_link`, do not show in the selection toolbar.
    - Moodle turns the selection toolbar off (`false`) in text areas of 5 rows or less: keep it off there.
    - The editor is only used for `FORMAT_HTML` fields; a description stored in `FORMAT_MOODLE` opens in a plain text area, which is not a bug of your plugin.
    - Write the default export as `export default getTinyMCE().then((tinyMCE) => {...; return [pluginName, Configuration];})`; `new Promise(async (resolve) => ...)` fails eslint (`no-async-promise-executor`). The `PluginManager.add` callback must be synchronous.
    - Styles for the edited content go in `editor_styles.css` (the content is in an iframe); `styles.css` only styles the page.
    - `plugin_with_configuration_for_external` and `is_enabled_for_external()` exist only from 5.0: implementing the interface breaks the install on 4.5.
20. **Tests (PHPUnit 9.6 on 4.5, PHPUnit 11 on 5.x):**
    - Use `@covers`, `@dataProvider` and the other annotations at class level, not PHPUnit attributes, while 4.5 is supported. Make data providers `public static` and have them return data only (no generators).
    - No `withConsecutive()`, `willReturnConsecutive()` or `onConsecutive()` (gone in 5.0). `setUp()` and `tearDown()` call the parent. From 5.1 deprecations and warnings fail the run.
    - Test files end in `_test.php` and their classes are `final`; plugin generators live in `tests/generator/lib.php`. External function tests extend `\core_external\tests\externallib_testcase` (present on 4.5 and 5.3).
    - Behat on 5.0+ (behat/gherkin 4.11): escape backslashes in `.feature` files.
21. **Activity modules on 5.x:** `FEATURE_MOD_PURPOSE` is required (`FEATURE_MOD_OTHERPURPOSE` is optional and 5.1+). Optional: a `mod_<x>\courseformat\overview` class for the course overview page (5.0+), the strings `modulename_summary` and `modulename_tip` (5.2+). Course module actions go through `core_courseformat\local\cmactions` on 5.2+ (`delete`, `move_before`, `duplicate`).
22. **Deprecating your own code:** `#[\core\attribute\deprecated('<replacement>', since: '<x.y>', mdl: '<issue>')]` plus `\core\deprecation::emit_deprecation_if_present([self::class, __FUNCTION__])` in the body, keeping the signature (4.4+).
23. **Subplugins:** from 5.0, `db/subplugins.json` uses `subplugintypes` with paths relative to the plugin; keep `plugintypes` too for 4.5. From 5.1, define the strings `type_<type>` and `type_<type>_plural`.

## Repository
Copy from `moodle-local_eduplay` and adapt names and texts:
- `.github/workflows/`: `ci.yml`, `docs.yml`, `release.yml`, `release-check.yml` and `moodle-release.yml`.
- `.github/scripts/release.sh`, `.github/dependabot.yml`, `docs/requirements.txt` and `docs/redirect/index.html`.

Root files:
- `README.md`, short and pointing to the documentation.
- `CHANGELOG.md` with a `## <release> - <YYYY-MM-DD>` section; `release.sh` reads it.
- `SECURITY.md`, `AUTHORS.md`, `LICENSE`.
- Optionally a short `AGENTS.md` with the code rules.

New repository:
- An empty repository has no default branch to receive a pull request: push a first commit (for example only `LICENSE`) to `main`, then open the pull request from the work branch.
- Enable GitHub Pages with the Actions source before the first documentation deploy (`gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow`); otherwise the deploy job of `docs.yml` fails. On a private repository the deploy fails until it is public.

Renaming or changing the plugin type:
- `gh repo rename` keeps a redirect from the old URL; update the local remote and every workspace that clones or mounts the old name.
- A new component is a new plugin for Moodle: sites must uninstall the old one, and its settings are not migrated. Say so in the installation page and the changelog.

Do not commit `docs/_build`, `dist/` or ZIP files. Avoid pre-commit hooks, docker-compose or composer setups that need PHP on the developer machine: validate locally with `act`.

## Local validation (before every push)
Run `ci.yml` with `act` for one combination (Moodle 5.3, PHP 8.3, pgsql) and `docs.yml`. Recipe, Windows notes and shortcuts inside the job container: [references/act.md](references/act.md).

What must pass:
- phplint, codechecker and phpdoc with `--max-warnings 0`;
- validate, savepoints, mustache;
- grunt with `--max-lint-warnings 0`, which also checks that `amd/build` matches `amd/src`;
- phpunit with `--fail-on-warning`;
- sphinx with `-W`.

Two failures are expected under `act` and do not count:
- **Mustache:** the HTML validator does not run under `act` ("Problem calling HTML validator"); check it on GitHub.
- **phpcpd:** broken in moodle-plugin-ci 4; keep it with `continue-on-error`.

Also test in a browser on a throwaway Moodle (see [references/act.md](references/act.md)). Broken UI flows pass CI.

## Documentation
- Sphinx with `moodle-docs-theme`, one folder per language (for example `docs/en` and `docs/pt_BR`). `docs.yml` publishes to GitHub Pages, with a language redirect at the root.
- Pages: `index` (what it does, a figure, main features, links), `installation`, `configuration`, `usage` and plugin-specific pages (for example `backup_restore`).
- Real screenshots in `docs/*/img`, taken on the throwaway Moodle with test data. The sample image must look like real use.
- Describe the actual behaviour. Read the code first: inherited documentation often promises features that do not exist.

## Release and Marketplace
Steps, pitfalls and the marketplace.moodle.com forms: [references/release-marketplace.md](references/release-marketplace.md).

## Marketplace review
Checklist of what the reviewer flagged, with the fix that passed: [references/review-checklist.md](references/review-checklist.md). Go through it before every submission or new version.
