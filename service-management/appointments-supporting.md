# Service Management – Appointments (Supporting Components)

## Overview
These components power the Appointment drawer workflow: collecting personal details, selecting purpose/host/type, booking a slot, and previewing a gatepass. Utilities cover icons, metrics, and validation.

## File: ScheduleAptDrawer.tsx (Wizard Shell)
- Purpose: Multi-step drawer for creating an appointment.
- Steps: `PersonalDetails` → `Purpose` → `SlotBooking` → `Gatepass`.
- State: Formik (initial values include visitor/contact details, purpose/type, date/slot, host, category, vendor/student fields, and gatepass response fields).
- Validation: Maps step → Yup schema (`validation.ts`). Only current-step fields are marked touched/validated on next.
- Submit flow:
  - Steps 0–1: validate, advance.
  - Step 2 (Slot): validate, build payload `{purposeid, personid(hostid), categoryid, visitorname, mobile, email, appointmentdate, appointmenttime, isonline}`; call `useCreateAppoitmentMutation`. On success, saves `aptvisitorcode/aptvisitordate/aptvisitortime`, advances to Gatepass.
  - Step 3: closes drawer.
- UI: `ProgressStepper` shows stepper; `BottomNavbar` controls next/back. Custom button labels per step.
- Edge cases: If validation fails or mutation throws, user stays on step (errors set via `setFormikYupErrors`).

## File: PersonalDetails.tsx (Step 1)
- Purpose: Capture visitor identity, category, and related info.
- Data sources: `useGetClassQuery` (class list), `useGetVisitorCategoryQuery` (categories), `useOptions` (maps to value/label).
- Behavior:
  - Category dropdown (SearchableSelect). Sets `categoryid` and local `selectedCategory`.
  - Conditional forms:
    - Parent: student id/name, class, visitor name, contact (10 digits), email.
    - Employee (vendor): vendor id/name, class, visitor name, contact, email.
  - Uses Formik context to bind fields and errors; shared input styling.
- Validation: Enforced at step level via `personalDetailsSchema` (required category, alphabetic visitor name, 10-digit contact, valid email).

## File: Purpose.tsx (Step 2)
- Purpose: Choose appointment purpose → then host; choose type (online/offline).
- Data: `useGetPurposeHostsQuery` returns purposes with hosts. `usePurposeIcon` maps purpose icon ids to MUI icons.
- UI/Interaction:
  - Accordion sections for Purpose and Type; toggled with Collapse.
  - Purpose tiles: selectable cards with icon; selecting sets `purposeid` and resets host.
  - Host list: when purpose selected, an animated (framer-motion `AnimatePresence`) panel shows radio list of hosts; sets `hostid`.
  - Type tiles: offline (0) / online (1); sets `typeid`.
- Motion: `AnimatePresence` + `motion.div` for host panel expand/collapse with fade/slide.
- Validation: `purposeSchema` (purposeid, typeid required).

## File: SlotBooking.tsx (Step 3)
- Purpose: Pick appointment date and time slot.
- Date:
  - `DateCalendar` (MUI X) with custom styles; Sundays disabled; stores `aptdate` (YYYY-MM-DD).
- Slots:
  - Fetch available slots via `useLazyGetTimeSlotsQuery` when `aptdate`, `purposeid`, `typeid` set.
  - Interprets API response:
    - `datetimeslot` [start, end] → filters preset morning/afternoon slots within range.
    - `daytimeslot` array of start/end pairs → builds ranges, filters preset slots within ranges.
  - Preset slots (`timeSlot`): morning & afternoon arrays.
  - Selecting a slot sets `slot` in Formik as ISO using `dayjs` merge of date + start time.
- UI: Slot chips with selected highlight; morning/afternoon grouped with icons.
- Validation: `slotSchema` (aptdate, slot required).
- Error handling: Logs on fetch failure; clears slots.

## File: Gatepass.tsx (Step 4)
- Purpose: Preview booked appointment and visitor code; offer download/guidance.
- Data: Reads Formik values `aptvisitordate`, `aptvisitortime`, `aptvisitorcode` set after successful booking.
- UI: Card with date/time (formatted via dayjs + `formatIsoToIST_Dayjs`), visitor code badge, static address block with “Get Directions” (opens Google Maps), “Download Gate Pass” button (non-wired), note card, WhatsApp opt-in checkbox.
- State: Local `checked` for WhatsApp checkbox only.

## File: validation.ts
- Yup schemas:
  - `personalDetailsSchema`: categoryid required; visitorname alphabetic; contact numeric 10 digits; email valid/required.
  - `purposeSchema`: purposeid required; typeid required.
  - `slotSchema`: aptdate date required; slot required.

## File: usePurposeIcon.tsx
- Maps numeric purpose values to labeled MUI TwoTone icons.
- API: `options` (label/value/Icon), `iconMap`, `getOptionByValue(value)`, `getIconByValue(value, props?)`.
- Used in `Purpose` tiles.

## File: ticketData.tsx
- Static counters for dashboard cards:
  - Today’s Appointment, Total Appointment, Online Appointment, Free Slots (numbers currently octal literals).
- Icons styled with colored backgrounds; consumed by `NewCounter` in `Appointments.tsx`.

## File: SlotBooking Time/Date Utils (inline)
- Uses dayjs + `isBetween` to filter preset slots into available buckets based on API time ranges.
- Converts selected slot to ISO for backend.

## File: Gatepass Formatting
- `formatDate` → “DD MMMM YYYY”; time via `formatIsoToIST_Dayjs`.
- Hardcoded address; map link opens new tab with Google Maps directions.

## API Touchpoints (beyond Appointments.tsx)
- `useCreateAppoitmentMutation` (book on step 3).
- `useGetPurposeHostsQuery` (purpose/host list).
- `useLazyGetTimeSlotsQuery` (available slots by date/purpose/type).
- Category/classes: `useGetVisitorCategoryQuery`, `useGetClassQuery`.
- All other API calls (edit/delete, download gatepass) are not wired here.

## Motion Usage
- `Purpose` uses `framer-motion` (`AnimatePresence`, `motion.div`) to animate host panel expand/collapse under a selected purpose. No other motion in these supporting files.

## Validations Overview
- Step-gated validation in `ScheduleAptDrawer` using Yup schemas per step.
- Formik `touched` set on current-step fields before validation to surface errors.

## Known Gaps / Future Enhancements
- Wire edit/delete actions in main grid and gatepass download.
- Persist WhatsApp opt-in from Gatepass.
- Replace hardcoded address in Gatepass with API data.
- Validate vendor/student lookups (Search buttons currently decorative).
- Handle API errors with toasts/snackbars (currently silent in slot fetch; booking errors just hold step).
- Use real counts in `ticketData` (remove octal literals).
- Support timezone-safe slot formatting end-to-end (slot stored as ISO; ensure backend expects ISO).
- Add loading/empty/error states to SlotBooking when no slots or fetch fails.
- Add “selected host/purpose/type” summary banner in drawer for clarity.
