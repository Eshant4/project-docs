# Book-an-Appointment: Availability & Custom Schedule Flow

## Where to Look
- `helpers-and-constants/service-management/settings/book-an-appointment/purpose/Availability.tsx`
- `purpose/AddPurposeDrawer.tsx` (Formik form + payload)
- `purpose/_timeUtils/timeUtils.ts` (time math/validation)
- Weekday partials: `purpose/_week-days/*.tsx` (per-day interval editors)

## High-Level Flow
1) User opens Purpose drawer (add/edit).
2) In the Availability section, they choose:
   - **Default Schedule** (Mon–Sun 9:00 AM–5:00 PM, stored in form values), or
   - **Custom Schedule** (unlocks date picker + time interval editing).
3) If Custom:
   - User selects one or more dates in the static calendar.
   - For each date, they can set start/end times per interval (non-overlapping, start < end).
   - They can mark dates unavailable (sets `unavailable: true`).
4) On submit:
   - Formik values (including `uniqueTimeIntervals` and weekday intervals) are sent in create/update purpose payload (currently staytime/event/hosts/icon are sent; add availability if backend supports).

## State/Props (Formik: `FormValues` in AddPurposeDrawer)
- `uniqueTimeIntervals`: array of `{ id, date?: Dayjs[], startTime?, endTime?, unavailable? }`.
- Day-of-week intervals: `sundayTimeIntervals`, … `saturdayTimeIntervals` (arrays of `{ id, startTime, endTime }`), default 9–5.
- `isDisabled`: true when Default Schedule is chosen; Custom toggles to false and enables editing.
- `duration`, `purposename`, `eventtype`, `hosts`, `iconValue` (other form fields).

## User Interactions in Availability.tsx
- **Schedule Toggle**: Menu (“Default Schedule” vs “Custom Schedule”).
  - Sets `isDisabled` (Formik) based on selection.
- **Date Picker**: `StaticDatePicker` with custom day renderer:
  - Click to toggle selection (multi-select). Selected dates stored in `selectedDates`.
  - All intervals’ `date` arrays are updated to the current selection.
- **Time Editing**:
  - Per-interval start/end dropdowns built from `generateTimeOptions()` (15-min increments).
  - Validation:
    - `validateTimesAgainst(index, newStart, newEnd, intervals)` checks start < end and no overlap with other intervals; errors tracked per interval in `errors` state.
- **Unavailable Toggle**: For custom dates, user can mark interval as unavailable (sets `unavailable: true`).
- **Default vs Custom Detection**: Column renderer in Purpose grid shows “Default” when availability matches defaultAvailability (all days 9–5); otherwise “Custom”.

## Time Utilities (`_timeUtils/timeUtils.ts`)
- `generateTimeOptions()`: 15-min slots 12:00 AM–11:45 PM.
- `getTimeValue("h:mm AM/PM")`: minutes since midnight.
- `convertMinutesToTime(totalMinutes)`: back to “h:mm AM/PM”.
- `validateTimesAgainst(index, newStart, newEnd, intervals)`: returns `{ invalidTime, overlapping }` messages.

## Payload Touchpoints
- Current submit payload in `AddPurposeDrawer` includes: `purposename`, `staytime`, `personid[]`, `eventtype`, `iconValue`, and `purposeid` on update.
- If backend supports availability storage, extend payload to include:
  - `uniqueTimeIntervals` (dates + times + unavailable flag)
  - Day-of-week intervals (for recurring/default schedules)
  - `isDisabled` (to indicate default schedule)
- Editing flow pre-fills hosts and duration; availability defaults to 9–5 unless extended.

## Edge Cases & Notes
- Multiple selected dates share the same intervals; changing times updates all selected dates’ intervals.
- Overlap prevention is per `uniqueTimeIntervals` list; ensure any added intervals run through `validateTimesAgainst`.
- The UI uses framer-motion for small transitions; no effect on data flow.
- Ensure `theme`/styling from parent drawer doesn’t block dropdowns/modal.
- Weekday components (Sunday…Saturday) follow the same interval editing pattern; hook them into validation if extended.

## Suggested Enhancements
- Persist availability in API payload (if supported) and hydrate on edit.
- Add inline error display for `errors` from `validateTimesAgainst`.
- Allow per-date multiple intervals (currently scaffolded via `uniqueTimeIntervals`).
- Add “copy weekday schedule to all” and “clear all dates” helpers.
- Show a summary chip of selected dates and custom times in the drawer header.
