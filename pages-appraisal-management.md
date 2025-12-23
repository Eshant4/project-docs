# Public Appraisal Form (pages/appraisal-management)

## URL Contract

- Expects query params in URL search (not router query): branchid, formid. Parsed via usePublicToken (reads window.location.search).

## Data Fetching

1. useGetFormStylesAndLogoQuery({formid}) – skipped until branchid/formid ready. Supplies header/button styles, branchname/logo.
2. useGetActiveOrInactiveQuery({formid, branchid}) – determines if form is active; controls gate. activeLoading guards UI.
3. useGetPublicformDetailsQuery({formid, branchid, eid}) – fetched only after OTP verification (verified && eid). Returns AppraisalFormDto to render.

## OTP Gate

- AccessVerificationModal shown until verified.
- onVerified sets eid + verified; form fetch then proceeds.

## Rendering

- If active + verified: AppraisalManagement renders full form with provided formstyle/logo; passes formLocked=false, responderId=eid.
- If inactive: shows Backdrop with MotionPaper, branch branding (logo/title), error message (from activeErrorData.message), and Retry button (reload page). Detects “opens soon” messaging via prefix check on errorMessageActive.

## UX Notes

- Uses Next.js layout override: Index.getLayout = (page) => <div>{page}</div>; not wrapped in main site shell.
- Handles loading state while checking active flag to avoid flashing “Inactive”.
- Branch name truncation logic for long names (affects font size).
