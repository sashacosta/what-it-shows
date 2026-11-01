<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem found in a merged ticket: the two causes causes.ts can tell apart, plus the
  // one-day moves the desk noted (desk-notes.json), which are neither and get their own row.
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));
  const rows = $derived([
    ...groupByCause(rest, tenantOf).map((g) => ({ problem: g.cause, reports: g.reports })),
    { problem: 'one-day move', reports: moved },
  ]);
  const days = (rs: Report[]) => rs.map((r) => r.receivedAt.slice(0, 10)).sort();
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code></td>
        <td>{row.reports.length}</td>
        <td>{new Set(row.reports.map((r) => r.tenant)).size}</td>
        <td>{days(row.reports)[0]} → {days(row.reports).at(-1)}</td>
        <td><a href={issues[row.problem]}>#{issues[row.problem].split('/').pop()}</a></td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
