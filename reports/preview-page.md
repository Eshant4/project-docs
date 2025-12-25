# Report Preview Page

## Purpose
Standalone page that streams report data from the backend and renders it via `ReportPreviewGroup`, supporting very large datasets through chunked streaming.

## Flow Overview
1) On mount, sends `postMessage('request-report-data')` to `window.opener`.
2) Waits for same-origin message containing `ReportPayload` (validated by `isReportPayload`).
3) When payload arrives:
   - Stores `reportData`.
   - Streams rows from backend via `oboe` (incremental JSON parser).
   - Buffers rows and flushes to state in batches to avoid blocking.
4) Renders `ReportPreviewGroup` with columns, rows, properties, saved sort/group settings, filters, branch info, logo.
5) Sets `allDataLoaded` when stream completes (enables print/export in child).

## Streaming Fetch (oboe)
- URL: `${NEXT_PUBLIC_SERVER_URL}/${apiVersion}/${reportData.url}`
- Headers: `accessToken`, `branchid`, `finyearid`, `roleid`, `uid` (read from cookies; guarded for browser).
- `oboe().node('data.*', ...)` emits each row; `.done` signals completion; `.fail` logs errors.

### Buffering Strategy
- Rows pushed into `buffer`.
- Flushes:
  - Immediately when buffer ≥ 500 rows, OR
  - Debounced every 200ms.
- On complete: final flush, then `setAllDataLoaded(true)`.

### Row Normalization
- `toReportRow`: keeps only primitive values (string | number | null | undefined); arrays are recursively normalized; nested objects become `null`. Ensures worker receives serializable rows.

## State
- `reportData: ReportPayload | null`
- `rows: ReportRow[]`
- `allDataLoaded: boolean`

## Security/Guards
- Same-origin check on message events.
- Cookies read only in browser (`document` guard).
- Ignores malformed payloads (type guard).

## Render
- Container `<Box>` with padding.
- Renders `ReportPreviewGroup` only when `reportData` exists, passing:
  - `headerName`, `columns`, `rows`, `properties`, `savedSortSettings`, `savedgroupsettings`, `currBranchName`, `orgLogo`, `currFinYear`, `filterslist`, `allDataLoaded`.
- Sets `ReportPreviewPage.isReportPreview = true` (flag for app).

## Integration Notes
- Expects parent window to respond to `request-report-data` with a `ReportPayload`.
- Large datasets: streaming avoids loading entire payload in memory; grouping/flattening is deferred to the worker via child component.
- Potential enhancements: expose loading/error UI during stream; add cancellation on unmount; handle auth errors explicitly.
