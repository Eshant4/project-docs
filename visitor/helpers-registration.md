# Visitor Registration Flow (Mobile Check-In/Out)

## Core Entry

- `VisitorRegistrationMobile`: orchestrates check-in/out steps based on branch config and context.

## State & Context

- Uses `MyProvider`/`useMyContext` (context-provider.tsx) holding GateState: flags (checkIn/checkOut, OTP requirements, upload/photo, feedback/follow-up), visitor details, branding (schoollogo/branchname/theme), step indices, error messages, recurring flags, counts, scanner/guard toggles.
- Helpers: `updateState`, `resetState`.

## API Hooks

- `useGetRegistrationVisitorEntryStepsQuery({ branchid })`: fetches branch config (theme, OTP/upload/feedback flags, allowed-by-visitor, noofperson).
- `useCreateVisitorLogMutation`: submit check-in payload.
- `useCreateVisitorCheckoutLogMutation`: submit checkout payload.

## Step Orchestration

- Base steps: CheckInCheckOut → ParentOrWalkin → MobileNumber → VisitorDetails.
- Conditional insertions:
  - OTP step (`OtpVerification`) for check-in/checkout when `ischeckinotprequired`/`ischeckoutotprequired` and not recurring.
  - WardDetails when parent with different number.
  - Upload steps (`SelfieVerification`, `UploadIdProof`) when enabled.
  - Feedback/ThankyouPage when `isfeedbackreq`.
  - ThankyouCheckout for checkout completion.
- Splash screen: logo after 1s, main content after 3s.
- Theme color: from API (`theme === "null" ? '#2B5796' : theme`).

## Submission Flow

- Validates steps, guards duplicate submit via refs; builds payload (branchid, locationid, visitor details, optional media), calls mutations; on success shows ThankYou dialog/page, optionally reloads on confirm; errors open modal via ErrorDialog.

## Supporting Components (key)

- `mobile-number.tsx`: capture mobile; may trigger blacklist check (state `isMobileBlacklisted`).
- `otp-verification.tsx`: handles OTP input/verify per flow.
- `visitor-details.tsx`: capture personal details; uses context.
- `UploadIdProof`, `selfie-verification.tsx`: handle file/photo capture when required.
- `ParentOrWalkin`, `WardDetails`: branch logic for parents/wards.
- `CheckInCheckOut`: initial choice of mode.
- `ThankYouDialog`, `ThankyouCheckout`: success messaging.
- `ManagedByFooter`: footer branding.

## Error/Guardrails

- If branch config missing/error → modalOpen with ErrorDialog; reload on confirm.
- If `isallowedbyvisitor === false`: forces check-in true and step minimum to skip choice.
- Handles orientation via `useMediaQuery('(orientation: landscape)')`.

## Future Notes

- Add explicit retry UI for config fetch; improve validations inside each step; surface blacklist reason when set.
