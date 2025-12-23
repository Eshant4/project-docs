# Visitor Verification & Guard Checkout

## Core Entry

- `VisitorVerificationComponent` (GatePass): guard/public verification with scanner/code and guard checkout.

## State & Context

- Shares `MyProvider`/GateState with registration (theme, gateStep, isallowedbyguard, visitorDetails, checkout flags/messages, byScanner/byVisitorCode, errorMessage).

## API Hooks

- `useGetRegistrationVisitorEntryStepsQuery(branchid?)`: loads guard allowances, branding, theme (skipToken when no branchid).
- `useCreateVisitorCheckoutByGuardMutation`: performs guard-initiated checkout.

## Flow

- Reads `branchid` from URL query (window.location.search).
- Sets splash (logo at 1s, content at 3s).
- Seeds state with `isallowedbyguard`, `schoollogo`, `branchname`, `theme`; if not allowed, bumps `gateStep`.
- Steps array: VerifyOrCheckout → VisitorpassScanner (scanner/manual) → EnterVisitorCode → RequestVerified → VisitorEarly.
- Checkout: when `checkoutByGuardSubmit` & `visitorid` present → call mutation; on success updates visitorDetails (photo, times, purpose, verified code=4), checkouttime, gateStep (scanner-aware); on error sets errorMessage and opens modal; always clears submit flag.
- Handles `checkoutError` flag to open modal.
- Uses `checkoutByScanner` to adjust gateStep progression post-checkout.

## Key Components

- `VerifyOrCheckout`: choose verify vs checkout.
- `VisitorpassScanner` / `ByScanner`: scanning path.
- `EnterVisitorCode`: manual code input.
- `RequestVerified`: verified view.
- `VisitorEarly`: early arrival handling.
- `ThankyouPage`: feedback/thank you (also reused in registration).
- `css.tsx`: shared layout styles.
- `VisitorVerificationComponent` wraps in ThemeProvider (secondary color), uses framer-motion for splash transitions.

## Styling/Branding

- Theme color from config (`theme === "null" ? '#2B5796' : theme`), logo/branch name shown in headers (Image with Next.js).

## Error Handling

- Mutation errors: set `errorMessage`, show ErrorDialog via modal.
- Invalid dates guarded by `formatDate` helper (returns "Invalid Date" when bad).
- Splash guards unmounted timers.

## Future Notes

- Add explicit loading/error UI for config fetch; add retry CTA in modal; unify scanner state transitions to avoid manual gateStep tweaks.
