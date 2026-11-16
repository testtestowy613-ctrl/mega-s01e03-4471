<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // The three problems ticket #4471 actually holds. Every number is computed from the export
  // at render time; nothing here is typed by hand.
  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(new Set<string>(notes.oneDayMove));

  type Problem = { key: string; what: string; reports: Report[]; note?: string };
  const problems = $derived<Problem[]>([
    {
      key: 'parser',
      what: 'the shared parser reads every date month first; day-first tenants get flipped dates',
      reports: merged.filter((r) => causeOf(r, tenantOf(r)) === 'parser'),
      note: 'the desk counts the one-day moves below under this cause, with a footnote (CauseTable)',
    },
    {
      key: 'hint',
      what: "booking-v3's date hint says MM/DD/YYYY, but the form reads its input the tenant's way (day first)",
      reports: merged.filter((r) => causeOf(r, tenantOf(r)) === 'hint'),
    },
    {
      key: 'one-day move',
      what: 'dates land one day off (the reports the desk read as one-day moves, desk-notes.json)',
      reports: merged.filter((r) => moved.has(r.id)),
      note: 'counted under parser in the desk\u2019s own view; filed as their own ticket here',
    },
  ]);

  const days = (rs: Report[]) => rs.map((r) => r.receivedAt.slice(0, 10)).sort();
</script>

<p class="sum">
  <strong>{merged.length}</strong> unique reports merged into #{ticket} ·
  <strong>{problems.length}</strong> problems, one ticket each ·
  counted as the desk counts them (CauseTable): parser {problems[0].reports.length}, hint {problems[1].reports.length},
  with the one-day moves footnoted under parser.
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>sample</th></tr>
  </thead>
  <tbody>
    {#each problems as p (p.key)}
      <tr>
        <td><code>{p.key}</code></td>
        <td>{p.reports.length}</td>
        <td>{new Set(p.reports.map((r) => r.tenant)).size}</td>
        <td>{days(p.reports)[0]} → {days(p.reports).at(-1)}</td>
        <td>{p.reports.slice(0, 3).map((r) => r.id).join(', ')}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each problems as p (p.key)}
  <p class="what"><code>{p.key}</code> — {p.what}.{#if p.note} {p.note}.{/if}</p>
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .what { color: var(--muted); font-size: 0.85em; margin: 0.2rem 0; }
</style>
