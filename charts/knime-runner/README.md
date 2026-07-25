# knime-runner

Headless KNIME batch-executor as a Kubernetes `CronJob`-used-as-a-template
(never fires on its own — `schedule` is a deliberately impossible date).
Analogous to an ECS Fargate task: prepared image + parameters at run time,
runs to completion, results land on a shared PVC.

## One-time setup before `helm install`

1. The image builds automatically via
   `CServinL/knime-workflows/.github/workflows/build-runner.yml` on every
   push to `runner/Dockerfile` (or manual dispatch), publishing to
   `ghcr.io/cservinl/knime-runner`. One manual step first: set the
   `KNIME_URL` repo variable (Settings > Secrets and variables > Actions >
   Variables) — get the current Linux tar.gz link from
   https://www.knime.com/downloads/download-knime, gated behind a form so
   it can't be scraped/automated. **The GHCR package is left public**
   (the image is just KNIME + a JRE, no workflow content baked in — those
   get git-cloned at run time — so there's nothing sensitive in the image
   itself, and it avoids needing an `imagePullSecret` on forge).
2. Create the token Secret (a GitHub Personal Access Token, used over HTTPS —
   never commit it, don't put it in `values.yaml`):
   ```
   kubectl create secret generic knime-workflows-token \
     --from-literal=token=<PAT> -n runners
   ```
   Live in namespace `runners`. Note: a PAT is account-scoped by GitHub's
   classic-token model, broader than a repo-only SSH deploy key would be —
   scope it as tightly as GitHub's UI allows for this token specifically.
3. `helm install`/`helm upgrade -n runners` with `workflow.name` set to a
   folder that exists in `knime-workflows` (matches the export convention in
   that repo's README).

## Triggering a run

```
kubectl create job --from=cronjob/knime-runner run-$(date +%s) -n <namespace>
```

This reuses the CronJob's pod template as-is (same `workflow.name` every
time). For a different workflow or `-workflow.variable` parameters per
invocation, build a standalone `Job` manifest from `templates/job.yaml`'s
`jobTemplate.spec` instead — `kubectl create --from=cronjob` doesn't support
per-invocation overrides. This is the piece OpenClaw/Node-RED would need to
generate programmatically once they're wired up to trigger this.

Results land on the `<release>-results` PVC, mounted at `/results` in the
container — workflows need their Writer nodes configured to write there.
