---
name: conventional-commits
description: Write commit messages in the Conventional Commits format. Use whenever you create a commit.
---

# Conventional Commits

Format: `<type>(<scope>): <subject>`

- **type** — one of `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `ci`, `style`, `perf`, `build`.
- **scope** — optional: the area touched (`auth`, `billing`, `ci`). Leave it out when it would repeat the subject.
- **subject** — imperative mood ("add", not "added"), no trailing period, under about 72 characters.
- A breaking change gets `!` after the type or scope (`feat(api)!: …`) and a `BREAKING CHANGE:` footer that says how to migrate.

## The body

Leave a blank line after the subject. Explain **why** the change is needed and anything a reviewer
would otherwise get wrong. The diff already shows what changed — do not narrate it.

## Examples

- `fix(auth): refresh the token before it expires, not after the first 401`
- `refactor(billing): compute invoices in one pass`
- `docs: explain how to run the suite against an isolated database`
