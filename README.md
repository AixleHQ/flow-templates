# Aixle Flow templates

Ready-made connectors, boards, workflows and whole projects for
[Aixle Flow](https://github.com/AixleHQ/flow). Every Flow installation — SaaS,
self-hosted, AWS Marketplace — mirrors this repository and shows its templates
at `/templates`. Installing one copies it into a project; secrets never travel
in a template.

## Layout

```
namespaces.yaml   publishers: who may publish under which namespace
templates/<namespace>/<slug>/
  template.yaml   the package — schema/template.v1.json is the contract
  README.md       what it does and who it is for (shown on the template page)
  SETUP.md        what to do after installing (required when it needs anything)
  …               files the package references (SKILL.md, snapshots, tool files, assets)
revoked.yaml      templates withdrawn from the catalog
```

A template's catalog id is `<namespace>/<slug>` — two publishers can each have a
`code-reviewer-agent`. Everything inside `template.yaml` is referenced by local
keys, never by database ids.

## Namespaces

Every template belongs to a publisher namespace registered in `namespaces.yaml`,
with the GitHub logins allowed to change it. CI refuses a pull request that
changes templates in a namespace its author does not own. To publish for the
first time, add your namespace entry in the same pull request as your first
template; the maintainers approve new namespaces and set `verified`.

## Publishing a template

The easy way is to let your agent do it: connect it to Flow's personal MCP
server and run the `publish_template` prompt on a project that works. It exports
the project, walks you through turning project-specific values into install
inputs, and opens the pull request from your fork.

By hand: add `templates/<namespace>/<slug>/`, run `bin/validate` (needs Docker), and open a
pull request against `main`. See [CONTRIBUTING.md](CONTRIBUTING.md).

## What a template may not contain

- secret values (declare them under `requires.secrets` — the installer asks for them)
- literal MCP header or env values (use `config_item:NAME` references)
- container images that are not pinned by digest (`image@sha256:…`)
- database ids, cards, comments or runs
