<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  type Row = { problem: string; what: string; count: number; tenants: number; seen: string; note?: string };
  // issues: problem → { url, state } of the ticket filed for it.
  let { ticket, issues = {} }: { ticket: number; issues?: Record<string, { url: string; state: string }> } = $props();

  const WHAT: Record<string, string> = {
    parser: 'shared parser reads day-first dates month first',
    hint: 'booking-v3 form hint says MM/DD/YYYY, the form reads the tenant’s day-first order',
    unplaced: 'no tenant on file: read the report',
    'one-day': 'date moves by one day, not a month: time zone',
  };

  const rows = $derived.by(() => {
    const all = (reports as Report[]).filter((r) => r.mergedInto === ticket);
    const merged = uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket);
    // Counted the desk's way: by cause, where each report is filed (causes.ts).
    const groups = groupByCause(merged, tenantOf);
    const out: Row[] = groups.map((g) => ({
      problem: g.cause,
      what: WHAT[g.cause],
      count: g.reports.length,
      tenants: g.tenants,
      seen: `${g.firstSeen} → ${g.lastSeen}`,
    }));
    // The one-day moves stay counted where they are filed, and get their own row.
    const moved = merged.filter((r) => notes.oneDayMove.includes(r.id));
    const filedUnder = groups.filter((g) => readAs(g, notes.oneDayMove)).map((g) => g.cause);
    if (moved.length) {
      const days = moved.map((r) => r.receivedAt.slice(0, 10)).sort();
      out.push({
        problem: 'one-day',
        what: WHAT['one-day'],
        count: moved.length,
        tenants: new Set(moved.map((r) => r.tenant)).size,
        seen: `${days[0]} → ${days[days.length - 1]}`,
        note: `also counted under ${filedUnder.join(', ')}`,
      });
    }
    return { rows: all.length, unique: merged.length, out };
  });
</script>

<p class="sum">
  <strong>{rows.unique}</strong> reports in #{ticket}
  ({rows.rows} rows in the export; {rows.rows - rows.unique} came twice) ·
  <strong>{rows.out.length}</strong> problems
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows.out as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code><br /><span class="what">{r.what}</span></td>
        <td>{r.count}{#if r.note}<br /><span class="what">{r.note}</span>{/if}</td>
        <td>{r.tenants}</td>
        <td>{r.seen}</td>
        <td>
          {#if issues[r.problem]}
            <a href={issues[r.problem].url}>{issues[r.problem].url.replace(/^.*\/issues\//, '#')}</a><br /><span class="what">{issues[r.problem].state}</span>
          {:else}<span class="none">none yet</span>{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .what { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
