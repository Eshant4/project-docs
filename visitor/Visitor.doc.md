# Visitor Module – End-to-End Guide

## 1) Overview

- Handles visitor pre-registration, gate pass verification, on-site check-in/out, guard dashboard, approval flows, and public-facing links.
- Flows covered:
  - Public registration (mobile-first) with OTP, selfie/ID upload, optional ward details, check-in/out, feedback.
  - Verification link (guard-side) to validate or checkout visitors via code/scanner.
  - Gate pass (POC details) and visitor approval previews for links with visitor/branch ids.
  - Guard dashboard for stats, visitor list, and manual entry.
- Central state/context keeps gate settings, visitor details, OTP/checkout states, branding, and feature toggles from branch configuration.

## 2) Folder Structure (relevant)

- `src/helpers-and-constants/visitor/registration-and-verification/`
  - `context-provider.tsx`: shared state store for registration/verification flows.
  - `registration/check-in/*`: public check-in/out steps (MobileNumber, OtpVerification, VisitorDetails, UploadIdProof, SelfieVerification, ParentOrWalkin, WardDetails, ThankYou dialogs, CheckInCheckOut, ManagedByFooter).
  - `verification/*`: guard-facing verification flow (VisitorVerificationComponent, VerifyOrCheckout, VisitorpassScanner, ByScanner, EnterVisitorCode, RequestVerified, VisitorEarly, VerifyOrCheckout, ThankyouPage, VerifyOrCheckout CSS helpers).
  - `public/*`: link-driven pages (PocDetails for gatepass, RequestPreview for approvals).
- `src/helpers-and-constants/visitor/visitor-status/VisitorStatus.tsx`: status chips/badges (not inspected).
- `src/helpers-and-constants/visitor/visitor-entry/visitor-entry-drawer.tsx`: likely drawer UI for entries (not inspected).
- `src/helpers-and-constants/visitor/reports/columns/Columns.tsx`: datagrid column definitions.
- Pages:
  - `src/pages/visitor-registration`: renders VisitorRegistrationMobile.
  - `src/pages/verification-link`: renders VisitorVerificationComponent.
  - `src/pages/gatepass` + `[id].tsx`: POC details, handles visitorid/branchid from query.
  - `src/pages/visitorapproval` + `[visitorid].tsx`: RequestPreview for visitor approval, takes visitorid/branchid.
  - `src/pages/guard`: guard dashboard with stats, visitor list, add-visitor form.
- Note: `src/pages/upload-document` does not exist.

## 3) Component Responsibilities

- `context-provider`: Defines GateState (checkIn/out flags, OTP flags, upload/photo/feedback toggles, visitor details, branding, steps, errors) + MyProvider hook wrappers updateState/resetState.
- `VisitorRegistrationMobile`: Main mobile public flow. Loads branch config (useGetRegistrationVisitorEntryStepsQuery) to set feature flags/theme. Builds step sequence dynamically (CheckInCheckOut → ParentOrWalkin → MobileNumber → VisitorDetails → optional OTP, WardDetails, ThankyouPage, feedback, Selfie/ID uploads, ThankyouCheckout). Submits check-in (useCreateVisitorLogMutation) or checkout (useCreateVisitorCheckoutLogMutation). Manages splash, modal errors, and guard disallow states.
- `VisitorVerificationComponent`: Guard-side verification/checkout. Reads branchid from URL, seeds branding + permissions, steps through VerifyOrCheckout → Scanner choice → EnterVisitorCode → RequestVerified/VisitorEarly. Supports checkoutByGuard mutation (useCreateVisitorCheckoutByGuardMutation) and updates context with visitor details/status codes.
- `PocDetails`: Public gatepass details entry (used by /gatepass).
- `RequestPreview`: Public/approval preview for visitor requests (used by /visitorapproval).
- Guard page (`pages/guard`): Layout with TopStatCards (useGetDashboardQuery), VisitorPanel (list), AddVisitorForm (manual entry).
- Gatepass/approval dynamic pages: decode query params (visitorid/branchid) and validate presence; fallback invalid link message.

## 4) Props + State (key)

- Context GateState fields: checkIn/checkOut, parent/walkin flags, OTP requirements (ischeckinotprequired/ischeckoutotprequired), upload flags (isuploadid/isuploadphoto), feedback/follow-up flags, allowed-by-visitor/guard, scanner toggles, step/gateStep, visitorDetails (photo/timein/visitorid/name/purpose/verified status/toMeetMobile), checkout flags/messages, recurring flags, branding (schoollogo/branchname/theme), counts (noofperson).
- VisitorRegistrationMobile local: showSplash/showLogo/modalOpen, isParent, isMobileBlacklisted, updatedSteps, submittedStepRef, checkoutSubmit, hasSubmitted.
- VisitorVerificationComponent local: showSplash/showLogo/modalOpen, branchid from URL, updatedSteps (fixed array), uses context for gateStep, checkoutByGuardSubmit, checkoutByScanner.

## 5) Data Flow (UI → Logic → UI)

- Public registration:
  1. URL params (branchid/locationid) load branch config (steps + branding).
  2. Context seeded with flags (OTP required? uploads? feedback? allowed? theme/noofperson).
  3. Stepper renders dynamic sequence based on flags and checkIn/checkOut selection.
  4. Input collection (mobile → OTP if required → visitor details → optional ward/selfie/ID → feedback/thankyou).
  5. Submit via useCreateVisitorLogMutation (check-in) or useCreateVisitorCheckoutLogMutation (checkout); success triggers ThankYou/ThankyouCheckout components; errors open modal.
- Guard verification:
  1. Branch config fetched; sets isallowedbyguard, branding, theme; gateStep maybe forced to scanner/code step if visitor disallowed.
  2. Flow chooses verify vs checkout; scanner or manual code; RequestVerified/VisitorEarly shown based on status.
  3. If checkoutByGuardSubmit set and visitorid available → useCreateVisitorCheckoutByGuardMutation; updates visitorDetails, checkouttime, verified status, success message; errors set errorMessage and show modal.
- Gatepass/approval links:
  - Extract visitorid/branchid from query; render PocDetails/RequestPreview; invalid params show inline error.

## 6) API Calls (hooks)

- Registration: `useGetRegistrationVisitorEntryStepsQuery` (branch config/flags/theme), `useCreateVisitorLogMutation` (check-in), `useCreateVisitorCheckoutLogMutation` (checkout).
- Verification: `useGetRegistrationVisitorEntryStepsQuery` (same config), `useCreateVisitorCheckoutByGuardMutation` (guard-driven checkout).
- Guard dashboard: `useGetDashboardQuery`.
- Others likely inside VisitorPanel/AddVisitorForm (not opened here).

## 7) Reusable Components/Helpers

- Step components: CheckInCheckOut, ParentOrWalkin, MobileNumber, OtpVerification, VisitorDetails, UploadIdProof, SelfieVerification, WardDetails, ThankYouDialog, ThankyouPage, ThankyouCheckout.
- Verification components: VerifyOrCheckout, VisitorpassScanner, ByScanner, EnterVisitorCode, RequestVerified, VisitorEarly, RequestPreview, PocDetails.
- Context helpers: `useMyContext`, `updateState`, `resetState`.
- Styling helpers: verification/css.tsx (layout constants), ManagedByFooter reused from registration.
- Reports/Columns: reusable datagrid column config for visitor reports.

## 8) Conditional UI Behavior

- Steps mutate based on API flags:
  - OTP steps inserted for check-in/checkout when required and not recurring.
  - WardDetails inserted for parent with different number.
  - Feedback step added when required; selfie/ID upload steps added when flags enabled.
  - If visitor disallowed (`isallowedbyvisitor === false`), auto-set checkIn true and step min to skip chooser.
- Splash screens: 1s logo, 3s splash before showing main content.
- Guard verification: if isallowedbyguard is false, gateStep jumps; checkoutByScanner flag drives gateStep increment after checkout.
- Invalid query params (gatepass/approval) render inline error message.

## 9) TypeScript Interfaces (key)

- `GateState` (context-provider): holistic state of registration/verification.
- `PersonalDetails`, `WardDetails`, `FeedbackDetails`, `VisitorDetailsState`, `CountrySelect`.
- `VerifiedCode` union (0–6).
- `MyContextType` with setState/updateState/resetState.
- Registration types (typesRegisteration.ts): `ApiError`, `BranchConfigResponse`, `CheckoutPayload`, `CheckoutResponse`, `VisitorData` (used in VisitorRegistrationMobile).
- Verification types: `CreateVisitorCheckoutByGuardResponse` (from service hook).
- Page params: gatepass/visitorapproval dynamic routes expect `visitorid` and `branchid`.

## 10) Styling System

- MUI theme with secondary color (#e64b4c), dynamic theme color from branch config (`theme` or default #2B5796).
- Framer-motion used for splash transitions and step changes.
- Responsive handling via `useMediaQuery('(orientation: landscape)')`.
- Reusable CSS snippet in `verification/css.tsx` (layout constants for backgrounds, wrappers).
- Branding: logo + branch name displayed in headers of verification and registration flows.

## 11) Error Handling

- VisitorRegistrationMobile: if branch config missing/error → modalOpen with ErrorDialog; submit errors trigger dialog; reload on confirm.
- VisitorVerificationComponent: checkout mutation errors set `errorMessage` and show modal; checkoutError flag opens modal; formatDate handles invalid dates safely.
- Dynamic pages show inline “Invalid link: missing visitorid or branchid.”
- Guard dashboard relies on service hooks; no explicit error UI shown in page (likely inside child components).

## 12) Performance Considerations

- useMemo/useRef to avoid rerender loops on steps; step arrays cloned only when flags change.
- Splash timers cleared on unmount.
- SkipToken used to avoid calling verification config without branchid.
- Prevents duplicate submissions via inFlightRef/submittedStepRef (VisitorRegistrationMobile).
- Scroll containers styled with stable scrollbars to avoid layout shift (guard page).

## 13) Future Enhancements

- Centralize validation/disabled states inside step components to reduce modal reliance.
- Add explicit retry/refresh UI when branch config fetch fails (instead of generic modal).
- Expose upload preview/download for stored documents (if persisted).
- Add loading/empty states for VisitorPanel/AddVisitorForm pages if missing.
- Type-safety improvements: replace many `any`/non-null assertions in step components with stricter types.
- Add analytics for step drop-offs and OTP failures.
- Add localization for labels/messages.

## Quick How-Tos

- Public registration: share link with `?branchid=&locationid=` → splash → choose Check-In/Check-Out → flow inserts OTP/ward/upload/feedback steps as required by branch config → submit → Thank You/Checkout message.
- Verification link: `/verification-link?branchid=` → splash → choose verify vs checkout → scan or enter code → see verified/early states; guard checkout auto-calls checkout mutation when flag set.
- Gatepass link: `/gatepass/[id]?branchid=` → PocDetails rendered; shows invalid message if params missing.
- Visitor approval: `/visitorapproval/[visitorid]?branchid=` → RequestPreview shown; inline error if params missing.
- Guard dashboard: `/guard` → stats cards (TopStatCards) + VisitorPanel (list) + AddVisitorForm (manual entry) inside GuardLayout.
