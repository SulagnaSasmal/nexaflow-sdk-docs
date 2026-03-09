# Client

The `NexaFlow` class is the entry point for all SDK operations.

---

## Constructor

```ts
new NexaFlow(options?: ClientOptions): NexaFlow
```

### ClientOptions

| Option | Type | Required | Default | Description |
|---|---|---|---|---|
| `apiKey` | string | No | `process.env.NEXAFLOW_API_KEY` | Your NexaFlow API key |
| `environment` | `'production'` \| `'test'` | No | Inferred from key prefix | Target environment |
| `baseUrl` | string | No | `https://api.nexaflow.dev` | Override the control plane URL (for on-premises deployments) |
| `timeoutMs` | integer | No | `10000` | Default request timeout for API calls |
| `httpAgent` | http.Agent | No | — | Custom HTTP agent (for proxy or mTLS) |
| `logger` | Logger | No | `console` | Custom logger passed an object with `debug`, `info`, `warn`, `error` methods |

**Example:**

```js
const nf = new NexaFlow({
  apiKey: process.env.NEXAFLOW_API_KEY,
  timeoutMs: 5000,
});
```

---

## Methods

### `nf.ping()`

Checks connectivity to the NexaFlow control plane.

```ts
nf.ping(): Promise<PingResult>
```

**Returns:**

```ts
{
  ok: boolean;
  region: string;       // Control plane region (e.g., 'us-east-1')
  latencyMs: number;    // Round-trip latency in milliseconds
}
```

---

### `nf.defineWorkflow(id, definition)`

Defines a workflow and returns a `WorkflowInstance`. Does not register the workflow with the control plane until you call `.register()`.

```ts
nf.defineWorkflow(id: string, definition: WorkflowDefinition): WorkflowInstance
```

See [Workflows](workflows.md) for the full `WorkflowDefinition` schema.

---

### `nf.defineAction(name, handler)`

Registers a custom action handler for the current client instance.

```ts
nf.defineAction(name: string, handler: ActionHandler): void
```

```ts
type ActionHandler = (
  params: Record<string, unknown>,
  context: RunContext
) => Promise<Record<string, unknown>>;
```

Custom actions must be registered before triggering any workflow that uses them.

---

### `nf.events.emit(name, payload)`

Fires a named event, which can trigger any registered workflow with a matching event trigger.

```ts
nf.events.emit(name: string, payload: Record<string, unknown>): Promise<EmitResult>
```

**Returns:**

```ts
{
  eventId: string;
  triggeredRunIds: string[];  // IDs of runs started by this event
}
```

---

### `nf.events.on(name, handler)`

Subscribes to a NexaFlow system event. System events are distinct from workflow trigger events.

```ts
nf.events.on(name: string, handler: EventHandler): Unsubscribe
```

**Available system events:**

| Event name | When it fires |
|---|---|
| `run.started` | A new run has been created and is queued |
| `run.completed` | A run finished all steps successfully |
| `run.failed` | A run failed after exhausting retries |
| `run.cancelled` | A run was manually cancelled |
| `step.completed` | A single step completed successfully |
| `step.failed` | A single step failed |

**Example:**

```js
const unsubscribe = nf.events.on('run.failed', (event) => {
  console.error(`Run ${event.runId} failed on step ${event.failedStepId}`);
});

// Remove the listener when no longer needed
unsubscribe();
```

---

### `nf.runs.get(runId)`

Retrieves the full details of a run, including step outputs and the execution log.

```ts
nf.runs.get(runId: string): Promise<RunDetails>
```

---

### `nf.runs.list(options)`

Returns a paginated list of runs.

```ts
nf.runs.list(options?: {
  workflowId?: string;
  status?: RunStatus;
  limit?: number;      // Default: 20. Max: 100.
  cursor?: string;
}): Promise<{ runs: RunSummary[]; nextCursor: string | null }>
```

---

### `nf.runs.cancel(runId)`

Cancels a run that is currently `queued` or `running`.

```ts
nf.runs.cancel(runId: string): Promise<void>
```

---

### `nf.runs.replay(runId, options?)`

Creates a new run using the same context as a previous run. Typically used to retry a failed run from the dead-letter queue.

```ts
nf.runs.replay(runId: string, options?: {
  contextOverrides?: Record<string, unknown>;
}): Promise<RunDetails>
```
