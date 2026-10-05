# Deploy signals

How to find a repo's pipeline, and the read-only command that answers "is commit X on environment Y". All commands only read. `gh api` switches to POST when given `-f`/`-F`/`--input`, so every call here uses `-X GET` with filters in the query string.

## Contents
- [Finding the pipeline](#finding-the-pipeline)
- [Is my commit deployed](#is-my-commit-deployed)
- [False negatives](#false-negatives)

## Finding the pipeline

| Signal in the repo | Pipeline |
|---|---|
| `.github/workflows/*.yml` with `environment:` on a job | GitHub Actions, which records GitHub deployments (unless `deployment: false`) |
| `.gitlab-ci.yml` with `environment:` | GitLab environments |
| `apiVersion: argoproj.io/v1alpha1`, `kind: Application`/`ApplicationSet` | Argo CD |
| `Chart.yaml`, `helmfile.yaml`, `values-<env>.yaml` | Helm |
| `kustomization.yaml` with a Flux `Kustomization` | Flux |
| `dinghyfile` | Spinnaker (Armory pipelines-as-code) |
| `task-definition.json`, `ecs-params.yml`, `copilot/` | AWS ECS |
| `vercel.json`, `netlify.toml`, `fly.toml`, `Procfile` + Heroku remote | Platform-managed deploys |
| `fastlane/Fastfile`, `eas.json` | Mobile builds (TestFlight, EAS channels) |
| None of these | Deploys live elsewhere (another repo, an internal tool). Ask the user where. |

Also check the telemetry side: `DD_VERSION`, `DD_GIT_COMMIT_SHA`, or a `version`/`git.commit.sha` tag means spans in an environment carry the deployed commit.

## Is my commit deployed

Resolve the commit first: for a merged PR, use `gh pr view <n> --json mergeCommit,state`. Squash and rebase merges put a different SHA on the environment than the branch head.

| Pipeline | Command | Read it as |
|---|---|---|
| GitHub deployments | `gh api -X GET "repos/{o}/{r}/deployments?environment=<env>&per_page=5"` then `gh api -X GET repos/{o}/{r}/deployments/<id>/statuses` | The newest `success` status's deployment `sha` is what runs there |
| Any, given a deployed SHA | `gh api -X GET repos/{o}/{r}/compare/<deployed>...<mine> --jq .status` | `behind` or `identical`: included. `ahead` or `diverged`: not yet |
| Local history | `git merge-base --is-ancestor <mine> <deployed>` | Exit 0: included |
| Actions runs | `gh run list --commit <sha> --json workflowName,status,conclusion,url` | Did the deploy workflow run and succeed for this commit |
| Argo CD | `argocd app get <app> -o json` → `.status.sync.revision`; `argocd app history <app>` | Never add `--refresh` or `--hard-refresh`; those trigger a refresh |
| Helm | `helm history <release> -o json` | Revision, status, app version |
| Kubernetes | `kubectl rollout status deployment/<name> --watch=false`; image tag via `kubectl get deploy <name> -o jsonpath='{..image}'` | Only `get`/`describe`/`rollout status`, never `exec`, `apply`, `delete`, `rollout restart` |
| Flux | `flux get kustomizations` | Applied revision per kustomization |
| Spinnaker | `spin pipeline execution list --pipeline-id <id> --succeeded` | Never `save`, `execute`, `delete` |
| Datadog | `search_datadog_events` for deploy events; spans filtered by `version`/`git.commit.sha` | The commit is running, not just deployed |

## False negatives

- `deployment: false` on a workflow job, or deploys run outside Actions: no GitHub deployment record, yet the code shipped.
- Image-tag deploys: the environment names a tag, not a commit. Map the tag to a commit through the build workflow or the registry labels.
- Monorepos: each service has its own pipeline, so check the service the change touches.
- Deployed is not exercised. A feature flag or canary can keep new code idle. Check both separately.
