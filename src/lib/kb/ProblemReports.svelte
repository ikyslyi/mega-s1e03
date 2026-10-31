<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // The reports behind one problem, so the fix can be checked against each of them.
  let { ticket, problem }: { ticket: number; problem: string } = $props();

  // Same counting as ProblemTable: by cause, one-day moves listed apart and still filed by cause.
  const rows = $derived.by(() => {
    const merged = uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket);
    if (problem === 'one-day') return merged.filter((r) => notes.oneDayMove.includes(r.id));
    return groupByCause(merged, tenantOf).find((g) => g.cause === problem)?.reports ?? [];
  });
</script>

<details>
  <summary>{rows.length} reports</summary>
  <table>
    <thead><tr><th>report</th><th>received</th><th>tenant</th><th>channel</th><th>says</th></tr></thead>
    <tbody>
      {#each rows as r (r.id)}
        <tr><td><code>{r.id}</code></td><td>{r.receivedAt.slice(0, 10)}</td><td>{r.tenant}</td><td>{r.channel}</td><td>{r.body}</td></tr>
      {/each}
    </tbody>
  </table>
</details>
