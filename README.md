# helm-chart

The `check-helm-vm` bed's install leg — a real chart
(`prometheus-pushgateway`) installed into a k3s cluster via the external
`helm-release` install step.

`helm-chart` installs a chart into the k3s cluster after waiting in-venue for the
node to be Ready. This is the `check-helm-vm` bed's install leg: the bed's own
plan runs in verify-only mode (mutating steps skipped) and `fleet add` lowers
only candy plans' `run:` steps, so the mutating install must live in a candy
whose `run:` steps execute during `fleet add` — never in the bed's own plan. The
`k3s-server` candy's node-ready check is a `check:` step that does not run during
`fleet add`, so this candy waits for the node itself before the
`helm upgrade --install --wait` can schedule the chart's pods: it polls the
apiserver's `/readyz` in a bounded `until` loop (`sleep 1` cadence, 300s
deadline) until the kubeconfig's API server answers, then runs
`kubectl wait --for=condition=Ready node --all`.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `helm-chart` |
| Requires | `vm-k3s-server`, `plugin-helm/candy/plugin-helm` |
| Install step | `step:helm-release` (chart `prometheus-pushgateway`, release `web-pushgateway`, namespace `web`) |
| Service / port | none |

## How to use it

Compose the layer into an image: the image node names this repo in its inner
`candy:` list (the outer `candy:` key holds the image spec — `base:` plus the
list of candies):

```yaml
version: 2026.261.1747
my-k8s-box:
  candy:
    base: cachyos
    candy:
      - '@github.com/opencharly/layer-helm-chart:v2026.240.0119'
```

Then, inside the built guest:

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get pods -n web
helm list -n web
```

## Layout

- `charly.yml` — the `helm-chart:` candy entity: the `require:` deps, the
  node-ready `run:` step, the `helm-release:` install step, and the `helm:`
  `check:` step.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Family skill: `/charly-kubernetes:helm` — the `step:helm-release` install step
  and the `helm:` check verb this candy exercises (this candy carries no
  `skill:` entity of its own)
- `/charly-kubernetes:check-k8s` — the `kube:` check verb
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
