# Overview

NexaFlow is a workflow automation SDK that lets you define and run multi-step business processes as code. It abstracts queue management, retry logic, distributed tracing, and observability so your application only needs to describe *what* should happen and *when*.

---

## Architecture

NexaFlow runs as a lightweight client in your application. Workflow definitions are stored in NexaFlow's cloud control plane, and execution state is managed by the NexaFlow runtime — not your database.

```
Your application
     │
     │  defineWorkflow / run / pause / resume
     ▼
NexaFlow SDK (client library)
     │
     │  HTTPS (REST + WebSocket)
     ▼
NexaFlow Control Plane
     │
     ├── Workflow Engine  ─── executes steps in order
     ├── Trigger Router   ─── listens for events and schedules
     ├── Retry Controller ─── manages backoff and dead-letter queues
     └── Audit Logger     ─── writes structured execution history
```

The SDK never runs steps directly. It sends your workflow definition to the control plane and receives execution results back. This means:

- Step failures don't crash your application process
- Long-running workflows (hours, days) are supported without holding open connections
- Execution history persists even if your application restarts

---

## When to use NexaFlow

NexaFlow is well-suited for:

- **Post-event automation** — sending emails, provisioning resources, or updating records after a user action
- **Scheduled jobs** — nightly data exports, weekly digests, billing cycle processing
- **Approval workflows** — multi-step processes that require human review before proceeding
- **Retry-sensitive integrations** — calling third-party APIs that have rate limits or occasional failures

NexaFlow is not designed for:

- Sub-millisecond latency requirements (use a message bus)
- Streaming pipelines (use Apache Kafka or similar)
- In-process computation chains (use a task runner or Promise chain)

---

## Key concepts

| Concept | What it is |
|---|---|
| **Workflow** | A named, versioned definition of a process — triggers, steps, and branches |
| **Trigger** | The condition that starts a workflow — an event, a schedule, or a manual call |
| **Step** | One unit of work within a workflow — calls an action with parameters |
| **Action** | A built-in or custom integration (database query, HTTP call, email send, etc.) |
| **Run** | A single execution of a workflow, with its own context and state |
| **Run context** | The data passed into a run and the accumulated outputs of its steps |

These concepts are explained in detail in [Core Concepts](core-concepts.md).

---

## SDK versions

| SDK version | Control plane API version | Support status |
|---|---|---|
| 2.x (current) | v2 | Active |
| 1.x | v1 | Security fixes only — EOL December 2026 |

If you're on SDK v1, see the [Migration Guide](migration-guide.md) for the upgrade path.
