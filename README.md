name: t2000 Job Watcher

on:
  schedule:
    - cron: "*/5 * * * *"
  workflow_dispatch:

jobs:
  watch:
    runs-on: ubuntu-latest

    steps:
      - name: Check t2000 public open jobs
        shell: bash
        run: |
          set -euo pipefail
          echo "Checking t2000 public open jobs..."
          echo "Time: $(date -u)"

          curl --fail --silent --show-error --location \
            "https://api.t2000.ai/v1/open-jobs?status=open&limit=100" \
            -o /tmp/open-jobs.json

          python - <<'PY'
          import json
          from pathlib import Path

          data = json.loads(Path('/tmp/open-jobs.json').read_text())
          jobs = data.get('openJobs', [])
          print(f"Open jobs found: {len(jobs)}")
          for job in jobs:
              print(
                  f"- {job.get('id')} | ${job.get('maxUsdc')} | "
                  f"{job.get('title')} | SLA {job.get('slaMinutes')} min"
              )
          PY
