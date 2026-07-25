# knime-runner

Headless KNIME batch-executor as a Kubernetes `CronJob`-used-as-a-template
(never fires on its own — `schedule` is a deliberately impossible date).
Analogous to an ECS Fargate task: prepared image + parameters at run time,
runs to completion, results land on a shared PVC.

## One-time setup before `helm install`

1. Build and push the image (see `CServinL/knime-workflows/runner/Dockerfile`
   — needs a `KNIME_URL` build-arg, get it from
   https://www.knime.com/downloads/download-knime, gated behind a form so
   it can't be automated):
   ```
   docker build --build-arg KNIME_URL=<...> -t ghcr.io/cservinl/knime-runner:latest \
     CServinL/knime-workflows/runner
   docker push ghcr.io/cservinl/knime-runner:latest
   ```
2. Create the read-only deploy-key Secret (the key itself is a GitHub deploy
   key on `CServinL/knime-workflows`, read-only, not the user's personal
   SSH key — never commit the private key):
   ```
   kubectl create secret generic knime-workflows-deploy-key \
     --from-file=id_ed25519=<path-to-private-key> -n <namespace>
   ```
3. `helm install`/`helm upgrade` with `workflow.name` set to a folder that
   exists in `knime-workflows` (matches the export convention in that repo's
   README).

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
