# Report Merge Worker

## Purpose
Offload grouping/flattening of report data to a Web Worker using Comlink to keep the UI responsive.

## Tech
- Worker module: `mergeWorker.ts`
- Client wrapper: `mergeWorkerClient.ts`
- RPC: Comlink (`wrap`, `expose`)

## API
- `merge(input: MergeInput): MergeOutput`
  - `MergeInput`: `{ rows: ReportRow[]; savedSortSettings: SortSetting[]; uniqueRowKey?: string }`
  - `MergeOutput`: `{ level: GroupLevel; grandTotal: number; visibleGroups: string[]; mergedGroups: MergedGroupEntry[]; flatRows: FlatRow[] }`
- Client usage: `const { worker, api } = createMergeWorker(); const result = await api.merge(...);` (remember to terminate worker on unmount).

## Core Algorithms
- **wrapToNode**: sanitize arbitrary row -> `GroupNode` (keeps only string/number/null), copies `count`, recurses `subgroup`.
- **detectLevel**: inspect subgroups to infer L0–L3.
- **filterSort**: drops denied columns (`upddate`) and sorts with empty data.
- **groupFromL0**: builds L1 grouping from flat rows on a field.
- **mergeL1/L2/L3**: merges nested groups; de-dupes leaves using `getRowKey` (prefers `uniqueRowKey`, else sort columns); recomputes counts bottom-up.
- **recomputeCounts/leafCount**: maintain accurate counts.
- **flatten**: emits `FlatRow[]` in display order (group headers, totals, leaves) for L0–L3, plus grand totals.

## Level Handling
- L0 with sort: attempts to group by first sort column (if data) to become L1.
- L1–L3: merges by saved sort order; supports up to 3-level grouping.
- Visible groups: keys from merged map; grandTotal computed from leaf counts.

## Utilities
- `getRowKey`: stable key from uniqueRowKey or concatenated sort columns; falls back to JSON of row.
- `filterSort`: ensures only meaningful sort settings are used in merge.

## Error/Edge Handling
- Sanitizes input rows to avoid non-serializable worker payloads.
- If no valid sort/group, produces a flat list with grand total.
- Counts are recomputed after merge for consistency.

## Notes
- Designed for large datasets; worker keeps main thread free.
- Integrates with `ReportPreviewGroup`, which handles UI/print/export.
- Add termination (`worker.terminate()`) when done to avoid leaks.
