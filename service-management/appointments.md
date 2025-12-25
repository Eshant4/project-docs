# Service Management – Appointments Module
Covers `src/helpers-and-constants/Service-management/appointments/Appointments.tsx`.

## Purpose
- Display appointment metrics and a paginated grid of appointments.
- Provide a drawer to schedule new appointments.
- Enforce button permissions via dashboard auth.

## Core Components & Hooks
- React functional component `Appointments`.
- Data hooks: `useGetAppointmentsQuery` (fetch list), `useDataGridPagination` (page/pageSize state).
- Permissions: `useDashboardAuth()` → `events` passed to action buttons and grid header.
- Drawer: `useDrawer()` controlling `DrawerLayOut` that hosts `ScheduleAptDrawer` (form not shown here).
- UI: `CustomHeaderwithDataGrid`, `CustomDataGrid`, `NewCounter` cards, `DataGridButton` actions.
- Utilities: `formatTimeUTC` (formats appointment time), `logger` for debug.

## Data Contracts (in-file interfaces)
- `Appointment` (API shape):
  - Nested: `admuser.username`, `aptpurpose.purposename`.
  - Fields: `aptdate`, `bookingdate`, `aptid`, `purposeid`, `slotid`, `studentid`, `typeid`, `branchid`, `userid`, `contact`, `email`, `parentname`, `studentname`, `upddate`.
- `AppointmentRow` (grid shape):
  - `aptid`, `username`, `purposename`, `appointmenttime?`, `aptdate`, `typeid`, `isonline?`, `contact`, `branchid`, `userid`, `appointmentdate?`, `mobile?` (implicit from API).

## Data Flow (UI → Logic → UI)
1) Mount: `useGetAppointmentsQuery({ rowsPerPage, pageNumber })` fires using paginationModel.
2) On success: map API rows to include `username` (from `admuser.username`) and `purposename` (from `aptpurpose.purposename`); set local `rows`.
3) Pagination change triggers `refetch`.
4) Render:
   - Counters: `ticketData` array → `NewCounter` cards.
   - Grid: `CustomHeaderwithDataGrid` wraps `CustomDataGrid` with columns defined in `COLUMN`.
   - Action button “Schedule Appointment” opens right drawer via `toggleDrawer`.
5) Drawer: `DrawerLayOut` shows `ScheduleAptDrawer` for create (edit/delete not wired yet).

## Grid Columns & Rendering
- `username`: Avatar with first letter + name + aptid subtitle.
- `purposename`: plain text.
- `appointmenttime` (“Booked On”): formatted via `formatTimeUTC`.
- `appointmentdate` (“Meeting On”): raw value.
- `isonline` (“Type of Meet”): displays “Online” or “In - Person”.
- `mobile` (“Contact”): raw.
- `Action`: `DataGridButton` (view disabled). Hooks:
  - `editOnClick`: calls `handleEditRow` (currently logs only).
  - `onDeleteClick`: calls `handleDeleteRow` (currently logs only).
- `getRowId`: `row.appointmentid` (note: API provides `aptid`; ensure IDs align).

## Drawer Flow
- Opened from header button; right-anchored.
- Title “Schedule Appointment”.
- Width: 600 (responsive 100vw on small).
- Body: `ScheduleAptDrawer` (implements form; not covered here).

## Validations & Error Handling
- This file does not perform form validation or show error states for fetch failures.
- Overlap/time validation likely lives inside `ScheduleAptDrawer`.
- TODO: Wire edit/delete actions and surface fetch/load/error states in the grid.

## Animation/Motion
- No framer-motion in this file. Drawer animations handled by `DrawerLayOut` (MUI slide/transition).

## Metrics Cards (`ticketData`)
- `ticketData` imported; rendered as 4 `NewCounter` cards (title/number/icon). Update this data source for real metrics.

## Permissions (`useDashboardAuth`)
- Returns `events`, passed to `DataGridButton` and `CustomHeaderwithDataGrid` to control enabled actions based on user permissions.

## Time/Date Formatting
- Uses `formatTimeUTC` for `appointmenttime`.
- `aptdate`/`bookingdate`/`appointmentdate` are displayed raw; consider consistent formatting.

## Extension Points / Future Enhancements
- Implement edit/delete (API mutations + confirmations + toasts).
- Align `getRowId` with API (`aptid` vs `appointmentid`).
- Add loading/empty/error UI in grid and cards.
- Add filters/search (by purpose, host, date range, online/in-person).
- Add column for `aptdate` vs `bookingdate` explicitly if needed.
- Add row click to open details or edit drawer.
- Add CSV export or bulk actions if supported by permissions.
- Respect server pagination metadata (rowCount already wired from `data?.data?.count`).

## Gatekeeping & Edge Cases
- If API returns empty data, grid shows empty rows (no crash).
- Pagination refetch occurs when `paginationModel.page` changes.
- Drawer toggle safely handles mouse/keyboard events.
- No optimistic updates; all state comes from latest query.

## Quick Dev Checklist
- Confirm `appointments.service` response shape matches mapping (`admuser`, `aptpurpose`).
- Ensure `ScheduleAptDrawer` posts new appointments and calls `refetch` on success.
- Add toast/error handling on fetch failure.
- Verify `DataGridButton` permission handling via `events`.
- Standardize date/time fields (meeting date vs booking date vs slot time).
