# Helpers: Appointment Booking Settings

## Entry: MainStepper
- Two-tab Stepper (`SetupTabs`):
  1) Add Host → `ToMeetVisitor`
  2) Add Purpose → `AddPurposeVisitor`
- Manages active tab state; supplies icons and basic styling.

## Host Management (`tomeet/ToMeet.tsx`, `tomeet/AddHostDrawer.tsx`)
- Lists hosts in `CustomDataGrid` with pagination (`useDataGridPagination`).
- Fetch: `useGetHostQuery({ page, pageSize })`; transforms payload to rows/count.
- Actions:
  - Add/Edit opens right drawer (`AddHostDrawer`) with Formik + Yup validation (name/email/phone).
  - Delete confirms via `useErrorDialog`, calls `useDeleteHostMutation`.
- Create/Update APIs: `useCreateHostMutation`, `useUpdateHostMutation`.
- UI bits: Avatar from first letter, formatted created date, toast feedback on mutations.

## Purpose Management (`purpose/Purpose.tsx`, `purpose/AddPurposeDrawer.tsx`, `purpose/Availability.tsx`)
- Lists purposes in `CustomDataGrid` (pagination).
- Fetch: `useGetPurposeDataQuery(paginationModel)`; delete via `useDeletePurposeMutation`.
- Columns: purposename, duration (staytime), hosts (AvatarGroup from `tomeetnames` cleaned of titles), event type, availability (Default/Custom), created date, actions (edit/delete).
- Drawer (`AddPurposeDrawer`):
  - Formik form with fields: purposename, duration (select), eventtype, iconValue/IconName, hosts (multi-select via `ParamMultipleSelectWithLabelKeys`), availability/custom intervals, isDisabled flag for schedule.
  - APIs: create/update via `useCreatePurposeMutation` / `useUpdatePurposeMutation`.
  - Host options loaded with `useGetallHostQuery`.
  - Reusable sections/BottomNavbar for submit/cancel; toast feedback on success/error.
  - Uses `usePurposeIcon` (appointments helper) for icon options.
- Availability (`Availability.tsx`):
  - Formik-aware availability editor.
  - Default vs Custom schedule toggle (menu).
  - Weekday components (Sunday–Saturday) and `uniqueTimeIntervals` for date-specific overrides.
  - Date picker (StaticDatePicker) with custom day selection; supports marking unavailable.
  - Time validation utilities (`_timeUtils/timeUtils.ts`): generateTimeOptions, convertMinutesToTime, getTimeValue, validateTimesAgainst (overlap and start<end checks).
  - Tracks errors per interval; updates Formik values on changes.

## Utilities/Types
- `purpose/_timeUtils/timeUtils.ts`: time list + overlap validation.
- Weekday partials (`_week-days/*.tsx`): per-day UI for time intervals (not detailed here but consumed by Availability).
- Common styles: boxStyle definitions for inputs/drawers; consistent MUI styling for grids and inputs.

## Data Flow (Hosts)
1) Load hosts via `useGetHostQuery` → rows/count mapped.
2) Grid renders rows; edit/delete actions.
3) Add/Edit drawer → Formik submits to create/update mutations → toast + close drawer; delete confirms via dialog then mutation.

## Data Flow (Purposes)
1) Load purposes via `useGetPurposeDataQuery` with pagination.
2) Grid shows purpose data and host avatars; edit/delete actions.
3) Add/Edit drawer builds payload (purposename, staytime, personid[], eventtype, iconValue) → create/update mutation → toast + close; optional availability customization handled in-place; hosts prefilled when editing.

## Conditional UI
- Drawer titles switch “Add”/“Edit” based on `selectedRowData`.
- Availability column shows “Default” when matches defaultAvailability; otherwise “Custom”.
- Duration select prefilled to `staytime` on edit; hosts preselected based on `personid` + `tomeetnames`.

## Error Handling
- Host delete: confirm dialog; toast error on failure; success toast with server message.
- Purpose delete/create/update: confirm dialog for delete; toasts for success/error.
- Form validation: Yup for host form (name/email/phone). Availability validations prevent overlapping/invalid times.

## Styling
- MUI SX on tabs, grids, and inputs; custom padding for settings layout.
- Drawers use `DrawerLayOut` with fixed width and BottomNavbar footer buttons.
- Avatar/Badge colors default; icon colors for action buttons (purple edit, red delete).

## Future Enhancements (suggested)
- Add explicit loading/error states in grids (placeholders/spinners) instead of silent fallback.
- Persist availability customization for purposes (currently primarily form-level; ensure payload includes availability if backend supports).
- Debounce pagination refetch to reduce API calls.
- Add search/filter on host and purpose grids.
