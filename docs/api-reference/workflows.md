# Workflows

This reference covers the `WorkflowDefinition` schema and the methods available on a `WorkflowInstance`.

---

## WorkflowDefinition

Pass a `WorkflowDefinition` object to `nf.defineWorkflow()`.

```ts
interface WorkflowDefinition {
  trigger: TriggerDefinition;
  steps: (StepDefinition | ParallelBlock)[];
  deadLetterQueue?: DeadLetterQueueConfig;
  tags?: Record<string, string>;
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `trigger` | TriggerDefinition | Yes | What starts a run — see [Triggers](triggers.md) |
| `steps` | array | Yes | Ordered list of steps and parallel blocks |
| `deadLetterQueue` | DeadLetterQueueConfig | No | DLQ settings — see [Error Handling](../error-handling.md#dead-letter-queues) |
| `tags` | object | No | Key-value pairs for filtering and search in the dashboard |

---

## StepDefinition

```ts
interface StepDefinition {
  id: string;
  action: string;
  params?: Record<string, unknown>;
  if?: string;
  retry?: RetryPolicy;
  timeoutMs?: number;
  onError?: OnErrorConfig;
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | Yes | Unique step identifier within the workflow |
| `action` | string | Yes | Name of the action to execute (e.g., `'http.post'`) |
| `params` | object | No | Parameters passed to the action; supports `{{ }}` templates |
| `if` | string | No | Template expression; step runs only when this evaluates to `true` |
| `retry` | RetryPolicy | No | Retry configuration — see [Error Handling](../error-handling.md#retry-policies) |
| `timeoutMs` | integer | No | Per-step timeout in milliseconds |
| `onError` | OnErrorConfig | No | Fallback action to run if this step fails |

---

## ParallelBlock

Wrap multiple steps in a `parallel` block to run them concurrently.

```ts
interface ParallelBlock {
  parallel: StepDefinition[];
}
```

The workflow waits for all steps in the block to complete (or fail) before proceeding to the next item in the sequence. If any parallel step fails, the parallel block fails.

---

## WorkflowInstance methods

`nf.defineWorkflow()` returns a `WorkflowInstance`.

### `.register()`

Registers the workflow definition with the NexaFlow control plane. Creates a new version if the workflow already exists.

```ts
workflow.register(): Promise<RegisterResult>
```

**Returns:**

```ts
{
  workflowId: string;
  version: number;
  registeredAt: string;   // ISO 8601
}
```

### `.run(context, options?)`

Triggers a synchronous run and waits for it to complete. Use this during development and testing; for production, prefer event-driven or schedule triggers.

```ts
workflow.run(
  context: Record<string, unknown>,
  options?: RunOptions
): Promise<RunResult>
```

**RunOptions:**

| Option | Type | Default | Description |
|---|---|---|---|
| `timeoutMs` | integer | `60000` | How long to wait for the run to complete before throwing a timeout error |
| `waitForCompletion` | boolean | `true` | Set to `false` to return immediately after the run is queued |

**RunResult:**

```ts
{
  runId: string;
  status: 'completed' | 'failed' | 'cancelled';
  stepOutputs: Record<string, unknown>;   // keyed by step ID
  startedAt: string;
  completedAt: string;
  durationMs: number;
}
```

### `.pause(runId)`

Pauses an in-progress run before it advances to the next step. Useful for implementing human-in-the-loop approval flows.

```ts
workflow.pause(runId: string): Promise<void>
```

### `.resume(runId, data?)`

Resumes a paused run, optionally injecting additional data into the run context.

```ts
workflow.resume(
  runId: string,
  data?: Record<string, unknown>
): Promise<void>
```

### `.versions()`

Returns the version history for this workflow.

```ts
workflow.versions(): Promise<WorkflowVersion[]>
```

```ts
interface WorkflowVersion {
  version: number;
  registeredAt: string;
  registeredBy: string;   // User or service account ID
  activeRunCount: number; // Runs currently executing this version
}
```
