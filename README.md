# moodle-plugin-skill

[English](#english) · [Português (Brasil)](#português-brasil)

## English

An [Agent Skill](https://agentskills.io) for building, reviewing, documenting and releasing **Moodle plugins that pass the Moodle Marketplace review**. It is packaged as a Claude Code plugin, and the skill folder also works in other agents that read `SKILL.md`.

It encodes what was learned on approved plugins ([EduPlay Moodle Suite](https://github.com/eduplay-moodle-suite)) and on a plugin fixed after a review ([mod_imagemap](https://github.com/moodle-by-kelsoncm/mod_imagemap)):

- **Plugin standard:** Moodle 4.5 and 5.3 LTS, `lang/en` only, GPL boilerplate everywhere, templates, ES6 AMD modules with `amd/build`, External Services instead of AJAX scripts, no `PARAM_RAW`, backup/restore for activities, blocks and other plugin types, Privacy API.
- **Review checklist:** what the Marketplace reviewer flags, with the fix that passed.
- **Local validation:** `moodle-plugin-ci` through [`act`](https://github.com/nektos/act) (Windows notes included), eslint/grunt inside the job container, a throwaway Moodle for browser tests and screenshots.
- **Documentation:** Sphinx in several languages published on GitHub Pages.
- **Release and Marketplace:** draft releases with verification, manual or automatic upload to the Marketplace, the plugin page and resubmission after a review.

### Install in Claude Code

```text
/plugin marketplace add moodle-by-kelsoncm/moodle-plugin-skill
/plugin install moodle-plugin@moodle-by-kelsoncm
```

The skill loads by itself when a task is about a Moodle plugin. You can also call it as `/moodle-plugin:moodle-plugin`.

### Use in other agents

Copy or link `plugins/moodle-plugin/skills/moodle-plugin` into the agent's skills folder. Example:

```bash
git clone https://github.com/moodle-by-kelsoncm/moodle-plugin-skill.git ~/moodle-plugin-skill
ln -s ~/moodle-plugin-skill/plugins/moodle-plugin/skills/moodle-plugin ~/.claude/skills/moodle-plugin
```

### Contents

| File | Content |
|---|---|
| [`SKILL.md`](plugins/moodle-plugin/skills/moodle-plugin/SKILL.md) | When to use it, plugin standard, repository layout, local validation, documentation, release |
| [`references/review-checklist.md`](plugins/moodle-plugin/skills/moodle-plugin/references/review-checklist.md) | Marketplace review checklist |
| [`references/act.md`](plugins/moodle-plugin/skills/moodle-plugin/references/act.md) | `act` recipe, container shortcuts, legacy JS conversion, throwaway Moodle |
| [`references/release-marketplace.md`](plugins/moodle-plugin/skills/moodle-plugin/references/release-marketplace.md) | Release, Marketplace upload, plugin page, resubmission |

### Contributing

Found something the reviewer flags that is not here, or a step that changed? Open an issue or a pull request with the evidence (the reviewer's text, the CI log or the Moodle docs link). Validate the plugin with `claude plugin validate ./ --strict` and `claude plugin validate ./plugins/moodle-plugin --strict`.

## Português (Brasil)

Uma [Agent Skill](https://agentskills.io) para criar, revisar, documentar e publicar **plugins Moodle que passam na revisão do Moodle Marketplace**. Vem empacotada como plugin do Claude Code, e a pasta da skill também funciona em outros agentes que leem `SKILL.md`.

Ela reúne o que funcionou em plugins aprovados ([EduPlay Moodle Suite](https://github.com/eduplay-moodle-suite)) e num plugin corrigido depois da revisão ([mod_imagemap](https://github.com/moodle-by-kelsoncm/mod_imagemap)): padrão do plugin, checklist do revisor, validação local do `moodle-plugin-ci` com `act`, documentação Sphinx, release e página no Marketplace.

Instalação no Claude Code:

```text
/plugin marketplace add moodle-by-kelsoncm/moodle-plugin-skill
/plugin install moodle-plugin@moodle-by-kelsoncm
```

A skill é carregada sozinha quando a tarefa envolve um plugin Moodle. Em outros agentes, copie ou faça um link de `plugins/moodle-plugin/skills/moodle-plugin` na pasta de skills do agente. A skill está em inglês, para servir à comunidade Moodle; o agente responde no idioma da conversa.

## License / Licença

MIT. See [LICENSE](LICENSE).
