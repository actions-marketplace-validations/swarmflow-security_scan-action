# SwarmFlow Security Gate — GitHub Action

Fail your CI build when [SwarmFlow](https://swarmflow.one) finds code-security issues at or above a severity threshold. Drop it into any workflow to block merges on critical/high vulnerabilities.

> This action evaluates the latest SwarmFlow findings for your repository via the SwarmFlow API and fails the job when the gate doesn't pass.

## Quick start

1. In SwarmFlow → **Settings → API Keys**, create an API key (`swf_live_…`).
2. In your GitHub repo → **Settings → Secrets and variables → Actions**, add a secret named **`SWARMFLOW_API_KEY`** with that key.
3. Add this workflow as `.github/workflows/swarmflow.yml`:

```yaml
name: SwarmFlow Security Gate
on: [push, pull_request]

jobs:
  security-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: swarmflow-security/scan-action@v1
        with:
          api-key: ${{ secrets.SWARMFLOW_API_KEY }}
          threshold: high   # fail on high or critical
```

That's it. The job fails (red ❌) if any finding is at or above `threshold`, and passes (green ✅) otherwise. A summary table is added to the run.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | ✅ | — | Your SwarmFlow API key. Use a repo secret: `${{ secrets.SWARMFLOW_API_KEY }}`. |
| `threshold` | | `high` | Minimum severity that fails the build: `critical`, `high`, `medium`, `low`. |
| `repo-url` | | current repo | Repository URL to evaluate. Defaults to `${{ github.server_url }}/${{ github.repository }}`. |
| `api-url` | | `https://swarmflow-production.up.railway.app` | SwarmFlow API base URL. Override only for self-hosted/staging. |
| `fail-on-error` | | `true` | If `true`, a failed gate (or API error) fails the build. Set `false` to report only. |

## Outputs

| Output | Description |
|--------|-------------|
| `passed` | `true` if no findings at/above the threshold. |
| `critical` / `high` / `medium` / `low` | Finding counts by severity. |

### Using outputs

```yaml
      - uses: swarmflow-security/scan-action@v1
        id: swarmflow
        with:
          api-key: ${{ secrets.SWARMFLOW_API_KEY }}
          threshold: critical
          fail-on-error: false   # don't fail; just read results
      - name: Comment on result
        run: echo "Critical=${{ steps.swarmflow.outputs.critical }} High=${{ steps.swarmflow.outputs.high }}"
```

## Notes

- The gate evaluates your repository's current SwarmFlow findings. Connect the repo and run a scan in SwarmFlow first (or schedule scans) so there are findings to evaluate.
- `threshold: critical` blocks only on critical issues; `high` (default) blocks on high **and** critical, and so on.
- The action prints a summary table to the GitHub Actions run (Step Summary).

## License

MIT
