# GitOps repository example (environments as folders, not branches)

A minimal, correct layout for the GitOps repo that phase-4.md's step-by-step builds.
Structured per Codefresh/Octopus's "stop using branches for environments" guidance:
one `main` branch, one folder per environment, promotion by file copy.

```
base/                      # env-agnostic app definition (Component + Workload)
envs/
  development/             # each env is a FOLDER on main, never a branch
    releasebinding.yaml    # this env's slice of config (replicas, resources)
  staging/
    releasebinding.yaml
  production/
    releasebinding.yaml
platform/
  fleet-gitrepo.yaml       # applied once; makes Fleet sync this repo into Cluster 1
```

Mapping to the article's `base / variants / envs` model:

- `base/` == article's `base/` (common to all environments).
- `envs/<env>/` == article's `envs/` (per-environment leaf config).
- The article's `variants/` mixins (prod/non-prod, region, gpu) are omitted here
  because OpenChoreo's `ReleaseBinding.componentTypeEnvironmentConfigs` already
  carries per-environment sizing inline - add a `variants/` layer (and switch the
  envs to Kustomize overlays) only if you outgrow that, or for non-OpenChoreo apps.

Promotion (e.g. dev -> staging) is a file operation, never a branch merge:

```bash
# copy the promotable config forward, then adjust only what must differ
cp envs/development/releasebinding.yaml envs/staging/releasebinding.yaml
# edit metadata.name + spec.environment + sizing in the staging copy, commit, push
```

This example is illustrative and correct against the live CRD schemas, but has not
been synced through a live Fleet `GitRepo` (that needs a real GitHub repo). Applying
`base/` + one `envs/*/releasebinding.yaml` directly with `kubectl apply` against the
control plane deploys exactly as phase-3.md's greeter-service did.
