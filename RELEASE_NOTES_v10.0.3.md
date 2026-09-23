# AlphaFlow Studio v10.0.3 — Stable

**Release date:** 2026-09-23 · **Tag:** `v10.0.3` · **Channel:** `stable`
**Upgrade from:** any `10.0.x` (in-place, no migration) · **Breaking changes:** none

---

## TL;DR

This release ships the **KPI Sync Agent** — the piece that finally removes the two things that made every other Notion KPI dashboard expensive: an **Enterprise subscription** and a **paid automation platform**.

No Notion Enterprise. No Make.com. No n8n. No Zapier. No per-operation bill. The agent runs on the Notion plan you already have, talking to the Notion API with your own integration token.

---

## 🌟 Headline feature — KPI Sync Agent

Notion rollups break the moment you want a KPI that isn't a single relation hop: month-over-month deltas, weighted pipeline, burn vs. plan, tasks closed per phase. Historically the fixes were Enterprise rollups or a Make/n8n scenario that costs money every time it fires — and breaks silently when someone renames a property.

The KPI Sync Agent does the whole job inside the Studio:

| | |
|---|---|
| **Reads** | Your Ops Engine graph — Projects → Phases → Tasks (and any custom source DB you register) |
| **Computes** | KPI values, period deltas and weighted aggregates in the agent, not in the page |
| **Writes** | Idempotent upserts keyed on `KPI ID + Period` — re-runs never duplicate rows |
| **Runs** | On demand, or on a schedule you set (hourly / daily / weekly) |
| **Costs** | Nothing per run. No third-party automation subscription |
| **Requires** | Your Notion plan as-is + a Notion integration token. **No Enterprise tier** |

**Built to be trusted, not just run:**

- **Dry-run diff first** — every sync can render the exact before → after for each KPI before a single write lands. Review, then commit.
- **Rate-limit aware** — batched reads/writes with exponential backoff, so a 5,000-task workspace doesn't trip Notion's limits mid-sync.
- **Touch-nothing writes** — the agent only writes the KPI value/snapshot properties. Titles, statuses, owners, relations and page content are never modified.
- **Run log** — each run appends a row (started, finished, rows read, rows written, errors) to a sync-log database, so a stale number is always traceable to a run.
- **Failure = visible** — a failed run reports the failing KPI and property, instead of leaving yesterday's number on the dashboard looking correct.

### 5-minute setup

1. Create an internal Notion integration and share the dashboards + source databases with it.
2. In the Studio, paste the token into the agent panel — it stays on your machine / your Worker secret, never on a vendor server.
3. Register the KPIs: name, source database, filter, aggregation, target property, period grain.
4. Hit **Dry run**, approve the diff, then enable the schedule.

---

## Key features in 10.0.3

- **Agent Studio UI, rebuilt around the split screen** — card-grid entry, then work side-by-side with the result; you see the input and the output at once instead of navigating away.
- **Flagships pinned** — Content Creator and TTS sit at the top of the grid as first-class tools, not buried examples.
- **Keys stay local** — your own keys live in your browser / your Worker secret. The Studio is the interface; it is not a key escrow.
- **Native key pool, zero BYOK friction** — out of the box you get a built-in pool (3× Gemini + 1× DeepSeek V4 Flash) so a new buyer runs an agent in the first minute; bring your own keys only when you want them.
- **16 modules across 3 categories** — Org, Personal Growth, Marketing Powerhouse — all sharing the same Ops Engine spine (Projects → Phases → Tasks).
- **Select options are clean** — Knowledge Hub `Type` and Assets Logistics `Source Type` no longer carry emoji in their option values, so every option is typeable, searchable and filterable again (and relations/match filters behave).
- **Licence gating that works for outsiders** — the gate validates a licence key, not a Workspace domain, so customers on another domain install without a Workspace account setup on your side.
- **Local cockpit on `localhost:8080`** — shipped with the allow-list enforced. Opening the cockpit over `file://` is intentionally refused by CORS; run `start_server.bat` instead.

---

## Fixes

- Relation/match filters no longer silently miss rows because an option value contained an emoji.
- Repeated syncs no longer create duplicate KPI rows (idempotency key now enforced on write, not just on read).
- Scheduled runs no longer die on transient 429s; they back off and resume.
- Dry-run output now reports unchanged KPIs as unchanged instead of listing them as writes.

## Upgrade notes

- **In-place.** Replace the app files, reload, done. No database schema migration, no property renames.
- If you previously ran the agent with a hand-edited KPI property, delete those duplicate rows once — after this release duplicates can't be recreated.
- Existing licence keys keep working. Nothing to re-issue.

## Compatibility

- Notion: Free / Plus / Business / Enterprise — **no tier requirement**
- Notion API: current public API via internal integration token
- Browsers: current Chrome / Edge / Brave
- Automations: no Make.com, n8n, Zapier or any other third-party runner required or used

## Known limitations

- One Notion workspace per agent run; multi-workspace KPI roll-ups are queued for a later release.
- Period grains beyond day / week / month / quarter are not yet configurable.
- Notion API rate limits still apply to the *first* full backfill — a very large source database may take more than one run (subsequent runs are incremental).

## What's next

- Multi-workspace KPI roll-up
- Per-KPI alerting (notify when a KPI crosses a threshold, evaluated by the same agent)
- Year-over-year comparison views in the dashboard
