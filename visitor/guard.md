

# Page: src/pages/guard/index.tsx

## Purpose
- Guard dashboard showing stats, live visitor list, and manual add form within `GuardLayout`.

## Structure
- TopStatCards fed by `useGetDashboardQuery(0)`; items fallback to 0/placeholder.
- Two columns (responsive): 
  - Left: VisitorPanel (scrollable) listing visitors.
  - Right: AddVisitorForm (scrollable) for guard to add a visitor.
- Scroll areas use stable scrollbars to avoid layout shift.
- Wrapped in `GuardLayout`; `Index.enableRedirect = true`.

## Key Dependencies
- Icons (check-in/out, parent), `TopStatCards`, `VisitorPanel`, `AddVisitorForm`, MUI Grid/Card.
- API: `useGetDashboardQuery`.

## Behavior
- Memoizes stats array for safety; applies background styling and padding.
- No explicit error/loading handling here; assumed inside child components.
