# dsh-subscriptions

Publishable DSH plugin. Scoped name `@goodandready/dsh-subscriptions` must match
in `package.json`, `cordis.patch.yml` -> `name:`, and `lib/client.js` loader id.

- This branch is a personal GitHub fork of upstream `v0.6.8`; retain the upstream MIT attribution.
- Git: `git-cursor`. No infra paths, IPs, or secrets in the tree.
- Spec: `docs/architecture/2026-08-20-dsh-subscriptions-design.md`
- Plan: `docs/plans/2026-08-20-dsh-subscriptions.md`
- Tests: `npm test`. After `file:` installs, remove then add so pnpm copies files.
- v1 has no tools and does not call `dsh-key-rotation`.
- OAuth client ids in Config defaults are vendor-public CLI values, overridable.
