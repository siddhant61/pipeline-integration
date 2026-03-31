# pipeline-integration

`pipeline-integration` is the orchestration-only repository for the JWST demo chain.

It does not duplicate or contain the source code of the three pipeline repositories. Instead, it coordinates them through reusable GitHub Actions workflows and artifact handoffs:

1. `material-ingestion-pipeline` produces the canonical upstream handoff package
2. `content-research-pipeline` consumes the upstream package and emits the canonical downstream handoff package
3. `media-generation-pipeline` consumes the downstream package and emits stable output artifacts

This repository exists to provide:
- one browser-runnable entrypoint for the full chain
- one place to observe end-to-end workflow runs
- one place to collect handoff artifacts and chain-level logs
- one place to document the integration contract between repos

## Scope

This repository owns:
- orchestration workflows
- integration documentation
- chain-level validation notes
- release/run coordination

This repository does **not** own:
- ingestion logic
- research synthesis logic
- media generation logic
- shared contract evolution by itself

## Canonical demo

The first orchestrated demo chain is:

`JWST: telescope architecture, star formation, and the early universe`
