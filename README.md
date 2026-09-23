# AMS3 Project Reports

Generates developer contribution reports and a KPI dashboard for the AMS3
project, straight from GitHub — no local clone required. Commits, diff
stats, merge history, and PR review stats are read via the `gh` CLI; the
delivery plan is read from `docs/dev-tasks/*.csv` in the target repo.

## Prerequisites

- Python 3.8+
- [`gh`](https://cli.github.com/) CLI, authenticated (`gh auth login`) with
  access to the target repo

## Setup

```bash
python ams3_report.py --init
```

Writes `ams3_report_config.json` with a starter config. Edit it to set:

- `github_repo` — `owner/repo` slug (or pass `--repo` each run)
- `branch` — branch to report on
- `start_date` — first Monday to include
- `tasks_dir` — folder containing per-module delivery-plan CSVs
- `modules` — module name → path-prefix mapping used to attribute commits
- `people` — maps each engineer to their known emails / git names / GitHub
  logins, so commits and PR reviews from different identities roll up to
  one person
- `exclude` — identities to drop entirely (bots, shared/session accounts)
- `publish_command` — optional shell command run after `--publish`;
  `{file}` is replaced with the path to the generated report

## Usage

Find raw git identities to fill into the config's `people`/`exclude`:

```bash
python ams3_report.py --repo OWNER/REPO --discover
```

Build the contribution report:

```bash
python ams3_report.py --repo OWNER/REPO
```

Writes `reports/ams3-dev_contribution_report-latest.html` and a
date-stamped copy. Fetching per-commit diff stats via the GitHub API is the
slow part — expect a few minutes on a repo with hundreds of commits.

Build and publish in one step:

```bash
python ams3_report.py --repo OWNER/REPO --publish
```

Runs `publish_command` from the config against the freshly built report.

Write the bundled KPI dashboard template:

```bash
python ams3_report.py --init-kpi
```

Writes `ams3-developer-kpi-latest.html` to `output_dir`. This is a static
snapshot — it isn't regenerated from git/gh — so update its data by editing
the file directly.

If `--repo` is omitted, `github_repo` from the config is used. Report HTML
templates are bundled in `ams3_template.py` and rendered in-memory, so
building a report needs no network access besides `gh`.

## Output

Reports are written to `output_dir` (default `reports/`), which is
git-ignored — regenerate them rather than committing them.
