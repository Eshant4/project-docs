# Appraisal Helpers & Constants

## Folder Map
- appraisal-helpers/: runtime form rendering (public employee experience) + shared UI helpers.
- manager/: manager-side review utilities and previews.
- forms/: form builder UI (list/create/edit) + preview utilities for builders.
- emp-migration/: data migration helpers (employees, dept, wing, designation, scale).
- forms/steps/form-customization/: styling knobs and preview scaffolding.

## Public Employee Form Flow (appraisal-helpers/)
- AppraisalManagement.tsx
  - Props: form (AppraisalFormDto | null), formid, responderId, currentRole ('employee' default), uiTypeResolver, visibleForRole, formLocked, onSubmitPayload.
  - Builds merged form styles (DEFAULT_FORMSTYLES + server styles + inline logos/backgrounds).
  - Sorts and filters questions, parses sections (parseFormSections) and section images, builds initial Formik state via buildInitialValues.
  - Validation per question type (scale/short/long/upload) using Yup; upload limits: maxUploads = attachmentsallowed (>=1), maxFileSizeMB = 23.
  - Multi-step navigation: sections derived from server; tracks step direction for animations; showFormBody toggled after instruction block.
  - Submission: validates all, builds responses payload (text/rating + FormData for uploads), calls useResponseSubmitMutation, optionally calls onSubmitPayload, shows PostSubmitDialog on success.
  - Handles locked/submitted states (button label changes to Print/Locked), print triggers window.print.
  - Layout pieces: header (logo/title/background), personal info (ProfileMiniRow), instructions (RatingScaleSetup), questions (QuestionsList), footer buttons (Previous/Continue/Submit).
- AccessVerificationModal.tsx
  - Two-stage OTP gate for public forms.
  - Stage phone: validates 10-digit mobile, calls useSendOtpMutation; Stage OTP: four single-digit inputs; timer with resend after 59s; on successful validateOtp -> calls onVerified({eid}).
  - Accepts brand theming (headerStyle/buttonStyle, logos) and shows Access Denied message if OTP send fails.
- QuestionsList.tsx (not in pages but used by AppraisalManagement)
  - Renders per-question UI based on resolveUIType (short/long/scale/upload), supports read-only mode and server attachments, forwards Formik handlers.
- AppraisalQuestionForm.tsx
  - Standalone question renderer for previews/manager view; normalizes question types; supports manager prefills, custom upload renderer, slider scaleType, optional footer.
- rows.tsx
  - Types: Question, AppraisalFormDto, SubmitPayload, UIQuestionType; DEFAULT_FORMSTYLES.
  - Helpers: renderChipCell/renderFormStatusCell chips, parseFormSections (part/section parsing), buildInitialValues, globalPrintStyles, box/layout style constants.

## Manager Review Helpers (manager/)
- managerHelper.tsx
  - ExtractformValues(formValues) -> ManagerForm: merges styles (same defaults as public), parses sections (parseFormSections from forms/appraisalMap), maps questionbank with section/part metadata and preview images, and builds primaryData (emp name/code/dept/doj).
- EmpAppraisal.tsx
  - Fetch/transform employee appraisal rows via useLazyGetEmployeeDataQuery; maps flat question rows into grouped EmployeeAppraisal records with sorted questionbank.
  - Renders selectable cards (status chip, meta tags) with pagination; setEmpPicked gets the selected EmployeeAppraisal.
- AppraisalForm.tsx
  - Dual view: summary card (employee info + ManagerDummyFormPreview) when reviewBtn=false; review mode shows AppraisalQuestionForm (read-only) alongside ManagerReviewPane for manager inputs.
  - Merges question metadata with manager-specific section info before rendering.
- ManagerReviewPane.tsx / buildManagerResponses.ts
  - Capture/save manager ratings/text; buildManagerResponses adapts manager answers to API payload (see file for exact field mapping).
- ManagerDummyFormPreview.tsx
  - Lightweight preview of form styles/questions for manager context (no submission).

## Form Builder Helpers (forms/)
- appraisalMap.ts + steps/formTypes.ts
  - Definitions for AppraisalForm structure used by builder and preview; section mapping utilities (getSectionImagePreview, SectionGroup).
- CreateForm.tsx + FormWizardContext.tsx + useRegisterWithWizard.ts
  - Builder wizard orchestration; handles step state, field registration, template creation/edit flow.
- MostusedForms.tsx / SidebarSteps.tsx / StepCard.tsx / Step components (StepOne..StepSix, right/steps folders)
  - UI for choosing templates, navigating wizard steps, configuring form content (sections, questions, styles, access control).
- Fields (FieldRow, FormikCustomTextField, FormikSelectWithIcons, VisibilityCheckboxGroup)
  - Formik-connected inputs for builder steps.
- Preview/customization (steps/form-customization/*.tsx)
  - Live preview and style controls: DummyAppraisalPage (ActualFormPreview), HeaderSettingsPanel, ColorFontControls, QuestionsSettingsPannel, ButtonSettingsPannel, etc.
- rows.tsx (forms folder)
  - renderSubmissionRateCell helper for datagrids; shared chip renderer used in manager pages.

## Emp Migration (emp-migration/)
- EmployeeMigration.tsx + helpers (DeptMigration, WingMigration, ScaleMigration, DesignationMigration)
  - UI/workflows for migrating employee-related masters; common styles in common.ts; invoked by pages/appraisal/employee-migration.

## Notes for Extending
- Styles: always merge with DEFAULT_FORMSTYLES to avoid missing keys; headerBgOpacity logic in AppraisalManagement uses rgba overlay when headerBgMode==='image'.
- Sections: parseFormSections expects a nested object { "Part 1": { "Section A": [1,2] } }; indices are (sortorder+1).
- Attachments: Formik stores upload answers as File[]; server attachments are mapped via serverAttachments to display existing files in read-only mode.
- Validation: adjust Yup rules in AppraisalManagement when adding new question types; mapQuestionType controls UI mapping.
