# keep-supabase-alive

A minimal GitHub Actions workflow that prevents your Supabase free-tier project from being paused due to inactivity.

## How it works

Supabase pauses free-tier projects after **7 days of inactivity**. This workflow runs on a schedule twice a week and
sends a single read-only HTTP request to your database, just enough to register activity and keep the project awake.

No dependencies. No extra tables. Just `curl`.

## Setup

### 1. Copy the workflow file

Copy `.github/workflows/keep-supabase-alive.yml` into your project's repository at the same path.

### 2. Add GitHub Actions variables and secrets

In your repository go to **Settings → Secrets and variables → Actions** and add:

**`SUPABASE_TABLE_NAME`** (under the *Variables* tab — not sensitive, safe to store in plain text)

Go to `https://github.com/<your-username>/<your-repo>/settings/variables/actions` and add a new variable:

| Variable name | Value |
|---|---|
| `SUPABASE_TABLE_NAME` | Any table that exists in your project, e.g. `users` |

**Secrets** (under the *Secrets* tab — add these as **repository secrets**, not environment secrets):

**`SUPABASE_URL`**

Go to your project overview at `https://supabase.com/dashboard/project/<your-project-id>`. The URL is displayed on that
page and can be copied directly. It looks like:

```
https://xxxxxxxxxxxxxxxxxxxx.supabase.co
```

**`SUPABASE_SECRET_KEY`**

Go to `https://supabase.com/dashboard/project/<your-project-id>/settings/api-keys` and find the **Secret keys** section.
Click the key to reveal it, then copy it. It looks like:

```
sb_secret_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

> Use the secret key (not the publishable key), it bypasses Row Level Security so the ping works regardless of your
> table policies.

### 3. Push and test

Push the workflow file to your repository. To verify it works immediately:

1. Go to **Actions → Keep Supabase Alive**
2. Click **Run workflow**
3. Confirm the step exits successfully — you should see a JSON response (even an empty `[]` means the ping worked)

## What's built in for safety

| Protection | Detail |
|---|---|
| `timeout-minutes: 5` | Kills the entire job if it hangs no runaway GitHub Actions minutes |
| `curl --max-time 10` | Cuts the HTTP request after 10 seconds before the job timeout kicks in |
| `curl --fail` | Exits non-zero on 4xx/5xx a misconfigured key or bad URL fails loudly instead of silently succeeding |
| `curl --retry 2` | Retries twice on transient network errors so you don't get false failure alerts |
| `permissions: {}` | Locks the `GITHUB_TOKEN` to zero permissions this job doesn't touch the repo |
| Secret presence check | Fails immediately with a clear message if either secret is missing, before curl runs |

## Schedule

By default, the workflow runs at **00:00 UTC every Sunday and Friday**, twice a week, well within the 7-day inactivity
window.

To change the schedule, edit the cron expression in the workflow file:

```yaml
- cron: '0 0 * * 0,5'  # Sunday=0, Friday=5
```

A [cron expression editor](https://crontab.guru) can help if you want a different cadence.

## Why not [supabase-inactive-fix](https://github.com/travisvn/supabase-inactive-fix)?

That project is great if you need to manage **multiple Supabase projects** from a single workflow. It supports a config
file, multiple keys, detailed logging, and write operations.

If you only have one project and want something you can drop in and forget about, this is the simpler alternative.

## License

MIT
