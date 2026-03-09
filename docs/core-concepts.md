# Core Concepts

Understanding NexaFlow requires a clear mental model of five things: workflows, triggers, steps, actions, and runs. This page explains each concept and how they connect.

---

## Workflows

A **workflow** is a named, versioned definition of a process. It describes what should happen, in what order, and under what conditions — but it doesn't run anything itself. Think of a workflow like a recipe: the recipe tells you the steps, but following the recipe is a separate act.

```js
const workflow = nf.defineWorkflow('invoice-processing', {
  trigger: { ... },
  steps: [ ... ],
});
```

Workflows are registered with the NexaFlow control plane. Once registered, they can be triggered at any time, from any instance of your application.

### Versions

Every time you call `register()` after updating a workflow's definition, NexaFlow stores a new version. Previous versions remain available and can still be triggered — this lets you run active workflow executions to completion without interruption while rolling out updates.

---

## Triggers

A **trigger** defines the condition that starts a new run of a workflow. NexaFlow supports three trigger types:

### Event trigger

Fires when your application emits a named event.

```js
trigger: {
  type: 'event',
  name: 'order.placed',
}
```

To fire an event trigger from your application:

```js
await nf.events.emit('order.placed', { orderId: 'ord_789', customerId: 'cust_456' });
```

The event payload becomes the run context's `trigger` object, accessible in step parameters as `{{ trigger.orderId }}`.

### Schedule trigger

Fires on a cron schedule.

```js
trigger: {
  type: 'schedule',
  cron: '0 9 * * 1',  // Every Monday at 9:00 AM UTC
}
```

Schedule triggers do not receive an external payload. Populate run context using a first step that fetches data.

### Webhook trigger

Fires when an HTTP POST request arrives at a NexaFlow-generated webhook URL.

```js
trigger: {
  type: 'webhook',
  secret: process.env.WEBHOOK_SECRET,  // Used to verify HMAC signature
}
```

NexaFlow verifies the request signature before starting a run. The webhook URL is available in the dashboard under **Workflows > [your workflow] > Trigger**.

---

## Steps

A **step** is a single unit of work within a workflow. Steps run in the order they are defined. Each step calls an action with a set of parameters and produces an output.

```js
{
  id: 'send-notification',
  action: 'slack.post',
  params: {
    channel: '#alerts',
    message: 'Order {{ trigger.orderId }} has shipped.',
  },
}
```

Step IDs must be unique within a workflow. The output of a step is available to all subsequent steps as `{{ steps.<step-id>.<outputField> }}`.

### Conditional steps

Add an `if` condition to a step to make it run only when the condition is true:

```js
{
  id: 'flag-high-value',
  action: 'db.update',
  if: '{{ trigger.amount > 10000 }}',
  params: {
    table: 'orders',
    id: '{{ trigger.orderId }}',
    set: { highValue: true },
  },
}
```

If the condition evaluates to false, the step is skipped and its output is `null`.

### Parallel steps

Wrap steps in a `parallel` block to run them concurrently:

```js
{
  parallel: [
    { id: 'send-email', action: 'email.send', params: { ... } },
    { id: 'send-sms', action: 'sms.send', params: { ... } },
  ],
}
```

All steps in a parallel block must complete before the next step in the sequence begins.

---

## Actions

An **action** is an integration that a step can call. NexaFlow ships with a library of built-in actions and supports custom actions that call your own code or external APIs.

### Built-in actions

| Action | What it does |
|---|---|
| `http.get` | HTTP GET request |
| `http.post` | HTTP POST request |
| `email.send` | Send an email via your configured email provider |
| `slack.post` | Post a message to a Slack channel |
| `db.query` | Run a read query against your connected database |
| `db.update` | Run a write query against your connected database |
| `data.transform` | Apply a JSONata expression to reshape data |
| `workflow.trigger` | Start another workflow as a sub-process |

### Custom actions

Define a custom action to wrap your own application logic:

```js
nf.defineAction('crm.createContact', async (params, context) => {
  const contact = await myCRM.contacts.create({
    email: params.email,
    name: params.name,
    source: 'nexaflow',
  });
  return { contactId: contact.id };
});
```

Register the action before running any workflows that use it.

---

## Runs

A **run** is a single execution of a workflow. Every time a trigger fires, NexaFlow creates a new run with:

- A unique run ID
- A copy of the trigger payload as the initial context
- An execution record in the audit log

### Run states

| State | Meaning |
|---|---|
| `queued` | Trigger received; run waiting to start |
| `running` | One or more steps are executing |
| `completed` | All steps finished successfully |
| `failed` | A step failed and retry attempts were exhausted |
| `cancelled` | Manually cancelled before completion |
| `paused` | Waiting for a human approval step to resolve |

### Accessing run context

Within step parameters, use the `{{ }}` template syntax to reference values from the run context:

- `{{ trigger.<field> }}` — data from the trigger payload
- `{{ steps.<stepId>.<field> }}` — output from a previous step
- `{{ env.<VAR> }}` — an environment variable set in the NexaFlow dashboard

Templates are evaluated at execution time, not at definition time.
