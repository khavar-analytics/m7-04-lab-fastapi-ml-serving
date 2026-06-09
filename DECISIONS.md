# API Design Decisions

## 1. Versioning

Path-based versioning (`/v1/`) was chosen because the version is immediately visible in the URL, making it easy to test in a browser, share in documentation, and use without any special client configuration. Header-based versioning was rejected because it is invisible by default and requires clients to remember to send an extra header on every request.

## 2. Batch Ordering and Partial Failures

When `predict-batch` is called with 32 items and one image is corrupt, the API still processes all remaining images and returns results for the valid ones. The corrupt image entry in the response will contain an error object instead of labels, keyed by its caller-supplied `id`, so the caller knows exactly which image failed. This partial-success approach was chosen because failing the entire batch due to one bad image would be wasteful and force callers to re-submit 31 valid images.

## 3. Async Lifecycle

A job moves through four states: `pending` (accepted, not yet picked up) → `processing` (model is running) → `complete` (result ready) or `failed` (model error). Results are retained for 24 hours after the job reaches a terminal state (`complete` or `failed`), after which the job ID returns 404. A 24-hour window was chosen to give partner systems enough time to retrieve results even if their polling is delayed, without holding data indefinitely.
