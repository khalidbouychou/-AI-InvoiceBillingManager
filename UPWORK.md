# Upwork Portfolio Entry — Invoicer

Copy-paste ready. Section 1 is the listing itself; Section 2 explains how to publish it so it ranks.

---

## 1. The listing

### Title

Pick one — all are front-loaded with the terms clients actually search:

1. **React SaaS Dashboard — AI Invoicing & Billing App (React 19, Tailwind, Recharts)**
2. **Invoice & Billing SaaS UI — React Dashboard with Charts, PDF Export & Dark Mode**
3. **Full Frontend Build: AI Invoice Manager — React 19 + Vite + Tailwind CSS v4**

Keep it under 70 characters where you can; Upwork truncates long titles in search results.

---

### Description

> Paste everything between the lines into the Upwork description field.

---

**A complete, production-grade frontend for an AI-powered invoicing and billing platform — 14 screens, built from scratch in React 19.**

Freelancers and small studios lose hours every month to invoicing. I designed and built the entire user-facing product for a billing tool that removes that friction: create an invoice, track what's owed, chase overdue payments, and read your revenue at a glance.

**What I built**

- **Invoicing engine UI** — line-item editor with live subtotal, discount and VAT calculation, draft/sent/paid states, and automatic invoice numbering
- **PDF invoice generation** — client-ready documents rendered in-browser with `@react-pdf/renderer`, no server round-trip
- **Analytics dashboard** — revenue trends, a collections gauge, invoice-status breakdown and accounts-receivable aging, built on Recharts
- **Client CRM** — client records with per-client billing history, totals and payment-status charts
- **Expenses, payments and a reusable services catalog** — the full money-in/money-out picture
- **AI feature surfaces** — receipt scanning, business summaries, payment-reminder drafting and note writing, each wired to a documented API contract
- **Full design system** — themed light and dark modes driven entirely by CSS custom properties, a `cva`-based component library, and a ⌘K command palette

**Tech stack**

React 19 · Vite · Tailwind CSS v4 · TanStack Query (React Query) · React Router 7 · Recharts · Framer Motion · `@react-pdf/renderer` · Axios

**Engineering details that matter**

- **Backend-agnostic data layer.** Every API call sits behind a thin adapter, so the app runs on a mock layer today and switches to a live REST backend by changing one file per resource — no changes to hooks, pages or state.
- **Server state handled properly.** TanStack Query owns caching, loading and error states, with deliberate cross-entity invalidation so a recorded payment instantly updates the invoice, the dashboard and the reports.
- **Theming without a rewrite.** Every colour is a CSS custom property, so a complete rebrand is a token change, not a find-and-replace across 40 components.
- **Responsive desktop and tablet layouts**, a keyboard-driven ⌘K command palette (arrow-key navigation, Escape to dismiss, fuzzy matching across invoices and clients), and a clean production build.

*Note: this demo runs on a deterministic in-memory dataset so it can be explored instantly with no signup — the AI responses are stubbed to the same contract the live endpoints implement. Happy to walk you through the code or wire it to your backend.*

**I can build this for you**

If you need a SaaS dashboard, an internal admin tool, a billing or reporting interface — or a design system your team can actually build on — send me a message with what you have in mind. I'll reply with a concrete plan and timeline.

---

### Skills to tag

Upwork matches portfolio items to searches largely through these. Use every slot you get:

`React` · `JavaScript` · `Tailwind CSS` · `Vite` · `Front-End Development` · `Web Application` · `UI/UX Design` · `Dashboard` · `Data Visualization` · `SaaS` · `Responsive Design` · `REST API` · `Web Design` · `CSS` · `HTML5`

---

## 2. How to publish this so it ranks

Upwork surfaces portfolio items through keyword relevance and profile strength. These are the levers that actually move it:

**Keywords**
- Mirror the exact words clients type. They search "React dashboard" and "invoice app", not "bespoke billing solution". The title, first sentence and skill tags carry the most weight.
- Put your strongest keyword in the **first 5 words of the title** — search results truncate, and skimming clients read left to right.
- Write naturally. Stuffing the same term ten times reads badly to humans and does nothing for the algorithm.

**The first two lines do the work**
- Clients skim a grid of thumbnails. Your opening sentence must state *what it is* and *how big it is* — "14 screens, built from scratch in React 19" beats "a modern invoicing solution".
- Lead with the client's problem ("invoicing eats hours every month"), not with your tech list. Tech goes lower, where it doubles as keyword coverage.

**Images**
- The **first image is your thumbnail** — it decides whether anyone clicks. Use `06-dashboard-ai-summary.png`: it's dense, colourful and instantly reads as a real product.
- Suggested order: `06` dashboard → `02` landing page → `09` invoice detail → `15` reports → `19` dark dashboard → `17` invoice editor.
- Upwork allows multiple images per item. Fill the slots — items with more images hold attention longer, and dwell time helps.

**Profile-level factors that affect this item's reach**
- Portfolio items rank better from a 100%-complete profile with a matching specialised profile.
- Your profile title and overview should use the same keywords as this item — consistency compounds.
- Job Success Score and recent activity feed into placement. A fresh, well-tagged portfolio item added today outranks a stale one.

**Honesty is a conversion tool**
- Say plainly that the demo runs on mock data. Clients ask "can I see it live?" — having already answered builds trust instead of costing it mid-conversation.
- Never claim a backend, integration or user count you can't demonstrate. One "actually, that part isn't built" in a call ends the deal.

**Then**
- Add a live demo link if you deploy it (Vercel/Netlify, free) — a clickable demo raises conversion more than any copy tweak.
- Link the GitHub repo. Reviewable code is a strong differentiator for developer hires.
