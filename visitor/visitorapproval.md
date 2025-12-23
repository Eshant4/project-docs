# Page: src/pages/visitorapproval (index.tsx & [visitorid].tsx)

## Purpose

- Public approval/preview of visitor requests using `RequestPreview`.

## Routing

- `/visitorapproval`: renders `RequestPreview` without enforced params.
- `/visitorapproval/[visitorid]`: extracts `visitorid` + `branchid`; shows inline “Invalid link” if missing; otherwise `RequestPreview visitorid={pathId} branchid={branchid}`.
- `getLayout` returns plain div; `isInMainLayout` disabled on dynamic route.

## Dependency

- `@/helpers-and-constants/visitor/registration-and-verification/public/RequestPreview`.
