# HOS08A — CI/CD with Azure Static Web Apps · Mission Plan

**Student:** Mohammad Ra'uf Naser Al Batayneh (`M7MDRAUF`)
**Course:** CS504 — Software Engineering, City University of Seattle
**Goal:** Deploy a static website to Azure Static Web Apps wired to a GitHub Actions CI/CD pipeline, prove the pipeline works by shipping a change through a `development` → `main` pull request, then tear the Azure resource down.

**Deliverable repos**
| Purpose | Repo |
|---|---|
| Source + pipeline | https://github.com/M7MDRAUF/HOS08 (public) |
| University submission | https://github.com/cityuseattle/cs504-hos08-M7MDRAUF (private) |

---

## Global Constraints

Values taken verbatim from the screenshots embedded in `HOS08A - CICD.docx`:

| Setting | Value |
|---|---|
| Subscription | Azure for Students Starter (the only subscription on this account) |
| Resource Group | `HOS08_group` (new) |
| Static Web App name | `HOS08` |
| Plan type | **Free: For hobby or personal projects** — any other tier incurs charges |
| Region (Functions/staging) | West US 2 — required: Central US is blocked by subscription policy |
| Deployment source | GitHub |
| Organization | `M7MDRAUF` |
| Repository | `HOS08` |
| Branch | `main` |
| Build preset | HTML |
| App location | `/` |
| Api location | *(empty)* |
| Output location | `/` |

**Authorship:** every commit is authored by the student only. No AI co-author trailers anywhere.
**Evidence:** every step gets a screenshot in `screenshots/`, numbered in execution order.

---

## Phase 0 — Recon & Workspace

- [x] Extract `HOS08A - CICD.docx` text and all 26 embedded screenshots
- [x] Read the Azure config screenshots to pin down exact field values
- [x] Verify tooling: git 2.55, gh 2.98 (authed as `M7MDRAUF`, scopes `repo`+`workflow`), Node 24.10
- [x] Confirm `M7MDRAUF/HOS08` exists and is empty; confirm university repo is reachable
- [x] Clone repo to `3COURSES/HOS08`, set git identity to the student

## Phase 1 — Create the Static Website (Doc section A)

- [x] **1.1** Create `index.html` containing `<h1>Hello<h1>`
- [x] **1.2** Add a `README.md` describing the project
- [x] **1.3** Commit and push to `main`
- [x] **1.4** Screenshot: repo on GitHub showing `index.html`

## Phase 2 — Create the Azure Static Web App (Doc section B, steps 1–13)

Driven through the already-authenticated Azure portal in the browser.

- [x] **2.1** Screenshot: Azure portal home (signed in)
- [x] **2.2** Search "Static" → open **Static Web Apps**
- [x] **2.3** Click **Create**; fill Project Details + Static Web App details per Global Constraints
- [x] **2.4** Screenshot: the filled Basics blade
- [x] **2.5** Sign in / authorize `Azure-App-Service-Static-Web-Apps` against GitHub
- [x] **2.6** Fill Deployment details: Organization / Repository / Branch
- [x] **2.7** Fill Build Details: preset HTML, app `/`, output `/`
- [x] **2.8** Screenshot: the filled Deployment configuration blade
- [x] **2.9** **Review + create** → screenshot validation page → **Create**
- [x] **2.10** Screenshot: "Your deployment is complete"

## Phase 3 — First Pipeline Run (Doc section B, steps 14–15)

- [x] **3.1** **Go to resource**; screenshot the Overview blade with the generated URL
- [x] **3.2** Confirm Azure committed `.github/workflows/azure-static-web-apps-*.yml` to `main`
- [x] **3.3** Screenshot: GitHub Actions run for the workflow file commit
- [x] **3.4** Open the live URL; screenshot the rendered `Hello` page

## Phase 4 — Exercise the CI/CD Pipeline (Doc section B, steps 16–22)

- [x] **4.1** `git pull` to fetch the workflow file Azure added
- [x] **4.2** `git checkout -b development`; screenshot the terminal
- [x] **4.3** Edit `index.html` to `<h1>Hello World!<h1>`; screenshot the diff
- [x] **4.4** `git add --all` / `git commit -m "development branch"` / `git push -f origin development`; screenshot terminal
- [x] **4.5** Screenshot: GitHub prompting **Compare & pull request**
- [x] **4.6** Open the PR — screenshot the Azure SWA preview-environment comment
- [x] **4.7** Merge the PR, confirm merge; screenshot
- [x] **4.8** Screenshot: **Actions** tab with all workflow runs green

## Phase 5 — Verify the Deployment (Doc section B, steps 23–24)

- [x] **5.1** Azure Portal → Static Web Apps → `HOS08`; screenshot
- [x] **5.2** Open the production URL; screenshot showing **Hello World!** — proof the pipeline shipped the change

## Phase 6 — Documentation Deliverable

- [x] **6.1** Assemble a Word document with every step and its screenshot
- [x] **6.2** Write `README.md` with pipeline architecture and a summary of what was learned
- [x] **6.3** Verify the `.docx` renders correctly (convert to PDF and inspect the pages)

## Phase 7 — Submit

- [x] **7.1** Push source, workflow, screenshots and docs to `M7MDRAUF/HOS08`
- [x] **7.2** Push the deliverable to `cityuseattle/cs504-hos08-M7MDRAUF`
- [x] **7.3** Verify no `Co-Authored-By` / AI trailer on any commit in either repo

## Phase 8 — Cleanup (Doc section B, steps 25–28)

- [x] **8.1** Student confirmed teardown may proceed
- [x] **8.2** Azure Portal → Static Web Apps → `HOS08` → **Delete** → **Yes**; screenshot 26
- [x] **8.3** Verified the Static Web App is gone — portal lists "No static web apps to display" (screenshot 27) and both the production and staging hostnames return HTTP 404
- [ ] **8.4** Delete the now-empty resource group `HOS08_group` — **not completed**, see below

### Open item: the empty `HOS08_group` resource group

The Static Web App itself is deleted, which is what steps 25–28 require and the only
part that could consume the subscription. The empty resource group that contained it
is still listed (screenshot 28).

Two portal delete attempts did not take effect. The first failed because the
confirmation textbox was filled programmatically, which did not trigger the blade's
validation, so the **Delete** button was still disabled when clicked. The second
attempt typed the name key-by-key, the button became enabled, and the click registered
— but the group still appears after several minutes and repeated reloads, and the
portal's notifications panel reported nothing at all. The resource-group row's context
menu offers no delete action, so there is no alternative portal route.

A plausible explanation, untested: the resource group's own location is **Central US**,
the same region this subscription's policy blocks (see the `RequestDisallowedByAzure`
failure in Phase 2). A policy denying operations in that region could be rejecting the
delete silently.

An empty resource group carries no cost. Deleting it needs either Azure CLI
(`az group delete -n HOS08_group --yes`, which surfaces the real error) or a retry from
the portal later.

---

## Risk Register

| Risk | Mitigation |
|---|---|
| Wrong plan tier → account charged | Plan tier is verified on the Review+Create blade before clicking Create |
| Azure resource left running after grading | Phase 8 teardown, gated on student confirmation |
| `git push -f` on `development` | Only ever targets `development`, never `main` |
| Azure's GitHub authorization not yet granted | Handled interactively in the portal during Phase 2.5 |
| AI attribution leaking into commits | Local `user.name`/`user.email` pinned to the student; trailer audit in Phase 7.3 |
