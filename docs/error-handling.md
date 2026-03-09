# Error Handling

NexaFlow gives you precise control over how your workflows respond to failures. This guide covers retry policies, step-level error hooks, dead-letter queues, and how to write workflows that fail gracefully.

---

## Default failure behavior

When a step fails — because an action throws an error, returns an unexpected status code, or exceeds its timeout — NexaFlow:

1. Marks the step as `failed`
2. Stops executing subsequent steps
3. Marks the entire run as `failed`
4. Emits a `run.failed` event you can subscribe to

Without any retry configuration, failures are immediate and final. For most production workflows, you'll want to add retry policies.

---

## Retry policies

Add a `retry` block to any step to configure automatic retries on failure.

```js
{
  id: 'call-payment-api',
  action: 'http.post',
  params: { url: 'https://payments.example.com/charge', body: '{{ trigger }}' },
  retry: {
    attempts: 3,
    backoff: 'exponential',
    initialDelayMs: 1000,
    maxDelayMs: 30000,
  },
}
```

### Retry options

| Option | Type | Default | Description |
|---|---|---|---|
| `attempts` | integer | 1 | Total number of attempts including the first try |
| `backoff` | `'fixed'` \| `'exponential'` \| `'jitter'` | `'fixed'` | Delay strategy between attempts |
| `initialDelayMs` | integer | `1000` | Delay before the second attempt, in milliseconds |
| `maxDelayMs` | integer | `60000` | Cap on delay for exponential/jitter strategies |
| `retryOn` | string[] | all errors | List of error codes that trigger a retry; all others fail immediately |

### Backoff strategies

**Fixed:** waits exactly `initialDelayMs` between every retry.

**Exponential:** doubles the delay after each failure — `1s → 2s → 4s → 8s`, capped at `maxDelayMs`. Suitable for rate-limited APIs.

**Jitter:** applies randomized exponential backoff to reduce thundering-herd effects. Recommended for high-volume workflows that call shared services.

---

## Step-level error hooks

Use `onError` to run a fallback step when another step fails, instead of stopping the workflow.

```js
{
  id: 'send-sms-alert',
  action: 'sms.send',
  params: { to: '{{ trigger.phone }}', message: 'Your order has shipped.' },
  onError: {
    action: 'slack.post',
    params: {
      channel: '#ops-alerts',
      message: 'SMS delivery failed for order {{ trigger.orderId }}. Manual follow-up needed.',
    },
  },
}
```

If `send-sms-alert` fails after all retries, NexaFlow runs the `onError` action. If `onError` succeeds, the step is marked `failed_handled` and the workflow continues to the next step. If `onError` also fails, the workflow fails and stops.

---

## Timeout configuration

Set `timeoutMs` on any step to limit how long it can run before NexaFlow cancels it and treats it as a failure.

```js
{
  id: 'generate-report',
  action: 'http.post',
  params: { url: 'https://reporting.internal/generate' },
  timeoutMs: 30000,  // 30 seconds
  retry: { attempts: 2, backoff: 'fixed', initialDelayMs: 5000 },
}
```

If a step exceeds its timeout, NexaFlow records a `timeout` error and follows the retry/onError rules as normal.

---

## Dead-letter queues

When a run fails after exhausting all retries, NexaFlow can push the failed run to a dead-letter queue (DLQ) for inspection and manual replay.

Enable DLQ on a workflow:

```js
const workflow = nf.defineWorkflow('payment-processor', {
  trigger: { type: 'event', name: 'payment.submitted' },
  deadLetterQueue: {
    enabled: true,
    retentionDays: 14,
  },
  steps: [ ... ],
});
```

Failed runs in the DLQ are available in the NexaFlow dashboard under **Workflows > [name] > Dead-Letter Queue**. You can inspect the failure reason, edit the run context if needed, and replay the run.

### Replaying failed runs programmatically

```js
await nf.runs.replay('run_failedrunid123', {
  // Optionally override the run context for the replay
  contextOverrides: { retryReason: 'manual-replay' },
});
```

---

## Listening for run failures

Subscribe to the `run.failed` event to trigger alerting or escalation logic:

```js
nf.events.on('run.failed', async (event) => {
  await alerting.page({
    title: `Workflow failed: ${event.workflowId}`,
    body: `Run ${event.runId} failed at step "${event.failedStepId}": ${event.error.message}`,
    severity: 'high',
  });
});
```

---

## Testing failure scenarios

Use `nf.testing.simulateFailure()` in your test suite to verify that your retry and error handling logic works as expected without calling real external services:

```js
import { NexaFlow, testing } from '@nexaflow/sdk';

const nf = new NexaFlow({ apiKey: 'nxf_test_...' });

testing.simulateFailure('call-payment-api', {
  errorCode: 'rate_limit_exceeded',
  onAttempt: 1,  // Fail on the first attempt only; succeed after retry
});

const result = await workflow.run({ amount: 100 });
expect(result.status).toBe('completed');
expect(result.stepOutputs['call-payment-api'].retryCount).toBe(1);
```
