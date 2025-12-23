# Appraisal Module – End-to-End Guide

## 1) Overview
- Covers public appraisal form filling (employees), manager reviews, builder/listing of forms, performance/mock screens, migrations, and preview window.
- Key experiences:
  - Public form: OTP-gated, styled form rendering, multi-step questions, submission with attachments.
  - Manager review: dashboard to pick a form, list employees, review/preview responses.
  - Builder/listing: create/edit/lock/preview forms, manage active status and sharing link.
  - Preview window: standalone postMessage preview for builder.
  - Misc: performance mock UI, migration entrypoints, settings placeholder.

## 2) Folder Structure (relevant)
- `src/helpers-and-constants/appraisal/`
  - `appraisal-helpers/`: public form runtime (AppraisalManagement, AccessVerificationModal, QuestionsList, AppraisalQuestionForm) + styles/constants.
  - `manager/`: manager review helpers (managerHelper, EmpAppraisal, ManagerReviewPane, AppraisalForm, ManagerDummyFormPreview, buildManagerResponses).
  - `forms/`: builder and grid helpers (CreateForm, FormWizardContext, MostUsedForms, renderSubmissionRateCell, formTypes/appraisalMap, customization preview components).
  - `emp-migration/`: migration UIs (EmployeeMigration, Dept/Wing/Scale/Designation helpers).
- `src/pages/appraisal/`
  - `appraisal-forms/`: builder/list page.
  - `manager-review/`: manager dashboard.
  - `performance/`: mock criteria UI + sidebar/drawer placeholders.
  - `employee-migration/`: migration wrapper.
  - `settings/`, `dashboard/`: stubs.
- `src/pages/appraisal-management/`: public form entry (OTP + form render).
- `src/pages/appraisal-form-preview/`: standalone preview window.

## 3) Component Responsibilities (key ones)
- `AppraisalManagement` (public form runtime): merge styles, parse sections, render multi-step questions, validate/submit answers.
- `AccessVerificationModal`: OTP gate (send/validate) before form loads.
- `QuestionsList`: per-question renderer wired to Formik + server attachments.
- `AppraisalQuestionForm`: reusable question renderer for previews/manager view.
- `managerHelper.ExtractformValues`: normalize server form + styles + sections for manager preview/review.
- `EmpAppraisal`: fetch/transform employee rows and render selectable cards.
- `ManagerReviewPane`: capture/save manager responses (uses buildManagerResponses).
- `AppraisalForm` (manager page): toggles between summary preview and review pane.
- `appraisal-forms/index`: list/manage forms; open builder/preview; lock/share.
- `manager-review/index`: pick form → list employees → review/preview responses.
- `appraisal-form-preview/index`: postMessage handshake + ActualFormPreview render.
- `performance/*`: static mock layout (expandable criteria) + sidebar/drawer shells.
- `employee-migration/index`: wraps EmployeeMigration UI.
- `settings/index`, `dashboard/index`: placeholders.

## 4) Props + State (high-level)
- AppraisalManagement: `form`, `formid`, `responderId`, `currentRole`, `formLocked`, `uiTypeResolver`, `visibleForRole`, `onSubmitPayload`; state for `currentStep`, `showFormBody`, `showSuccess`, `stepDirection`, Formik state.
- AccessVerificationModal: `open`, `onVerified`, `branchid`, `formid`, branding styles; state for `stage (phone|otp)`, timers, otp digits, errors.
- Manager-review page: `reviewBtn`, `formSelected`, `pickedRow` (form), `empPicked` (employee), stats derived from API.
- Appraisal-forms page: `showNewForm`, `template`, `isEditingForm`, `employeePreview`, `rows`, `selectedRowData`, popper anchors for lock/copy.

## 5) Data Flow (UI → Logic → UI)
- Public form:
  - URL params (formid/branchid) → OTP modal (sendOtp/validateOtp) → eid set → fetch form styles + active status → fetch form details → AppraisalManagement builds styles/sections/initial Formik → user answers → validation → build responses + FormData (uploads) → submit → success dialog; locked/submitted alters footer buttons.
- Manager review:
  - Dashboard stats → select form (manager form data) → EmpAppraisal fetch employees → select employee → AppraisalForm merges question metadata → ManagerReviewPane handles manager inputs → submit via buildManagerResponses.
- Builder/list:
  - Fetch forms list → grid renders status chips/toggles → actions open CreateForm (edit/new) or AppraisalFormEmployee preview; lock popper updates lock state; copy link copies access URL.
- Preview window:
  - Opens child window → sends READY → parent posts APPRAISAL_PREVIEW_DATA → child renders ActualFormPreview.

## 6) API Calls (named hooks)
- Public: `useGetFormStylesAndLogoQuery`, `useGetActiveOrInactiveQuery`, `useGetPublicformDetailsQuery`, `useSendOtpMutation`, `useValidateOtpMutation`, `useResponseSubmitMutation`.
- Manager: `useGetManagerDashboardQuery`, `useGetManagerFormDataQuery`, employee data via `useLazyGetEmployeeDataQuery`.
- Builder/list: `useGetAppraisalFormQuery`, `useDeleteAppraisalFormMutation`, `useActiveFormMutation`, `useLockFormMutation`.
- Payload helpers: `buildManagerResponses` (manager submit).

## 7) Reusable Components/Helpers
- Chips/formatters: `renderChipCell`, `renderSubmissionRateCell`, `renderFormStatusCell`.
- Style defaults/constants: `DEFAULT_FORMSTYLES`, layout styles in appraisal-helpers/rows.tsx.
- Section parsing: `parseFormSections` (both public and manager), `getSectionImagePreview` (builder).
- Question UI: `QuestionsList`, `AppraisalQuestionForm`, `RatingScaleSetup`, `ProfileMiniRow`.
- Context/hooks: `FormWizardContext`, `useRegisterWithWizard` (builder).

## 8) Conditional UI Behavior
- Public form:
  - Locked/submitted → disables inputs; footer button shows Locked/Print.
  - showFormBody false → instructions only; button to start/see responses.
  - Section navigation animates with stepDirection; hides footer when loading/error.
  - Upload questions disabled when locked; server attachments shown via serverAttachments map.
- AccessVerificationModal:
  - Stage swap phone→otp; resend enabled after timer; error dialog blocks flow.
- Manager review:
  - Review button toggles panes; width adjusts; Preview vs Review label depends on managerreviewed flag.
- Builder grid:
  - Active Switch toggles isactive; lock popper options disabled unless matching current lock state; copy closes parent popover.

## 9) TypeScript Interfaces (key)
- Public: `Question`, `AppraisalFormDto`, `SubmitAnswer`, `SubmitPayload`, `Values` (answers map), `UIQuestionType`.
- Manager: `EmployeeAppraisal` (+ Raw), `QuestionBankItem` (AppraisalQuestionForm), `ManagerForm` (from ExtractformValues).
- Styles: `HeaderSettings`, `instructionSettings`, `questionSettings`, `scaleSettings`, `buttonSettings`, `FormStyles` (merged from defaults + server).

## 10) Styling System
- MUI SX throughout; typography/colors driven by merged `formstyle` from server over `DEFAULT_FORMSTYLES`.
- Header background supports color or image with overlay opacity; alignment controls for header/questions/buttons.
- Print styles via `globalPrintStyles` (hides footer on print, forces color adjust).
- Chips use status-based colors; grids use consistent padding/borders.

## 11) Error Handling
- AppraisalManagement: status switches to error → shows message; submit catch logs; form locked state prevents submit.
- AccessVerificationModal: OTP/API errors surfaced in UI; Access Denied dialog with Retry reload.
- Appraisal-forms page: RTK errors parsed via parseRTKError; toasts for mutations; confirm dialog on delete.
- Manager-review: loading/error fallbacks in ExistingPicker; form load failure shows basic messaging.
- Preview window: tries to close on bad context; shows timeout hint after 10s without data.

## 12) Performance Considerations
- useMemo for question sorting, section parsing, merged styles; React.memo around some layouts.
- Formik initialized once; reset when question ids change; validation on change disabled to reduce churn.
- Scroll containers use stable scrollbars (scrollbarGutter) to avoid layout shift.
- Preview window is SSR-disabled to avoid hydration work.

## 13) Future Enhancements
- Add real data + mutations for performance/AddCriteria drawer (currently stubbed).
- Complete migration tabs (Dept/Wing/Scale) in page-level UI.
- Add draft-saving/autosave for public form and manager review.
- Add upload preview/download support for server attachments in manager view.
- Improve error surfaces in AppraisalManagement (e.g., retry CTA for fetch errors).
- Expand builder docs/tests for new question types and validation rules.

## Quick How-To (flow summaries)
- Public form: Access link → OTP → form loads → “Start filling” → navigate sections → Submit → success dialog.
- Manager review: Manager dashboard → select form → pick employee → click Review → fill manager responses → submit; Preview toggles if already reviewed.
- Builder/list: Appraisal Forms page → “Create New” or Edit → wizard flow → Save; lock/activate via grid; copy link for sharing.
- Preview window: From builder click Preview → child window opens and waits for READY/ DATA handshake → renders styled form preview.
