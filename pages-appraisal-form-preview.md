# Appraisal Form Preview Window (pages/appraisal-form-preview)

## Purpose
- Opens as a separate window from the builder to show a live preview of AppraisalForm payloads.

## Handshake & Messaging
- On mount: sets mounted gate to avoid hydration issues.
- Listener: window.addEventListener('message'); expects { type: 'APPRAISAL_PREVIEW_DATA', payload: AppraisalForm } from the opener with matching origin; stores lastEvent info for debugging.
- Ready ping: once mounted, sends postMessage({ type: 'APPRAISAL_PREVIEW_READY' }, window.location.origin) to window.opener so the builder knows to send data.
- Auto-close: if opened via reload or no opener, attempts window.close().
- Timeout UX: after 10s without formValues, shows helper text asking to click preview again.

## Rendering
- While waiting: shows “Opening preview…” + Loader; optional timeout message.
- When formValues present: renders ActualFormPreview (DummyAppraisalPage) with styles from formValues.formstyles and hardcoded themeColor/fontFamily overrides.

## Usage Notes
- SSR disabled via dynamic(() => Promise.resolve(Index), { ssr: false }).
- NextPageWithLayout keeps layout minimal (<div> wrapper).
