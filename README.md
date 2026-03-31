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
4. Optionally edit the topic input (defaults to the canonical JWST demo topic).
5. Click **Run workflow** to start.

The workflow executes four jobs in sequence:

| Job | Stage | Calls | Artifact produced |
|-----|-------|-------|-------------------|
| `ingest` | 1 – Material Ingestion | `material-ingestion-pipeline/.github/workflows/manual-validate-upstream.yml` | `jwst-upstream-handoff` |
| `research` | 2 – Content Research | `content-research-pipeline/.github/workflows/manual-build-downstream.yml` | `jwst-downstream-handoff` |
| `generate` | 3 – Media Generation | `media-generation-pipeline/.github/workflows/manual-generate-sceneplan.yml` | `scene-plan-outputs` |
| `summary` | 4 – Chain Summary | *(local job)* | Prints artifact inventory |

## Expected artifacts

After a successful run, three artifacts are available on the workflow run page:

| Artifact name | Source stage | Contents |
|---------------|-------------|----------|
| `jwst-upstream-handoff` | Stage 1 – Material Ingestion | Upstream handoff package (`integration_fixtures/jwst/upstream/`) |
| `jwst-downstream-handoff` | Stage 2 – Content Research | Downstream handoff package (`integration_fixtures/jwst/downstream/`) |
| `scene-plan-outputs` | Stage 3 – Media Generation | ScenePlan, MediaPackage, and RunManifest (`outputs/`) |

## Prerequisites

For the orchestration workflow to call into each pipeline repo, those repos must expose their manual workflows as **reusable workflows** by adding a `workflow_call` trigger alongside their existing `workflow_dispatch` trigger.

Each pipeline repo needs this change in its workflow file:

```yaml
on:
  workflow_dispatch: ...   # existing trigger (keep as-is)
  workflow_call:           # add this trigger
    inputs:
      topic:
        description: "Demo topic"
        required: false
        type: string
```

| Repository | Workflow file | Status |
|------------|--------------|--------|
| `siddhant61/material-ingestion-pipeline` | `manual-validate-upstream.yml` | Pending — needs `workflow_call` trigger added |
| `siddhant61/content-research-pipeline` | `manual-build-downstream.yml` | Pending — needs `workflow_call` trigger added |
| `siddhant61/media-generation-pipeline` | `manual-generate-sceneplan.yml` | Pending — needs `workflow_call` trigger added |

## Secrets

The orchestration workflow passes secrets to the called workflows using `secrets: inherit`. Ensure the following secrets are configured at the organization level or in this repository:

- `OPENAI_API_KEY` — used by content-research-pipeline
- `GOOGLE_API_KEY` — used by content-research-pipeline
- `GOOGLE_CSE_ID` — used by content-research-pipeline

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Workflow fails with "not found" on a called workflow | The target repo workflow does not have a `workflow_call` trigger | Add `workflow_call` trigger to the target workflow (see Prerequisites) |
| Artifact shows as MISSING in summary | The upstream stage did not produce the expected artifact | Check the logs for that stage's job |
| Secret errors in Stage 2 | API keys not configured | Add the required secrets to this repo or the organization |
| Permission denied on workflow call | Cross-repo workflow access not allowed | Ensure repos are in the same organization and workflow permissions allow cross-repo calls |

## Repository structure

```
pipeline-integration/
├── .github/
│   └── workflows/
│       └── run-jwst-chain.yml    # Orchestration workflow
└── README.md                     # This file
```
