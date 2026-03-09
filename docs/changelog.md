# Changelog

All notable changes to the NexaFlow SDK are documented here. The SDK follows semantic versioning.

---

## v2.3.0 — March 2026

### Added
- **`data.transform` action** — apply JSONata expressions to reshape data inline without a custom action
- **`workflow.trigger` action** — start a child workflow from a step; supports `waitForCompletion`
- **Python 3.12 support** — explicitly tested and supported
- **`nf.runs.replay()` context overrides** — pass `contextOverrides` to modify run data before replaying a failed run

### Fixed
- Fixed a race condition where `nf.events.on('run.completed')` could fire before `workflow.run()` resolved in high-latency environments
- Fixed `db.query` returning incorrect `rowCount` when using connection pooling with more than 10 connections
- Corrected TypeScript types for `WorkflowVersion.registeredBy` (was `number`, should be `string`)

### Changed
- `nf.runs.list()` now defaults to sorting by `startedAt` descending; previously returned results in an unspecified order

---

## v2.2.0 — January 2026

### Added
- **Parallel blocks** — run multiple steps concurrently using the `parallel` array syntax
- **`nf.runs.cancel()`** — cancel queued or running workflows programmatically
- **Schedule timezone support** — set a timezone on cron triggers using the IANA timezone database
- **`workflow.pause()` and `workflow.resume()`** — implement human-in-the-loop approval flows

### Fixed
- Fixed `http.post` with `contentType: 'form'` not correctly encoding nested objects
- Fixed `email.send` to correctly handle `to` as an array of addresses in the Python SDK

---

## v2.1.0 — November 2025

### Added
- **Dead-letter queues** — configure DLQ on any workflow; inspect and replay failed runs from the dashboard or via `nf.runs.replay()`
- **Webhook HMAC verification** — NexaFlow now enforces signature verification on webhook triggers when a `secret` is set
- **Jitter backoff strategy** — available in retry policies to reduce thundering-herd effects
- **Custom HTTP agent support** — pass `httpAgent` to the constructor for proxy and mTLS environments
- **`nf.events.emit()` return value** — now returns `triggeredRunIds` for traceability

### Fixed
- Fixed exponential backoff not respecting `maxDelayMs` for delays beyond 60 seconds

---

## v2.0.0 — September 2025

### Breaking changes

See the [Migration Guide](migration-guide.md) for the full upgrade path.

- Constructor now accepts an options object (`new NexaFlow({ apiKey })`) instead of a bare key string
- `workflow.execute()` renamed to `workflow.run()`
- Step outputs moved to `result.stepOutputs` namespace
- `nf.emit()` moved to `nf.events.emit()`
- `onError` callbacks replaced with declarative `onError` action blocks
- `workflow.deploy()` renamed to `workflow.register()`
- API keys now require explicit scopes for write operations

### Added
- Full Python SDK (previously experimental)
- `testing.simulateFailure()` for testing retry and error handling
- Structured error codes on all SDK errors
- Webhook trigger type
- Service account documentation and dashboard support

---

## v1.4.2 — July 2025 (final v1 release)

### Fixed
- Security: addressed a path traversal vulnerability in the local file action (removed in v2)
- Fixed memory leak when subscribing to `run.completed` events with a long-lived client

### Notes
v1 is now in security-only maintenance mode. EOL is December 2026. All new development is on v2.

---

## v1.4.0 — May 2025

### Added
- `retry.retryOn` option to selectively retry specific error codes
- `step.timeoutMs` per-step timeout configuration

---

## v1.0.0 — October 2024

Initial public release.

- Event and schedule triggers
- HTTP, email, Slack, and database built-in actions
- Custom action support
- TypeScript declarations
- Execution history API
