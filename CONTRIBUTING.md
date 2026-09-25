# Contributing a template

1. **One template per pull request**, in `templates/<slug>/`.
2. **Run `bin/validate`** before pushing. CI runs the same check, plus a secret
   scan and a check that every tool image resolves by its digest.
3. **Explain it.** `README.md` says what the template does and who it is for;
   `SETUP.md` says what to do after installing it (which integration to connect,
   which secret to add, which repository to attach). `SETUP.md` is required when
   the template has any `requires`.
4. **Changing a template** means increasing its `version` by one. Installs are
   one-shot copies, so existing projects never change; the version tells people
   which one they reviewed and installed.
5. **Third-party code.** A template can run a container image and ship prompt
   text. Say so in the pull request, pin images by digest, and expect the
   maintainers to read both.

The maintainers review every pull request. After merge, the template appears in
every installation's catalog within the hour.
