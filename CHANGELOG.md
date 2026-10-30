# Changelog

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
