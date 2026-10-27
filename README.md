# thermalwatch-live

Public *schedule-only* repository for [ThermalWatch](https://sih26ekaant.web.app).

This repository exists to hold two GitHub Actions schedules — nothing else. It
contains no application code, no data, no model and no configuration beyond the
two workflow files in `.github/workflows/`:

| Workflow | Cadence | What it does |
|---|---|---|
| `live-cycle.yml` | every ~22 minutes | checks out the private core repository, runs the ingestion → classification → archive → deploy cycle, destroys the runner afterwards. Each run dispatches its successor in its first seconds and paces that successor off the previous cycle's own start time |
| `uptime-ping.yml` | hourly | probes the site's `/api/health`, and fails when either the site stops answering, no successful `Live cycle` run has finished in 90 minutes (the chain stopped), or the served cycle is more than 12 h old (publishing stalled). A quiet satellite feed delays the deploy without breaking the loop, so liveness is judged from the run history rather than from the published timestamp |

Why a wrapper: a public repository gets unlimited GitHub Actions minutes, and
a 30-minute cadence would otherwise cost ~16,700 minutes/month on a private
repository. The core repository stays private; its contents never appear here.

## Required configuration (Settings → Secrets and variables → Actions)

**Secrets**

- `CORE_PAT` — fine-grained personal access token scoped to the core repo only
  (*Contents: read & write*), used to check it out and to read/write the
  rolling-state asset on its `live-artifacts` release.
- `LIVE_ENV` — the core project's `.env` (all provider API keys).
- `FIREBASE_SERVICE_ACCOUNT` — service-account JSON with
  `roles/firebasehosting.admin` on the Firebase project.

**Variables**

- `CORE_REPO` — the core repository, e.g. `Vex-15/SIH26`.

## Safety rails

`live-cycle.yml` refuses to run if this repository is ever made private without
the `ALLOW_PRIVATE_PAID` variable being set, because the ~22-minute cadence
would then exceed the 2,000 free minutes/month and start a bill.

Full setup notes live in the core repository's `docs/CI_SETUP.md`.
