---
name: moodle-plugin
description: Use whenever you create, review, fix, document, version or publish a Moodle plugin (mod_, local_, block_, tool_, tiny_, media_, repository_, filter_, format_, theme_, qtype_ and others), including answering the Moodle Marketplace reviewer (MMR-* or CONTRIB-* tickets), preparing a release, filling in the plugin page on marketplace.moodle.com, writing the plugin documentation or running moodle-plugin-ci locally with act. Encodes what passes the Marketplace review and what it rejects.
license: MIT
metadata:
  version: "1.0.0"
  repository: "https://github.com/moodle-by-kelsoncm/moodle-plugin-skill"
---

# Moodle plugins: development, review and publishing

## Why
Plugins built this way passed the Moodle Marketplace review on the first resubmission. The reviewer's findings on another plugin (backup/restore, `PARAM_RAW`, legacy AJAX, inline JavaScript, missing boilerplate...) cost a full round of rework. Follow this skill so a new plugin is approvable from the start.

Reference implementations (public, follow them when in doubt):
- `eduplay-moodle-suite/moodle-local_eduplay`, plus `moodle-media_eduplay`, `moodle-tiny_eduplay` and `moodle-repository_eduplay`: approved plugins with the CI, release and documentation setup described here.
- `moodle-by-kelsoncm/mod_imagemap` 1.3.0: an activity module fixed after a Marketplace review (backup/restore, External Services, AMD ES6).

## Before you start
- Follow the project's own conventions for issues, branches, commit messages and time tracking. If it has none, use one issue per reviewer finding and one branch per change.
- When the reviewer sends many findings, group them into a few stacked pull requests. Close the issues only after CI is green on the default branch. Merge only with the maintainer's approval.
- Before you add or edit workflows, check the latest major version of every action (`gh api repos/<owner>/<action>/releases/latest --jq .tag_name`); never write action versions from memory.

## Plugin standard
1. **Versions:** Moodle 4.5 LTS and 5.3 LTS.
   - `$plugin->requires = 2024100700;`
   - `$plugin->supported = [405, 503];`
   - `$plugin->release` in semver.
   - `$plugin->maturity` must match the release: a non-stable maturity is published as a pre-release.
2. **Language:** ship only `lang/en`; other languages come through AMOS. Keep strings in alphabetical order and without section comments (the code checker warns about both).
3. **GPL boilerplate in every file**, including `amd/src` JavaScript, CSS and Mustache. See [references/review-checklist.md](references/review-checklist.md).
4. **CSS:** only `styles.css`, which Moodle loads on its own. Never `$PAGE->requires->css()` for it. Scope rules with the page class (`.path-mod-<x>`) or plugin-specific classes.
5. **HTML** comes from templates (`$OUTPUT->render_from_template`). No `echo '<div>'` in pages.
6. **JavaScript:**
   - ES6 modules in `amd/src`, started with `$PAGE->requires->js_call_amd()`.
   - No `js_init_code`, no `onclick`, no `confirm()`, no `console`.
   - Large data goes in a `data-config` (JSON) attribute of the template; `js_call_amd` complains about arguments over 1024 characters.
   - Confirm with `core/notification` (`saveCancelPromise`).
   - Bootstrap modals through jQuery (`$(el).modal('show')`), which works with Bootstrap 4 (Moodle 4.5) and 5.
7. **`amd/build`:** commit the `*.min.js` and `.map` files built by Moodle's grunt. Never commit builds without `.min`.
8. **AJAX:** External Services in `classes/external/*` (`core_external\external_api`) and `db/services.php` (`'ajax' => true`, `capabilities`).
   - In `execute`: `validate_parameters`, `validate_context`, `require_capability`.
   - Check that every received id belongs to the instance of the course module (IDOR).
   - JavaScript calls them through `core/ajax`.
9. **Input:** never `PARAM_RAW`. Use the specific type; free text that becomes style or HTML goes through a dedicated sanitiser with an allow list, covered by PHPUnit.
10. **Errors:** `throw new moodle_exception(...)`; `print_error()` is gone.
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

## Repository
Copy from `moodle-local_eduplay` and adapt names and texts:
- `.github/workflows/`: `ci.yml`, `docs.yml`, `release.yml`, `release-check.yml` and `moodle-release.yml`.
- `.github/scripts/release.sh`, `.github/dependabot.yml`, `docs/requirements.txt` and `docs/redirect/index.html`.

Root files:
- `README.md`, short and pointing to the documentation.
- `CHANGELOG.md` with a `## <release> - <YYYY-MM-DD>` section; `release.sh` reads it.
- `SECURITY.md`, `AUTHORS.md`, `LICENSE`.
- Optionally a short `AGENTS.md` with the code rules.

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
