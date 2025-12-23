# Page: src/pages/verification-link/index.tsx

## Purpose

- Entry for guard/public verification flow. Renders `VisitorVerificationComponent`.

## Routing/Layout

- Path: `/verification-link`; no params required here, component reads `branchid` from window.location.
- `getLayout` returns plain div (no main layout).

## Dependency

- `@/helpers-and-constants/visitor/registration-and-verification/verification/VisitorVerificationComponent`.
