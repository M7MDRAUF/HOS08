# HOS08 — CI/CD with Azure Static Web Apps

Static website deployed to **Azure Static Web Apps** through a **GitHub Actions** CI/CD
pipeline. Built for CS504 (Software Engineering) at City University of Seattle.

**Author:** Mohammad Ra'uf Naser Al Batayneh (`M7MDRAUF`)

> **The Azure resource has been deleted.** Steps 25–28 of the handout require tearing it
> down after the demonstration, so the live URL below no longer resolves. Sections 7 and 8
> of the report, and screenshots `24`–`28`, capture the site working and then removed.

| | |
|---|---|
| **Live site** (deleted) | https://ashy-mushroom-075b8701e.7.azurestaticapps.net |
| **Azure resource** | Static Web App `HOS08` in resource group `HOS08_group` |
| **Hosting plan** | Free |
| **Region** | West US 2 (Functions API and staging environments) |
| **Pipeline** | [`.github/workflows/azure-static-web-apps-ashy-mushroom-075b8701e.yml`](.github/workflows/azure-static-web-apps-ashy-mushroom-075b8701e.yml) |

---

## What this project demonstrates

A change committed on a feature branch reaches production without anyone touching a
server. The pipeline is the only path to production.

```
 development branch          pull request              main branch
        │                          │                        │
        │  git push                │  opens PR              │  merge
        ▼                          ▼                        ▼
  ┌───────────┐            ┌───────────────┐        ┌───────────────┐
  │  GitHub   │───────────▶│ GitHub Actions│───────▶│ GitHub Actions│
  │  (source) │            │ preview build │        │  prod build   │
  └───────────┘            └───────┬───────┘        └───────┬───────┘
                                   │                        │
                                   ▼                        ▼
                          Azure staging env         Azure production env
                          (temporary URL)           (public URL)
```

Azure generated the workflow file and committed it to `main` when the resource was
created; it also stored the deployment token as the repository secret
`AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_MUSHROOM_075B8701E`.

## The pipeline

The workflow has two jobs:

| Job | Trigger | What it does |
|---|---|---|
| `build_and_deploy_job` | push to `main`, or a PR opened/synchronized/reopened against `main` | Checks out the repo and runs `Azure/static-web-apps-deploy@v1` with `app_location: "/"` and `output_location: "/"`. A PR deploys to a temporary staging environment; a push to `main` deploys to production. |
| `close_pull_request_job` | PR closed | Tears down the staging environment created for that PR. |

## Verified pipeline runs

| # | Trigger | Branch | Result |
|---|---|---|---|
| 1 | Commit `4e2dfef` — Azure adds the workflow file | `main` | Success (51s) |
| 2 | PR #1 opened | `development` | Success (1m 8s) |
| 3 | PR #1 closed | `development` | Success (29s) |
| 4 | Merge commit `fd470f2` | `main` | Success (1m 1s) |

Run #4 is the proof: `index.html` changed from `<h1>Hello</h1>` to
`<h1>Hello World!</h1>` on a branch, and that change appeared on the public URL with no
manual deployment step.

## One issue worth recording

The first `Review + create` failed validation:

```
Resource 'HOS08' was disallowed by Azure: This policy maintains a set of best
available regions where your subscription can deploy resources.
(Code: RequestDisallowedByAzure, Target: HOS08)
```

This is **not** a plan-tier or quota problem, which is the obvious first guess. The
portal had defaulted the *Region for Azure Functions API and staging environments*
(on the **Advanced** tab) to **Central US**, and the `Azure for Students Starter`
subscription's region policy does not permit it. Setting that field to **West US 2** —
the value the assignment handout specifies — made validation pass immediately.

Screenshots [`11`](screenshots/11-azure-validation-failed-policy.png),
[`12`](screenshots/12-azure-advanced-region-westus2.png) and
[`13`](screenshots/13-azure-validation-passed.png) capture the failure, the fix, and
the passing validation.

## Teardown

The Static Web App was deleted after the demonstration, as the handout requires. Both the
production and staging hostnames return HTTP 404, and the portal's Static Web Apps blade
reports no resources.

One loose end: the now-empty `HOS08_group` resource group is still listed. Two portal
delete attempts had no effect and produced no notification, and the resource-group row
offers no delete action in its context menu. A likely but untested explanation is that the
resource group's own location is Central US — the region this subscription's policy
refuses — so the delete may be denied by the same policy. An empty resource group costs
nothing; removing it needs the Azure CLI, which reports the real error:

```bash
az group delete --name HOS08_group --yes
```

## Repository layout

```
├── .github/workflows/     Azure-generated GitHub Actions pipeline
├── docs/
│   ├── PLAN.md            Phase-by-phase execution plan for this assignment
│   ├── HOS08A-CICD-Submission.docx   Illustrated report (26 pages, 28 figures)
│   └── HOS08A-CICD-Submission.pdf    PDF rendering of the report
├── screenshots/           28 numbered screenshots, in execution order
├── index.html             The static site
└── README.md
```

## Reproducing this

```bash
git clone https://github.com/M7MDRAUF/HOS08.git
cd HOS08
git checkout -b development
# edit index.html
git add --all && git commit -m "your change"
git push origin development
# open a PR against main, then merge it — the pipeline does the rest
```
