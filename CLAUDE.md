# CLAUDE.md

Guidance for AI assistants (Claude Code and others) working in this repository.

## What this is

**Lowe's Professional** — an Arabic, right‑to‑left (RTL) employee‑management web app: attendance / check‑in, payroll, leave requests, performance, announcements, and reporting. The UI is entirely in Arabic.

This repo holds a **single, self‑contained HTML file** with inline CSS and vanilla JavaScript. There is **no build step, no framework, and no package manager.** The backend is **Supabase** (Postgres + Auth/Storage), accessed directly from the browser via the Supabase JS SDK loaded from a CDN.

This is a **leaner variant** of the larger app maintained in the sibling `LOWES` repo (`index_v4.html`). Both point at the **same** Supabase project, but the two files are independent — changes here do not propagate to `LOWES` and vice versa.

## Product & business context

**Company:** LOWE'S Professional — a Turkey‑based skincare/beauty brand selling in **Syria (primary market 2026), Turkey, and the Gulf**. The app handles three currencies: **SYP, USD, TRY**.

**Where this file sits among the projects (same Supabase backend):**
- **`lowes-app-web` — the live production staff app** (React 18 + Vite + Supabase on Vercel). The current/canonical product.
- `LOWES/index_v4.html` — the full‑featured single‑file HTML app.
- **`lowes-app` (this repo)** — the leanest single‑file variant. Treat as **secondary/prototype** unless told otherwise.

**Roles** (`admin` / `manager` / `employee`, plus `media_buyer` in the larger apps) map to a real org with Syria and Turkey teams; accounts are **disabled, not deleted**, when staff leave.

**Brand vs. app colors:** official brand identity is **gold `#C9A646` + black + cream**; the app UIs use cream/navy/teal. Use the **gold brand identity** for anything customer‑facing (invoices, receipts, exports).

> ⚠️ Business source docs contain secrets (logins, ad‑account IDs, staff PINs). **Never commit those to any repo.**

## Repository layout

| Path | Purpose |
|------|---------|
| `lowes-app` | **The entire application.** ~1,250 lines of HTML/CSS/JS. Note: the file has **no `.html` extension** but it is an HTML document — open/serve it as HTML. This is the only source file. |

There is no `manifest.json`, service worker, or icons directory in this repo (unlike the `LOWES` repo, which is a full PWA).

## Architecture & conventions

### Single-file structure
`lowes-app` is organized top‑to‑bottom as:
1. `<head>` — meta tags, fonts, and CDN `<script>` tags.
2. A large `<style>` block — all CSS, driven by CSS custom properties (`:root`) with a `[data-theme="dark"]` override for dark mode. Theme color is cream (`#FBF3E2`); palette uses navy `#0f1f3d` and teal `#0d7377`.
3. The DOM — ~12 "screens", each a `<div class="screen" id="screen-...">`. One is visible at a time via the `.active` class (`.screen{display:none}` / `.screen.active{display:flex}`).
4. A `<script>` block — ~80 vanilla JS functions in global scope, no modules.

Screens present: `register`, `log`, `calendar`, `holidays`, `profile`, `pin`, `manager`, `admin`, `adm-emp`, `adm-pay`, `adm-rep`, `adm-set`.

### Navigation model
There is **no router.** Screens are divs toggled by adding/removing `.active`. There are no URL changes.

### Roles
Role‑gated flows for `employee`, `manager`, and `admin`, reached through a role picker → PIN/password login.

### Backend (Supabase)
- Client is created once: `supabase.createClient(SUPA_URL, SUPA_KEY)` (search for `SUPA_URL`, ~line 682).
- `SUPA_KEY` is the **anon/public** key and is intentionally shipped in the client. Supabase Row‑Level Security is the real access boundary — never assume the client can be trusted, and never paste a service‑role/secret key into this file.
- Data access is via `sb.from("<table>")...`. This lean app touches only a handful of tables: `profiles`, `attendance`, `performance`, `messages`, `leave_requests`, `salaries`, `monthly_payroll`, `announcements`, `system_settings`, `employees`. Grep `\.from("` for the current set.
- The Supabase project itself is a **large shared ERP/CRM backend (~110 tables)** also used by the bigger `LOWES` app — this file only uses a small slice of it. See `LOWES/CLAUDE.md` for the full data‑model map before assuming a table doesn't exist.
- **`profiles` is the central identity/user table** (holds `pin`, `password`, `role_type`, permissions, etc.), not `employees` (a small legacy table). `system_settings` is a generic `key`/`value` store.

### External libraries (CDN, no install)
- `@supabase/supabase-js@2` — backend client.
- `xlsx@0.18.5` (SheetJS) — Excel export/import for reports & payroll.
- `Tajawal` Google Font; print views include print‑specific styles (e.g. payslip generation).

### Language & direction
UI text is Arabic; document is `dir="rtl" lang="ar"`. Keep RTL layout and Arabic strings consistent with the surrounding code when adding UI.

## Working in this codebase

### Conventions to follow
- **Match the existing style:** compact vanilla JS, global‑scope function declarations, terse names, inline `onclick="..."` handlers. Do not introduce a framework, bundler, or modules.
- Keep everything in the single `lowes-app` file unless explicitly asked to split it.
- Use CSS variables from `:root` (and verify dark mode via `[data-theme="dark"]`) instead of hard‑coded colors.
- New screens follow `<div class="screen" id="screen-...">` and are shown by toggling `.active`.
- All data access goes through the single Supabase client and `sb.from("...")`.

### Running / testing
No test suite and no build. Serve the folder over HTTP and open the file as HTML, e.g.:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/lowes-app
```

(If the browser won't render it because of the extensionless name, copy it to `index.html` locally for testing — but commit changes to `lowes-app`.)

Any local run hits **live Supabase** with the shipped anon key, so treat it as **real data** and be careful with writes.

## Git workflow

- Active development branch for this work: **`claude/claude-md-docs-1ntst7`**. Develop and push here; do not push to other branches without explicit permission.
- Push with `git push -u origin <branch>`; after pushing, open a **draft** pull request if one doesn't already exist.
- Commit messages: clear and descriptive, present tense.
