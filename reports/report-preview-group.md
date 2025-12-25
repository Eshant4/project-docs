# ReportPreviewGroup

## Purpose
Display a report with grouping, totals, print, and export. Delegates grouping/flattening heavy work to a web worker (via Comlink).

## Props (key)
- `headerName: string`
- `columns: ReportColumn[]`
- `rows: ReportRow[]`
- `properties?: ReportProperties` (page size/orientation, margins, font/color, table design, uniqueRowKey, pageNumber, generated* metadata)
- `savedSortSettings: SortSetting[]`
- `savedgroupsettings: string[]` (grouped columns)
- `currBranchName?`, `orgLogo?`, `currFinYear?`
- `allDataLoaded?` (enables print/export buttons)
- `filterslist?`
- `onClose?` (unused here)

## Worker & Grouping (Comlink)
- Creates worker via `createMergeWorker()` (wraps `mergeWorker.ts` using Comlink).
- On rows/sort change:
  - Computes `uniqueRowKey` (props -> fallback to common ids).
  - Filters sort settings to valid columns with data; ignores `upddate`.
  - Calls `api.merge({ rows, savedSortSettings: effectiveSortSettings, uniqueRowKey })`.
  - Receives `level`, `grandTotal`, `flatRows`; falls back to flat rows + grand total if empty.
- Terminates worker on unmount.

## UI Structure
- Header: org logo, branch/finyear, headerName; shows Print/Export buttons when `allDataLoaded`.
- Actions:
  - Print: builds printable HTML/CSS and opens/prints a new window.
  - Export Excel: dynamic import `exportReportExcel`.
  - Export PDF: dynamic import `exportReportPDF`; includes sort/group info and logo.
- Table: `ReportTable` consumes `flatRows` (group rows, totals, data).
- Footer: grouping level, total rows; sorting summary; generatedBy/date/time; applied filters (via `formatAppliedFilters`).

## Printing Logic
- `buildPrintCSS`: applies page size/orientation, margins, fonts, colors, table design (alternating rows/cols), page numbers.
- `formatCellForPrint`: handles photos/idproof (image tags), timeout slicing, date fields, HTML escaping.
- Writes full HTML document to new window and triggers print; re-enables print button after `afterprint`.

## Exports
- Excel: passes columns, flatRows, headerName, branchName, fileName.
- PDF: passes headerName, branchName, finyear text, columns, flatRows, properties, logo, sortInfoText (marks grouped columns with `(G)`).

## State
- `level: GroupLevel`, `grandTotal: number`, `flatRows: FlatRow[]`, `showPrintButton: boolean`.
- `uniqueRowKey` derived via `useMemo`; `effectiveSortSettings` filtered via rows/columns.

## Notes & Enhancements
- Print/export guarded by `allDataLoaded` to avoid partial outputs.
- Could add loading/empty state when rows are empty.
- Consider better handling of nested array fields (currently normalized earlier).
