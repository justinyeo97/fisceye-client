# FiscEye

**Live website:** https://fisceye.vercel.app

**A finance and performance workspace for sales agencies operating across countries.**

FiscEye brings consultant salaries, operating expenses and placement revenue together so agencies can understand individual contribution, compare country performance and report to headquarters in their company's home currency.

## Screenshots

> ![FiscEye](image.png)

## Core features

- **ROI reporting:** Compare consultant, country and company performance by period.
- **Groq ROI analysis:** Generate an on-demand summary, observations, suggested investigations and limitations, with expandable supporting figures. Available to administrators, Finance and viewers when the API is configured.
- **Readable financial results:** Positive profit and ROI appear green, negative values red, and zero/unavailable values neutral. Supporting figures use comma separators while preserving decimal precision.
- **Light and dark mode:** Switch themes from the top-right control, including on sign-in. The initial theme follows the system preference; an explicit choice is saved in browser storage.
- **CSV exports:** Download filtered ROI, country allocations and conversion reconciliation.
- **Audit log:** Admins and Finance can review who changed records, including deletions.
- **Company creation:** Verified users without a company can create a workspace as its admin.
- **Automatic conversion:** Allocate expenses in regional currencies using historical rates.
- **Scheduled retries:** The backend can retry pending conversions with durable leases and backoff.
- **Headquarters consolidation:** Report revenue and costs in the company's home currency.
- **Consultant management:** Track roles, salaries, employment and country assignments.
- **Expense entry and imports:** Add expenses individually or upload CSV files.
- **Expense and placement deletion:** Admins and Finance can delete records with confirmation; expense conversions are removed too.
- **Invited access:** Let invited consultants view only their own expenses.
- **Company isolation:** Protect company records with membership-based permissions.

## Explore the pages

| Page                         | What it helps users do                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Dashboard**                | Review spending, conversion status and recent expenses at a glance.                                     |
| **Expenses / My expenses**   | Manage expenses and imports, including permitted deletion, or review personal expenses as a consultant. |
| **Consultants**              | Maintain the team, compensation, country assignments and login invitations.                             |
| **Countries**                | Configure regional reporting currencies.                                                                |
| **Placements**               | Add, edit or delete placement revenue records with the appropriate permissions.                         |
| **Conversions & exceptions** | Review pending historical conversions and retry unavailable rates.                                      |
| **Reports & reconciliation** | Compare ROI, request Groq explanations with supporting figures, and inspect allocations and conversions. |
| **My account**               | View the user profile, select an invited company and manage permitted company settings.                 |

## How ROI works

```text
Total costs = Operating expenses excluding marked payroll + Accrued base salary
Profit contribution = Recognized revenue - Total costs
ROI (%) = Profit contribution / Total costs x 100
```

Revenue follows its recognition date; operating expenses follow transaction dates. Salary follows effective-dated compensation, employment dates and country assignments, with partial periods prorated by calendar days. Payroll expenses already represented in salary history must be marked to avoid double counting.

Reports identify missing revenue, salary coverage or exchange rates and withhold incomplete profit/ROI. Zero-cost ROI is undefined. Results are live calculations based on recorded costs, rather than closed accounting statements.

## AI ROI analysis

On Reports, choose the ROI grouping and period, wait for the table to load, then click **Analyze ROI**. Analysis has its own loading and error states, so the table remains usable while generation runs or if the provider is unavailable.

The frontend sends the selected filters and report snapshot to `POST /reports/roi/analysis`. The API checks permissions, recomputes the report, checks for stale data, anonymizes consultant identities and calls Groq. The response contains a summary, observations, suggested investigations, limitations and expandable **Supporting figures** from backend calculations. Explicit date selections can include the preceding period; all-history views do not invent a previous period. AI interpretations still require review.

The backend caches matching analyses for thirty minutes and applies shared request/token budgets, a completion cap and provider-limit cooldowns. There are no automatic generation retries or paid-provider fallbacks. Cache and budget state is per API process and resets on restart. Keep the Groq account on its Free plan to maintain the free-usage goal; the application cannot verify the account's billing plan.

All selected rows contribute to totals, while detailed AI evidence is limited to the latest twelve periods and twelve groups by revenue. Consultant names/IDs and individual salary histories are omitted from the provider request; an anonymous consultant's aggregate costs can still concern one person.

## Tech stack

| Layer          | Technology                                                           |
| -------------- | -------------------------------------------------------------------- |
| Frontend       | React 19, JavaScript (JSX), Vite 7                                   |
| Routing        | React Router 7                                                       |
| Styling        | CSS, responsive layouts, React Bits VariableProximity heading effect |
| Authentication | Supabase Auth with email/password and Google sign-in                 |
| Database       | Supabase PostgreSQL, row-level security and effective-dated records  |
| Backend        | Node.js, Express, Zod validation and Supabase JavaScript client      |
| API hosting    | Railway                                                              |
| Currency data  | Historical exchange-rate providers with fallback handling            |
| AI analysis    | Groq Chat Completions through the Express backend; validated structured output |
| Verification   | Vitest, Vite production build and GitHub Actions                      |

This repository contains the frontend. The companion backend is maintained in [FiscEye API](https://github.com/justinyeo97/fisceye-api).

## Run locally

Use Node.js 24, matching CI. For a first-time checkout:

```powershell
npm ci
if (!(Test-Path .env.local)) { Copy-Item .env.example .env.local }
```

Configure these values in `.env.local`:

| Variable | Purpose |
| --- | --- |
| `VITE_API_BASE_URL` | Express base URL including `/api/v1`, for example `http://localhost:3000/api/v1`. The example file points to the Railway deployment. |
| `VITE_SUPABASE_URL` | Supabase project URL used for authentication. Use the same project as the API. |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Supabase publishable key. `VITE_SUPABASE_ANON_KEY` is also accepted for legacy projects. |

```powershell
npm run dev
```

Open the URL printed by Vite. To use a local backend, start the companion API separately and set its allowed browser origin to the Vite origin. Restart Vite after changing environment variables. Google sign-in also requires the frontend callback URL to be configured with the authentication provider.

`VITE_*` values are included in the browser build. Keep `GROQ_API_KEY` and any Supabase service-role key exclusively in the backend environment. The website does not have an API-key entry form.

## Build, test and deploy

```powershell
npm test
npm run build
npm run preview
```

`build` writes the production assets to `dist`; `preview` serves that build locally. If PowerShell blocks the npm script wrapper, use `npm.cmd` in place of `npm`. Frontend CI runs `npm ci`, `npm test` and `npm run build` with placeholder public configuration; automated checks do not require a live Groq key.

For frontend hosting, configure the three `VITE_*` variables above before building. Use the deployed API base URL and configure its CORS origins for the website. Changing frontend environment variables requires a new frontend build.

For Groq analysis on the hosted site, configure `GROQ_API_KEY` on the **Railway backend service**, then apply the variable changes with a backend deployment. A key saved only in the API's local `.env` is not uploaded by Git and does not configure Railway. Refresh Reports afterward. The default model is `openai/gpt-oss-20b`, served by Groq.

Troubleshooting:

- **AI analysis is not configured:** the API serving the report does not detect `GROQ_API_KEY`; check that backend's environment and restart/redeploy it.
- **The report has changed:** refresh ROI, then request analysis again.
- **Usage limit reached:** wait for the applicable minute/day limit to reset. The ROI table remains available.
- **Report too large:** select a narrower date period or consultant before requesting analysis.

## Documentation

- [Product and system architecture](ARCHITECTURE.md)
- [Website wireframe](docs/WIREFRAME.md) — page guide and editable Canva design.
- [Groq setup, data handling and usage controls](https://github.com/justinyeo97/fisceye-api/blob/main/docs/roi-analysis.md)
