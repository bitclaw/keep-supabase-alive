# keep-supabase-alive

A minimal GitHub Actions workflow that prevents your Supabase free-tier project from being paused due to inactivity — and automatically restores it if it does get paused.

## How it works

Supabase pauses free-tier projects after **7 days of inactivity**. This workflow:

1. **Checks the project status** via the Supabase Management API before every ping.
2. **Automatically restores the project** if it is already paused, then waits for it to come back online.
3. **Sends a database ping** to register activity and keep the project awake.

The schedule runs **daily**, well within the 7-day inactivity window and with no risk of a missed run slipping past the deadline.

## Setup

### 1. Copy the workflow file

Copy `.github/workflows/keep-supabase-alive.yml` into your project's repository at the same path.

### 2. Add required secrets and variables

Go to **Settings → Secrets and variables → Actions** in your repository.

#### Required secrets

| Secret | Where to find it |
|---|---|
| `SUPABASE_URL` | Project dashboard → project overview. Looks like `https://xxxxxxxxxxxxxxxxxxxx.supabase.co` |
| `SUPABASE_SECRET_KEY` | Project dashboard → Settings → API keys → Secret keys. Looks like `sb_secret_…` |

> Use the **secret key** (not the publishable key). It bypasses Row Level Security so the ping works regardless of your table policies.

#### Required variable

| Variable | Value |
|---|---|
| `SUPABASE_TABLE_NAME` | Any table that exists in your project, e.g. `users` |

#### Optional — auto-restore when already paused

Without these, the workflow can only *prevent* pausing. With them, it also *recovers* from a paused state automatically.

| Secret | Where to find it |
|---|---|
| `SUPABASE_ACCESS_TOKEN` | [supabase.com/dashboard/account/tokens](https://supabase.com/dashboard/account/tokens) — generate a personal access token |

| Variable | Value |
|---|---|
| `SUPABASE_PROJECT_REF` | Your project reference ID — the subdomain in your project URL, e.g. `ecubogrnsypaipdfsdny` |

### 3. Push and test

Push the workflow file to your repository. To verify it works immediately:

1. Go to **Actions → Keep Supabase Alive**
2. Click **Run workflow**
3. Confirm the ping step exits successfully — you should see a JSON response (even an empty `[]` means the ping worked)

## What's built in for safety

| Protection | Detail |
|---|---|
| `timeout-minutes: 10` | Kills the job if it hangs — increased from 5 to account for the 90-second restore wait |
| `curl --max-time 10` | Cuts the HTTP request after 10 seconds |
| `curl --fail` | Exits non-zero on 4xx/5xx — a misconfigured key or bad URL fails loudly instead of silently succeeding |
| `curl --retry 2` | Retries twice on transient network errors |
| `permissions: {}` | Locks the `GITHUB_TOKEN` to zero permissions — this job does not touch the repo |
| Secret presence check | Fails immediately with a clear message if required secrets are missing, before any curl runs |
| Graceful skip | Auto-restore step skips cleanly if the optional credentials are not configured |

## Schedule

The workflow runs **daily at 00:00 UTC**. This is intentionally more frequent than strictly necessary — GitHub Actions scheduled jobs can be delayed, and running daily ensures a missed or late run never puts the project at risk.

To change the schedule, edit the cron expression in the workflow file:

```yaml
- cron: '0 0 * * *'  # every day at midnight UTC
```

A [cron expression editor](https://crontab.guru) can help if you want a different cadence.

## Why not [supabase-inactive-fix](https://github.com/travisvn/supabase-inactive-fix)?

That project is great if you need to manage **multiple Supabase projects** from a single workflow. It supports a config file, multiple keys, detailed logging, and write operations (INSERT/DELETE).

This repo is a simpler drop-in for a single project, with the addition of auto-restore via the Management API — which neither approach had before.

## License

MIT
