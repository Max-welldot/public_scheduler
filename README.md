# AcquiProp public scheduler

This repository contains only a small GitHub Actions workflow. It does not contain AcquiProp application source code, database credentials, connected social account tokens, API keys, Django settings, or the scheduler secret.

The workflow calls the existing private AcquiProp endpoint every 30 minutes:

`https://acquiprop.com/campaigns/internal/automation/run/`

## Setup

1. Create a new **public** GitHub repository for this scheduler.
2. Extract this folder and push `.github/workflows/acquiprop-scheduler.yml` and this README to the repository's default branch.
3. Generate a long random secret locally:

   ```bash
   python -c "import secrets; print(secrets.token_urlsafe(48))"
   ```

4. Add the exact same value in two places:
   - On the existing AcquiProp Render web service, add the environment variable `ACQUIPROP_SCHEDULER_SECRET` and redeploy or restart the service.
   - In this public GitHub repository, go to **Settings → Secrets and variables → Actions → New repository secret**. Name it `ACQUIPROP_SCHEDULER_SECRET`.
5. Open **Actions → AcquiProp scheduled publishing → Run workflow** to test the connection.

Never put the secret in this repository, in a normal repository variable, in a commit, or in a screenshot. The workflow reads it only from GitHub Actions Secrets and sends it in the HTTPS Authorization header.

## Schedule and behavior

The schedule `7,37 * * * *` runs at minute 7 and minute 37 of each hour (UTC). GitHub scheduled starts can be delayed. AcquiProp's Render Free web service may also need about a minute to wake after idle time, so this setup does not promise exact-minute publishing.

The workflow uses a standard GitHub-hosted Ubuntu runner and does not check out source code or install application dependencies. GitHub documents standard hosted runner usage in public repositories as free. Render usage is separate: this design reuses the existing web service and does not create a Render Cron Job or another service. Render grants 750 Free instance-hours per workspace per calendar month, and a Free web service stays active until 15 minutes after its last inbound request. Two scheduler requests per hour can therefore keep an otherwise-idle AcquiProp web service active for roughly 30 minutes per hour, or about 360 instance-hours in a 30-day month. Real app traffic can keep it active for longer, and the hours are shared with other Free services in the workspace. Check Render's monthly usage before enabling the workflow.

If the AcquiProp domain changes, update the `endpoint` value in the workflow. Keep this repository limited to the workflow and non-secret setup notes.
