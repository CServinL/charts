# charts

Personal collection of Helm charts, published via GitHub Pages.

**Pages URL**: https://cservinl.github.io/charts

## Usage (CLI)

```
helm repo add cservinl https://cservinl.github.io/charts
helm repo update
helm search repo cservinl
```

## Usage (Rancher UI)

Apps → Repositories → Create:
- Name: `cservinl` (or your preferred alias)
- Index URL: `https://cservinl.github.io/charts`

Charts will then be browsable/installable under Apps → Charts.

## Charts

- **home-assistant** — minimal Home Assistant chart, no forced dependencies (no Postgres/CNPG requirement, no bundled zigbee2mqtt/nats).

## Releasing

Publishing is automated via `.github/workflows/release.yml` (`helm/chart-releaser-action`) — bump the chart's `version` in its `Chart.yaml` and push to `main`; it packages, creates a GitHub Release, and updates `index.yaml` on the `gh-pages` branch automatically.
