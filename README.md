# mega-s01e03-4471 — what ticket #4471 shows

Course challenge **S01E03 "what it shows"** (MEGA.dev). The support desk merged 412 reports
about dates into one ticket, #4471, and marked it known as a single issue. This repo holds the
analysis of the kb export, one GitHub issue per problem found, and the kb page about them.

## The three problems in #4471

| problem | reports | what it is |
|---|---|---|
| `parser` | 374 | The shared parser reads every date month first; day-first tenants (en-GB, de-DE, pl-PL, …) get flipped dates. This is the one bug the ticket admits to. |
| `hint` | 33 | booking-v3's date hint says `MM/DD/YYYY`, people type what the hint says, but the form reads its input the tenant's way (day first) — a second, separate bug. Only the `form` channel, only booking-v3 tenants (alder.physio, copper.studio, pale.harbour). |
| one-day move | 5 | Dates land one day off (R-41202, R-41209, R-41620); the same desk-note cluster also holds one timezone render (R-41344) and one invite that arrived before its booking (R-41236). |

374 + 33 + 5 = **412 unique reports**. The export has 427 entries; 15 ids appear twice
(the ticket system pages with overlap — `uniqueById` in `src/lib/kb/causes.ts`).

## How the counts were computed

True cause comes from the tenant, not the intake tag: `causeOf(report, tenant)` in
`src/lib/kb/causes.ts` — locale (day-first or not) plus booking form plus channel. The
intake tag is a guess from the reporter's browser and disagrees with the true cause on
38 reports (23 tagged `hint` are `parser`; 15 tagged `parser` are `hint`).

## What is where

- **Issues** — one ticket per problem, with report counts and evidence.
- `src/` — the kb export with the new page added:
  `src/routes/kb/incidents/4471-what-it-shows/+page.svx` (signed, ends with `<RenderStamp {signature} />`),
  its numbers computed at render time by `src/lib/kb/SplitTable.svelte`.
- `page-as-the-desk-sees-it.html` — the page rendered as the desk will see it.

## The kb (from the export)

**kb** = knowledge base. What the support desk knows about how our products break, written
down where the code can keep it honest.

One desk takes reports for every product we run: `sched-prod` (calendar invites for about
300k people) and the rest. When something goes wrong, the reports reach the desk first.
This kb is how the desk triages them, what each incident turned out to be, and why we
decided what we decided.

### What's here

```
src/routes/kb/      the pages (.svx: Markdown with components)
  triage/           how the desk handles incoming reports
  incidents/        one page per big ticket
  decisions/        numbered decision records
src/lib/kb/         components and the logic they run
src/lib/layouts/    the frame every page shares
src/lib/data/       exports from the ticket system
```

### Conventions

- Words in the page, logic in `src/lib` (imported as `#lib/…`). A number that can be computed belongs in a component.
- Every page starts with frontmatter: `title`, `owner`, `status`, `reviewed`.
- A decision is never edited after it is accepted. A new record replaces it.
- A page you add is signed: its frontmatter carries your `signature` and it ends with `<RenderStamp {signature} />`, so the desk knows who wrote it.

Exported 2026-10-04 from the `support` repo, `apps/kb/` → `src/`.
