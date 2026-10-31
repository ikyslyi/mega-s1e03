<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { splitTicket, type Problem } from '#lib/kb/problems';

  // The reports behind one problem, so the fix can be checked against each of them.
  let { ticket, problem }: { ticket: number; problem: Problem } = $props();

  const rows = $derived(splitTicket(reports as Report[], ticket, tenantOf, notes.oneDayMove).groups.find((g) => g.problem === problem)?.reports ?? []);
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
