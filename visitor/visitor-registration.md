# Page: src/pages/visitor-registration/index.tsx

## Purpose
- Public/mobile-first visitor registration and checkout flow.

## Routing/Layout
- Path: `/visitor-registration`; component reads query params (branchid, locationid) internally.
- `getLayout` returns plain div (no main layout).

## Dependency
- `VisitorRegistrationMobile` from registration helpers.
