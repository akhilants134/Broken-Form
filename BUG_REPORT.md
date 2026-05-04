# BUG_REPORT

This document lists the six bugs found in the original `src/App.jsx`, their root causes, and what I changed to fix them.

## Reproduction summary (before fixes)

- Submitting the form with empty fields succeeds and creates empty bug reports.
- Clicking Submit rapidly fires multiple requests; the button never disabled.
- After a successful submit the form fields were not cleared.
- Submitting a title containing `login` produced a server error that disappeared silently.
- Leaving `Title` empty displayed no inline error message next to the field.
- Entering `0` or negative numbers for `No. of Steps` was accepted.

## Bugs & Root Causes

1. Empty Submission

- Root cause: `validate()` returned `true` (no checks) and its result was ignored in `handleSubmit`. Submission proceeded unconditionally.

2. Double Submission

- Root cause: `loading` state was not set before the API call and the submit button did not read `loading` to disable itself.

3. Form Not Reset

- Root cause: After a successful API response, `setForm(EMPTY_FORM)` was never called so the form retained previous values.

4. Silent Server Error

- Root cause: The `catch` block swallowed errors (no routing to field-level errors or server banner). Structured errors returned from `api.js` were ignored.

5. No Field-Level Messages

- Root cause: Although `errors` state existed, the JSX did not reference or render `errors.<field>` for inputs, so users saw no inline messages.

6. Invalid Steps Count

- Root cause: No validation rule enforced that `stepsCount` must be a positive integer; negative/zero values bypassed checks.

## Fixes Implemented

- Implemented `validate(data)` which returns an errors object for: `title`, `severity`, `component`, `description`, and `stepsCount` (must be positive integer).
- `handleSubmit` now gates on `validate()`, sets `loading = true` before the `await`, resets `loading` in `finally`, and resets the form on success.
- `catch` inspects structured server errors (`err.field`) and routes them to `setErrors`, otherwise sets `serverError` to show a top banner.
- Inputs and selects now read `errors` and render a small inline message and visual indicator (border color) when present.
- `handleChange` clears field-level errors for the edited field and clears top-level `serverError`.

## Manual verification steps

1. Start app and open the form.
2. Submit with empty fields → the form now prevents submission and shows inline errors.
3. Fill fields and click Submit twice quickly → button disables after first click and shows `Submitting…`.
4. On successful submit the form clears and success banner shows ID.
5. Submit a title containing `login` (mock server returns 409) → the returned structured error appears next to the `Title` field.
6. Enter `-3` for `No. of Steps` → validation blocks submission and shows an inline error.

## Live URL

- (Add deployed URL here)

---

Commit: "Fixed all 6 form bugs: validation, loading state, error handling, form reset"
