# pipeline-integration

`pipeline-integration` is the orchestration-only repository for the JWST demo chain.

It does not duplicate or contain the source code of the three pipeline repositories. Instead, it coordinates them through reusable GitHub Actions workflows and artifact handoffs:

1. **material-ingestion-pipeline** — produces the canonical upstream handoff package
2. **content-research-pipeline** — consumes the upstream package and emits the canonical downstream handoff package
3. **media-generation-pipeline** — consumes the downstream package and emits stable output artifacts

This repository exists to provide:

- One browser-runnable entrypoint for the full chain
- One place to observe end-to-end workflow runs
- One place to collect handoff artifacts and chain-level logs
- One place to document the integration contract between repos

## Scope

This repository owns:

- Orchestration workflows
- Integration documentation
- Chain-level validation notes
- Release/run coordination

This repository does **not** own:

- Ingestion logic
- Research synthesis logic
- Media generation logic
- Shared contract evolution by itself

## Canonical demo topic

> JWST: telescope architecture, star formation, and the early universe

## Chain overview

```
┌─────────────────────────────┐
│  material-ingestion-pipeline│
│  manual-validate-upstream   │
│  ─────────────────────────  │
│  Artifact: jwst-upstream-   │
│            handoff          │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│  content-research-pipeline  │
│  manual-build-downstream    │
│  ─────────────────────────  │
│  Artifact: jwst-downstream- │
│            handoff          │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│  media-generation-pipeline  │
│  manual-generate-sceneplan  │
│  ─────────────────────────  │
│  Artifact: scene-plan-      │
│            outputs          │
└─────────────────────────────┘
```

## How to run

1. Go to the **Actions** tab in this repository.
2. Select the **Run JWST Chain** workflow in the left sidebar.
3. Click **Run workflow**.
4. Click **Run workflow** again to confirm.

The workflow executes four jobs in sequence:

| Job | Stage | Calls | Artifact produced |
|-----|-------|-------|-------------------|
| `ingest` | 1 – Material Ingestion | `material-ingestion-pipeline/.github/workflows/manual-validate-upstream.yml` | `jwst-upstream-handoff` |
| `research` | 2 – Content Research | `content-research-pipeline/.github/workflows/manual-build-downstream.yml` | `jwst-downstream-handoff` |
| `generate` | 3 – Media Generation | `media-generation-pipeline/.github/workflows/manual-generate-sceneplan.yml` | `scene-plan-outputs` |
| `summary` | 4 – Chain Summary | *(local job)* | Reports per-stage pass/fail and artifact inventory |

## Expected artifacts

After a successful run, three artifacts are available on the workflow run page:

| Artifact name | Source stage | Contents |
|---------------|-------------|----------|
| `jwst-upstream-handoff` | Stage 1 – Material Ingestion | Upstream handoff package (`integration_fixtures/jwst/upstream/`) |
| `jwst-downstream-handoff` | Stage 2 – Content Research | Downstream handoff package (`integration_fixtures/jwst/downstream/`) |
| `scene-plan-outputs` | Stage 3 – Media Generation | ScenePlan, MediaPackage, and RunManifest (`outputs/`) |

## Prerequisites

### 1. Reusable workflow triggers

Each pipeline repo must expose its manual workflow as a **reusable workflow** by including a `workflow_call` trigger alongside `workflow_dispatch`. All three repos already have this in place:

| Repository | Workflow file | `workflow_call` |
|------------|--------------|:---------------:|
| `siddhant61/material-ingestion-pipeline` | `manual-validate-upstream.yml` | ✅ Ready |
| `siddhant61/content-research-pipeline` | `manual-build-downstream.yml` | ✅ Ready |
| `siddhant61/media-generation-pipeline` | `manual-generate-sceneplan.yml` | ✅ Ready |

### 2. Cross-repo workflow access

GitHub restricts cross-repository reusable workflow calls by default. The three pipeline repositories must allow access from this orchestration repository.

**For each pipeline repo**, go to:

> **Settings → Actions → General → Access**

and select:

> **"Accessible from repositories in the 'siddhant61' user account"**

(or, if part of an organization, select **"Accessible from repositories in the organization"**).

Without this setting, the orchestration workflow will fail with a "not found" error when calling the reusable workflow.

### 3. Secrets

The orchestration workflow passes secrets to the called workflows using `secrets: inherit`. Ensure the following secrets are configured at the organization level or in this repository:

- `OPENAI_API_KEY` — used by content-research-pipeline
- `GOOGLE_API_KEY` — used by content-research-pipeline
- `GOOGLE_CSE_ID` — used by content-research-pipeline

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `error: <workflow> was not found` | Cross-repo access not enabled, or the target workflow lacks a `workflow_call` trigger | 1. Enable access in the target repo: **Settings → Actions → General → Access** (see Prerequisites §2). 2. Verify the workflow file has `workflow_call` under `on:`. |
| Artifact shows as MISSING in summary | The called stage did not produce the expected artifact | Check the logs for that stage's job |
| Secret errors in Stage 2 | API keys not configured | Add the required secrets to this repo or the organization |
| Stages 2/3 show "skipped" | An earlier stage failed, so dependent stages were skipped | Fix the failing stage first, then re-run |

## Repository structure

```
pipeline-integration/
├── .github/
│   └── workflows/
│       └── run-jwst-chain.yml    # Orchestration workflow
└── README.md                     # This file
```
