# AGENTS.md — layer-helm-chart

Standalone candy repo for the `helm-chart` layer — the `check-helm-vm` bed's
install leg: a real chart (`prometheus-pushgateway`) installed into a k3s cluster
via the external `helm-release` install step, after waiting in-venue for the node
to be Ready. The candy lives in `charly.yml` at the repo root. It carries **no
`skill:` entity**, so no owning `/charly-<family>:<name>` skill is projected into
the marketplace corpus.

Canonical files:

- `charly.yml` — the `helm-chart:` candy entity (no `skill:` entity present).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-kubernetes:helm` — the closest owning skill: the `step:helm-release`
  install step, the `helm:` check verb, the `helm_charts:` deploy field, and the
  `--enable-helm` apply path. Load before editing or troubleshooting the layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, per-distro `distro:` arms, package
  sections). Load before editing any entity field or plan step.

There is no dedicated `/charly-*:helm-chart` owning skill — this repo's candy
carries no `skill:` entity. The gap is recorded against
`opencharly/opencharly#291`; when one is authored, add it here.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` steps are the functional evidence: the `deploy`-context
  node-ready `run:` step, the `deploy`-context `helm-release:` install step, and
  the `runtime`-context `helm:` release-exists check. The `check-helm-vm` bed is
  the disposable R10 run.
- The `require:` on `plugin-helm/candy/plugin-helm` puts the helm provider in the
  deploy's scanned candy set, so `loadProjectPlugins` builds and connects it
  before the install timeline dispatches the step.

## Modify this repo

- Edit the `helm-chart:` candy entity in `charly.yml`.
- Keep the mutating install in this candy's `run:` steps — the bed's own plan is
  verify-only and does not lower `run:` steps during `fleet add`.
- The node-ready wait uses `kubectl wait` (bounded `sleep 1` polling of
  `/readyz`), never a bare sleep — keep it that way.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
