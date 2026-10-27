# gha-timeout-cancelled-repro

Reproduces how GitHub Actions treats a job that is canceled because it exceeded `timeout-minutes`, as seen from the jobs that depend on it.

Each workflow runs on `workflow_dispatch` and has one upstream job followed by four probe jobs that `need` it:

| Probe job | `if` |
|---|---|
| `if-not-cancelled-and-not-failure` | `${{ !cancelled() && !failure() }}` |
| `if-cancelled` | `${{ cancelled() }}` |
| `if-failure` | `${{ failure() }}` |
| `report` | `${{ always() }}`, prints `needs.<job_id>.result` (and outputs) to the job summary |

| Workflow | Upstream job |
|---|---|
| [`job-timeout.yml`](.github/workflows/job-timeout.yml) | One job that exceeds `timeout-minutes: 1` |
| [`matrix-timeout.yml`](.github/workflows/matrix-timeout.yml) | A two-job matrix: `fast` succeeds, `slow` exceeds `timeout-minutes: 1` on the first attempt only. Each matrix job sets an output named after itself |
| [`job-failure.yml`](.github/workflows/job-failure.yml) | Control: one job that fails with `exit 1` |
