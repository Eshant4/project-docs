# Page: src/pages/gatepass (index.tsx & [id].tsx)

## Purpose

- Serve public gatepass/POC flow using `PocDetails` from visitor helpers.

## Routing

- `/gatepass`: renders `PocDetails` without params (assumes `PocDetails` reads its own query).
- `/gatepass/[id]`: extracts `id` and `branchid` from query; if missing, shows inline “Invalid link” message; otherwise renders `PocDetails id={pathId} branchid={branchid}`.
- Both disable main layout via `getLayout`.

## Dependencies

- `@/helpers-and-constants/visitor/registration-and-verification/public/PocDetails`.

## Notes

- Ensure links always include both `id` and URL-encoded `branchid`.
- Add error UI inside `PocDetails` if server fails or params invalid.
