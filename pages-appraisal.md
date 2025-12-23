# Appraisal Pages (src/pages/appraisal)

## appraisal-forms/index.tsx
- Lists all appraisal forms with actions (preview, edit, lock, copy link).
- Data: useGetAppraisalFormQuery -> rows augmented with public link (https://jusklik.com/appraisal-management?{accesslink}).
- Mutations: useDeleteAppraisalFormMutation, useActiveFormMutation (toggle Switch), useLockFormMutation (lock employees/managers/both via secondary Popper).
- Popovers: ReusablePopup (copy link + open lock popper); lock presets are enforced to match current lock state (only matching option enabled).
- Modals/views:
  - CreateForm (builder) opened via "Create New"/Edit/template flows.
  - AppraisalFormEmployee: employee-style preview of selected form.
  - MostUsedForms: template chooser; passes template flags into CreateForm.
- Grid columns: form name chip with avatar color, Active Switch, Created/Modified dates, Submission Rate (renderSubmissionRateCell), Action icons (Preview/Edit/More).
- Clipboard: navigator.clipboard used to copy public link.

## manager-review/index.tsx
- Manager dashboard for reviewing employee submissions.
- TopStatCards from useGetManagerDashboardQuery.
- ExistingPicker: fetches manager-visible forms (useGetManagerFormDataQuery), shows cards with status chip (renderChipCell('submitted'|'pending')), submission rate bar, last modified (formatDate).
- When a form is selected: left pane (EmpAppraisal) lists employees; right pane (AppraisalForm) shows either summary or review mode. Review button toggles reviewBtn state and swaps layout width.
- AppraisalForm merges question metadata from ExtractformValues to show section/part labels and images; ManagerReviewPane handles manager responses.

## performance/*
- index.tsx: Layout wrapper that shows Sidebar + CustomHeaderwithDataGrid with Performance component in the grid slot; opens DrawerLayOut with AddCriteriaDrawer (placeholder).
- Performance.tsx: static mock data for criteria list with expandable rating details; controls expand/collapse with framer-motion; uses RenderChipCell (local) for frequency chips.
- Sidebar.tsx: step rail UI (Form Settings, Access Control) with dot animation; purely visual.
- AddCriteriaDrawer.tsx/HelperFiles.tsx: stubs for future drawer content.

## employee-migration/index.tsx
- Wrapper around helpers-and-constants/appraisal/emp-migration/EmployeeMigration inside a Paper. Tab scaffolding for other migrations is commented.

## settings/index.tsx
- Renders ReusableSettings layout wrapper (no appraisal-specific logic).

## dashboard/index.tsx
- Placeholder component returning “index”; no logic yet.
