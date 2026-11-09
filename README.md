# Invoice Register — Tracewell custom module

A single-page, printable register of posted invoices. It runs entirely in the browser
against the Tracewell REST API and uses the standard Keycard handoff from the Tracewell
sidebar — there is no backend and no sign-in form.

## Hosted copy

Published from the [`realbenpeters/invoicereports`](https://github.com/realbenpeters/invoicereports)
repository to **<https://realbenpeters.github.io/invoicereports/>**.

The site is plain static files. If GitHub Pages is not serving it yet, open the repository's
**Settings → Pages** and set **Source** to **Deploy from a branch**, branch **main**, folder
**/(root)**. The URL starts working after GitHub's first build (usually under a minute).

> **CORS requirement:** Tracewell's API only returns CORS headers for allowlisted origins.
> A request from `https://realbenpeters.github.io` is currently rejected, and unauthenticated
> requests from the documented localhost origins are rejected too. The page will show
> “Couldn’t load invoices” until the Tracewell platform team adds the serving origin
> (`https://realbenpeters.github.io`) to the API's CORS allowlist. Do not work around this
> with a proxy that forwards Keycards.

## What it shows

One row per posted, non-voided invoice:

| Column | Source |
| --- | --- |
| Date | `order_date` |
| Invoice # | `invoice_number` |
| Customer | `customer_name` (PO number is searchable and shown on hover) |
| Amount (no tax) | `order_total` minus the sum of the line-item `taxAmount` values |
| Tax | Sum of line-item `taxAmount` from the invoice detail |
| Total | `order_total` |

The footer totals the three amount columns. Line details are fetched from
`GET /api/operations/orders/:id` with limited concurrency and the totals fill in as they
arrive. If a detail request fails, the row is marked `unavailable`, the footer says how many
rows are excluded, and a Retry link appears — totals are never silently wrong.

## Filters

- **From / To** — inclusive date range on the invoice date (native date pickers).
- **Customer** — only customers that actually have invoices.
- **Search** — invoice number, customer name, or customer PO.
- **Quick ranges** — All time, This month, Last month, Last 30 days, This year.
- **Clear** — resets everything.

The Print button prints only the filtered rows, with a clean header (business name, filter
scope, timestamp), the table, and the totals row. Rows do not split across pages and the
column header repeats on each page.

Business name, currency and date format are read from `GET /api/settings/site` (falling back
to `/api/settings`), so the module is portable to other tenants.

## Authentication

The module never implements sign-in. Tracewell's sidebar opens it with `#k=<keycard>`; the
module stores that in `localStorage.module_keycard`, removes it from the URL bar, and sends
it as a Bearer token on every `/api/...` request. When a Keycard expires the page shows the
“open this module from Tracewell” gate again.

The Tracewell MCP tools are agent-facing. A static browser module cannot call the MCP
endpoint (and the module dev kit forbids it); this page uses the same underlying REST API
those tools wrap, authenticated by the signed-in user's Keycard.

## Register it in Tracewell

1. Sign in to Tracewell as a tenant admin.
2. Go to **Administration → Custom Modules → Add Module**.
3. Entry URL:
   - Hosted: `https://realbenpeters.github.io/invoicereports/`
   - Local: `http://localhost:5173/index.html`
4. Label: `Invoice Register`, icon: `🧾`, visible to at least `tenant_admin`.
5. Save, then click **Invoice Register** in the sidebar's **Modules** section.

## Run it locally

From this folder, on one of Tracewell's documented CORS ports (5000, 5173, or 4000):

```bash
python -m http.server 5173
```

Then open <http://localhost:5173/index.html>. Without a Keycard the page shows the gate; that
is expected. Opening it from Tracewell supplies the Keycard.

## Files

- `index.html` — the entire module (markup, styles, script).
- `module.json` — manifest used when registering the module.
- `README.md` — this guide.
