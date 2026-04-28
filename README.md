# Payments Refund API — Postman CSE Exercise (Service 1)

This repo onboards the **Payments Refund API** into Postman using CSE-maintained GitHub Actions. It creates a Postman catalog footprint (Spec Hub + generated suites + environments) and syncs exported artifacts back into this repo for platform ownership.

**Postman project/workspace name:** `cse-payment-refund-service-v1`

---

## What this workflow produces (end-to-end)

After a successful run you will have:

**In Postman**
- Workspace created/reused: `cse-payment-refund-service-v1`
- OpenAPI spec uploaded to **Spec Hub**
- Generated collections:
  - Baseline
  - Smoke
  - Contract
- Environments created/updated: `prod`, `stage`
- Operational assets:
  - Mock server
  - Smoke monitor

**In GitHub**
- Exported collections committed under `postman/collections/`
- `.postman/resources.yaml` mapping file committed (IDs + metadata)

---

## How the workflow works (decisions)

### Actions chain (explicit on purpose)
This workflow runs two production actions in order:

1) **postman-bootstrap-action**
- Creates/reuses the workspace
- Uploads spec to Spec Hub
- Generates Baseline/Smoke/Contract suites
- Creates/updates environments

2) **postman-repo-sync-action**
- Exports collections to `postman/collections/`
- Writes `.postman/resources.yaml`
- Creates mock + smoke monitor
- Commits changes back to the repo

I kept the chain explicit (instead of a composite orchestrator) to make the flow easy to explain in the demo/Q&A and easier for a platform team to maintain.

### Public repo spec fetch
Because the repo is public, the workflow can fetch the spec directly from GitHub via `raw.githubusercontent.com` (no local server workaround required).

### CI workflow generation disabled
`generate-ci-workflow: false` prevents automated edits to `.github/workflows` and keeps CI ownership with the platform team.

---

## Universal vs per-service changes

### Universal (same across services)
- Inputs: spec + env targets + auth/runtime variable contract
- Pipeline: Bootstrap → Repo Sync
- Outputs: Spec Hub + Baseline/Smoke/Contract + envs + exports + mock/monitor

### Changes per service
- Spec quality/drift
- Real runtime URLs per env
- Auth method (OAuth/JWT/API key/mTLS)
- Internal network requirements (may require in-network runners)
- Ownership/taxonomy/governance mappings

---

## What the customer platform team must provide

For a real enterprise rollout (AWS + mixed infra + GitHub Actions + one GitLab CI team):

- **Runtime URLs** (prod/stage) and gateway routing rules
- **Auth + secrets**: OAuth/JWT/API keys and rotation in a secret manager
- **mTLS/internal-only**: cert distribution + in-network runner placement
- **CI integration**:
  - GitHub Actions permissions
  - GitLab CI template to run Postman CLI + publish JUnit + gate merges
- **Governance**: workspace ownership model, naming, lifecycle/archiving, spec versioning policy

---

## Run instructions

### 1) Required secrets
Repo → Settings → Secrets and variables → Actions → Repository secrets:
- `POSTMAN_API_KEY`
- `POSTMAN_ACCESS_TOKEN`
- `GH_PAT` (classic PAT if org policy restricts `GITHUB_TOKEN`)

### 2) Run the workflow
GitHub → Actions → **Postman - Onboard Payments API** → Run workflow

---

## Validation

**In GitHub**
- `postman/collections/` populated
- `.postman/resources.yaml` exists
- commit like: `chore: sync Postman artifacts and metadata`

**In Postman**
Workspace `cse-payment-refund-service-v1` contains:
- Spec in Spec Hub
- Baseline/Smoke/Contract collections
- `prod` and `stage` environments
- Mock server + smoke monitor

---

## Trade-offs
- Contract coverage quality depends on spec accuracy; drift must be governed.
- Mocks/monitors help discovery and demos, but internal-only services still require in-network execution and real auth.
- Use a governed service account token with rotation for real customers (PAT here reflects org constraints).

---

## AI usage disclosure
AI accelerated YAML/README drafting and debugging. All outputs were validated by successful workflow runs and verifying created Postman assets + committed repo exports.
