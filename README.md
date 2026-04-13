# keep-supabase-alive

A minimal GitHub Actions workflow that prevents your Supabase free-tier project from being paused due to inactivity.

## How it works

Supabase pauses free-tier projects after **7 days of inactivity**. This workflow runs on a schedule twice a week and
sends a single read-only HTTP request to your database, just enough to register activity and keep the project awake.

No dependencies. No extra tables. Just `curl`.

## Setup

### 1. Copy the workflow file

Copy `.github/workflows/keep-supabase-alive.yml` into your project's repository at the same path.

### 2. Replace the table name

In the workflow file, replace `YOUR_TABLE_NAME` with any table that already exists in your Supabase project:

```yaml
curl -X GET "${{ secrets.SUPABASE_URL }}/rest/v1/YOUR_TABLE_NAME?select=id&limit=1" \
```

It only reads one row: no writes, no side effects.

### 3. Add GitHub Secrets

In your repository go to **Settings → Secrets and variables → Actions** and add:

| Secret name | Value |
|---|---|
| `SUPABASE_URL` | Your project URL, e.g. `https://xxxxxxxxxxxx.supabase.co` |
| `SUPABASE_KEY` | Your project's `anon` public key |

Both values are in your Supabase dashboard under **Project Settings → API**.

### 4. Push and test

Push the workflow file to your repository. To verify it works immediately:

1. Go to **Actions → Keep Supabase Alive**
2. Click **Run workflow**
3. Confirm the step exits successfully — you should see a JSON response (even an empty `[]` means the ping worked)

## Schedule

By default the workflow runs at **00:00 UTC every Sunday and Friday** — twice a week, well within the 7-day inactivity
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
