# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install
npm run dev      # Vite dev server on http://localhost:5173
npm run build    # production build to dist/
npm run lint     # eslint . (flat config, ignores dist/)
npm run preview  # serve the built dist/
```

There is no test framework in this project — no test runner, test script, or test files exist. Do not claim tests pass; verify changes with `npm run lint` and `npm run build`.

## What this repository is

The UI-only frontend for an AI invoice and billing manager (React 19 + Vite + Tailwind v4). There is **no backend, no `.env`, and no database**. Every real API call is written but commented out; the app runs against an in-memory mock store. Treat this as a boilerplate whose central design constraint is that it must stay swappable to a real backend without touching hooks, contexts, or pages.

## Architecture

### The four-layer data flow

```
pages/ + components/  →  hooks/use*.js  →  api/*.js  →  mock/api.js  →  mock/store.js
                         (React Query)     (swap point)  (fake handlers)  (in-memory data)
```

Each layer only knows the one below it. The consequence that matters: **`src/api/*.js` is the only layer that knows whether data is mock or real.** Everything above it is backend-agnostic already.

### The mock ↔ real swap contract

Every file in `src/api/` has the same shape — a `── Real API ──` block of commented axios calls and a live `── Mock ──` block delegating to `mock.*`:

```js
import { mock } from "@/mock/api";
// import { apiClient } from "./client";

export const invoicesApi = {
  // ── Real API (uncomment when backend is ready) ──
  // list: (params = {}) => apiClient.get("/invoices", { params }).then((r) => r.data.invoices),
  // ── Mock (local data) ──
  list: (params = {}) => mock.invoices.list(params),
};
```

To go live: uncomment the axios client in `src/api/client.js`, flip the comment blocks in each `src/api/*.js` (re-adding the `apiClient` import), then delete `src/mock/`. `vite.config.js` already proxies `/api` → `http://localhost:8000`.

**When adding or changing any API method, maintain both blocks in parallel.** The commented real-API lines are the de-facto spec for the backend — endpoint path, verb, and response envelope (`r.data.invoice`, `r.data.invoices`, …). Mock handlers must resolve to the *already-unwrapped* value that the real call resolves to, so the two are interchangeable. Breaking that symmetry is the main way to damage this codebase.

`src/api/client.js` currently exports `apiClient = null` as a placeholder so stray imports do not crash in mock mode.

### Mock layer specifics

- `mock/store.js` builds the dataset at module load with a seeded mulberry32 PRNG, so demo data is deterministic across reloads but **mutations do not persist** — a page refresh resets everything. That is expected.
- `mock/api.js` handlers `await delay(~280ms)` so loading skeletons and pending states exercise properly. Keep the delay when adding handlers.
- Mock CRUD mutates `store` in place and re-runs cross-entity reconciliation. `reconcileInvoice()` flips an invoice to `paid` once payments cover its total (and back to `sent` if they no longer do) — payments and invoice status are coupled, not independent.

### Domain rules (in `mock/store.js`, mirror them if you touch money)

- `computeTotals(items, taxRate, discount)` is the single source of invoice math: line `amount = quantity * rate`, discount clamped to at most subtotal, tax applied to the *post-discount* base. All values pass through `round2`. Never recompute totals inline in a page.
- `effectiveStatus(inv)` derives `overdue` at read time from `status === "sent"` plus a past `due_date`. Overdue is **never stored** — the store only holds `draft` / `sent` / `paid`. Serializers attach it as `effective_status`; render that, not `status`.

### React Query conventions

Hooks in `src/hooks/` own the query keys and the invalidation fan-out. Keys are exported (`invoicesKey`, `invoiceKey`, `settingsKey`, …) — import them rather than writing key arrays by hand.

Invalidation is deliberately wide because the entities are coupled: an invoice mutation invalidates `["invoices"]`, `["dashboard"]`, and `["clients"]`; a payment mutation additionally invalidates `["reports"]`. When adding a mutation, invalidate every aggregate that reads the mutated entity.

Client defaults live in `App.jsx`: `retry: 1`, `refetchOnWindowFocus: false`, `staleTime: 30s`.

### Providers and routing

`App.jsx` nests `QueryClientProvider → ThemeProvider → UIProvider → AuthProvider → RouterProvider`. `AuthProvider` sits inside `QueryClientProvider` because logout calls `queryClient.clear()`.

`routes.jsx` splits public routes (`/`, `/login`, `/register`) from a `ProtectedShell` parent that gates on `useAuth()` — it renders a loading state while `loading`, redirects to `/login` when there is no `user`, and otherwise renders `AppShell` with an `<Outlet />`.

In mock mode `mock.auth.me()` deliberately rejects with 401 so the app boots logged out and demos the login flow; `login`/`register` accept any credentials.

`AppShell` owns the sidebar, topbar, route-change page transitions, and the ⌘K / Ctrl+K command palette.

### Contexts

- `AuthContext` — `user`, `loading`, `login`, `register`, `logout`, `refresh`, `updateProfile`. All auth state lives here, not in React Query.
- `ThemeContext` — writes `data-theme` on `<html>` and persists to `localStorage` under `arr-theme`; falls back to `prefers-color-scheme`.
- `UIContext` — `useToast()` returns a callable with `.success` / `.error` / `.info` helpers, and renders its own animated toast viewport. Use it for user feedback rather than adding another notification mechanism.

Each context's hook throws if used outside its provider.

## Styling

Tailwind v4 via the `@tailwindcss/vite` plugin — configuration is CSS-first in `src/index.css` (`@theme` block), **not** a `tailwind.config.js`; there is no such file.

Colors come from CSS custom properties defined once on `:root`/`[data-theme="light"]` and redefined on `[data-theme="dark"]`. Components reference them through arbitrary values:

```jsx
className="bg-[var(--surface)] border-[var(--border)] text-[var(--ink-muted)]"
```

### Brand palette

Four brand colors, documented at the top of `src/index.css`. Every accent token resolves to one of them; keep new UI on this ramp.

| | Hex | RGB | Role |
|---|---|---|---|
| P1 | `#112E81` | `17, 46, 129` | deep navy — `--accent-strong` (light), `--accent-hero` |
| P2 | `#4647AE` | `70, 71, 174` | indigo — `--accent` (light), solid fills with white text |
| P3 | `#4382DF` | `67, 130, 223` | bright blue — `--accent-hero-2`, `--accent` (dark), charts |
| P4 | `#AACCD6` | `170, 204, 214` | pale blue — `--accent-strong` (dark), tints |

Supporting shades derived from these: `#7FA9E8`, `#C9DEF6`, `#E4ECFB` (P3 tints), `#1B2A6B`, `#101A42` (P1 shades). Surfaces, ink, and borders are navy-tinted neutrals derived from P1.

`--success`, `--warning`, and `--danger` stay green/amber/rose — they are semantic status colors, deliberately outside the brand ramp.

Chart series colors live in `src/pages/Dashboard.jsx` as `T1`/`T2`/`T3` (P3/P2/P1). `InvoiceDocument.jsx` and `public/favicon.svg` carry their own hardcoded copies of the palette because neither can read CSS variables.

**Do not use Tailwind's built-in color palette** (`bg-slate-100`, `text-gray-500`, …) — it will not follow the theme. Token set: `--bg`, `--surface`, `--surface-2`, `--border`, `--ink`, `--ink-muted`, `--accent`, `--accent-strong`, `--accent-soft`, `--accent-hero`, `--accent-hero-2`, `--success`, `--warning`, `--danger`, plus `--shadow-card` / `--shadow-hover` (exposed as `.shadow-card` / `.shadow-hover`). Utility classes `.font-display`, `.font-serif`, and `.tabular` (tabular numerals — use on money and metrics) are also defined there.

`src/components/ui/` holds the primitives (Button, Card, Badge, Input, Tabs, Avatar, Checkbox, Skeleton, EmptyState, IconButton). Variants use `class-variance-authority`; merge incoming classes with `cn()` from `@/lib/utils` (clsx + tailwind-merge). Extend an existing primitive's `cva` variants rather than hand-rolling a one-off styled element.

Formatting helpers also live in `@/lib/utils`: `formatMoney`, `formatNumber`, `formatDate`, `toDateInput`, `relativeTime`, and the `CURRENCIES` list.

## Imports

`@/` aliases `./src`, configured in both `vite.config.js` and `jsconfig.json` — keep the two in sync. Prefer `@/...` over deep relative paths.

## Feature-specific notes

- **PDF** — `components/invoice/InvoiceDocument.jsx` is a `@react-pdf/renderer` document with its own `StyleSheet` and hardcoded hex colors (the PDF renderer cannot read CSS variables), rendered through `PDFDownloadLink` in `pages/InvoiceDetail.jsx`. Theme token changes must be mirrored there by hand if the invoice PDF should match.
- **AI features** — `api/ai.js` exposes `receiptParse`, `businessSummary`, `paymentReminder`, and `writeNote`, consumed by Dashboard, Expenses, InvoiceDetail, and InvoiceEditor. In boilerplate mode these return canned responses from `mock.ai` with no model provider wired up; the commented real-API block shows the intended endpoints (`receiptParse` posts `multipart/form-data`).
- **Charts** — Recharts, used in Dashboard, Reports, ClientDetail, and `components/dashboard/StatCard.jsx`.
- **Settings** — `useSettings()` supplies company details, `currency`, `tax_rate`, `invoice_prefix`, and `next_seq`; InvoiceEditor and InvoiceDetail read it for defaults and numbering. Its update mutation writes the cache directly via `setQueryData` instead of invalidating.

## Notes

Pages are large single-file screens (InvoiceEditor ~580 lines, Landing ~530, Dashboard ~480). Read the whole page before editing it; related state is often colocated far from where it is rendered.
