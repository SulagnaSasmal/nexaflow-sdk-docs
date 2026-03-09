# Triggers

This reference covers the three trigger types: event, schedule, and webhook.

---

## Event trigger

Fires when your application emits a named event using `nf.events.emit()`.

```ts
{
  type: 'event';
  name: string;
  filter?: string;  // Optional template expression to filter events
}
```

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `'event'` |
| `name` | Yes | The event name to listen for |
| `filter` | No | A template expression that must evaluate to `true` for the trigger to fire |

**Example — fire only for high-value orders:**

```js
trigger: {
  type: 'event',
  name: 'order.placed',
  filter: '{{ trigger.amount > 500 }}',
}
```

**Emitting an event:**

```js
await nf.events.emit('order.placed', {
  orderId: 'ord_789',
  amount: 1250,
  customerId: 'cust_456',
});
```

The payload passed to `emit()` becomes the `trigger` object in the run context.

---

## Schedule trigger

Fires on a cron schedule. All schedules run in UTC.

```ts
{
  type: 'schedule';
  cron: string;
  timezone?: string;  // IANA timezone name; default is 'UTC'
}
```

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `'schedule'` |
| `cron` | Yes | Standard five-field cron expression |
| `timezone` | No | Run the schedule in a specific timezone (e.g., `'America/New_York'`) |

**Common cron expressions:**

| Expression | Meaning |
|---|---|
| `0 * * * *` | Every hour at :00 |
| `0 9 * * 1-5` | Weekdays at 9:00 AM |
| `0 0 1 * *` | First day of every month at midnight |
| `*/15 * * * *` | Every 15 minutes |

**Example:**

```js
trigger: {
  type: 'schedule',
  cron: '0 6 * * *',         // Every day at 6:00 AM UTC
  timezone: 'Europe/London',
}
```

Schedule triggers do not receive a payload. The run context's `trigger` object contains the scheduled fire time:

```js
{
  trigger: {
    scheduledAt: '2026-03-09T06:00:00.000Z',
    workflowId: 'daily-digest',
  }
}
```

---

## Webhook trigger

Fires when NexaFlow receives an HTTP POST request at a unique webhook URL.

```ts
{
  type: 'webhook';
  secret?: string;        // HMAC-SHA256 signing secret for request verification
  parseBody?: 'json' | 'form' | 'raw';  // Default: 'json'
}
```

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `'webhook'` |
| `secret` | No | If set, NexaFlow verifies the `X-NexaFlow-Signature` header on every request |
| `parseBody` | No | How to parse the incoming request body |

**Getting the webhook URL:**

After registering a workflow with a webhook trigger, retrieve the URL from the NexaFlow dashboard (**Workflows > [name] > Trigger**) or via the API:

```js
const trigger = await nf.workflows.getTriggerUrl('invoice-intake');
console.log(trigger.webhookUrl);
// https://hooks.nexaflow.dev/wh_abc123...
```

**HMAC signature verification:**

When a `secret` is configured, NexaFlow computes an HMAC-SHA256 signature of the raw request body and delivers it in the `X-NexaFlow-Signature` header as `sha256=<hex-digest>`. NexaFlow rejects requests with missing or invalid signatures before starting a run.

Use a secret for any webhook that receives data from an external service. Signing secrets should be stored in your NexaFlow environment variables, not hard-coded.

**The run context for a webhook trigger:**

```js
{
  trigger: {
    method: 'POST',
    headers: { 'content-type': 'application/json', ... },
    body: { ... },        // Parsed request body
    rawBody: '...',       // Original body string (available when parseBody !== 'raw')
    sourceIp: '1.2.3.4',
  }
}
```
