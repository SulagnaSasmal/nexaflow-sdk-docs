# Quickstart

This guide walks you through building and running your first NexaFlow workflow. By the end, you'll have a working workflow that fires on a simulated event, runs two sequential steps, and returns a result.

**Time to complete:** approximately 10 minutes.

**What you need:**
- NexaFlow SDK installed — see [Installation](installation.md)
- `NEXAFLOW_API_KEY` set in your environment

---

## Step 1: Initialize the client

Create a new file called `my-first-workflow.js` (or `my_first_workflow.py` for Python) and initialize the NexaFlow client.

**Node.js:**
```js
import { NexaFlow } from '@nexaflow/sdk';

const nf = new NexaFlow({ apiKey: process.env.NEXAFLOW_API_KEY });
```

**Python:**
```python
import os
from nexaflow import NexaFlow

nf = NexaFlow(api_key=os.environ["NEXAFLOW_API_KEY"])
```

---

## Step 2: Define a workflow

A workflow definition describes the trigger, the steps, and how data flows between them.

This example workflow does two things when a `user.signup` event fires:
1. Looks up additional user data from a mock HTTP endpoint
2. Sends a welcome email using the built-in email action

**Node.js:**
```js
const workflow = nf.defineWorkflow('welcome-flow', {
  trigger: {
    type: 'event',
    name: 'user.signup',
  },
  steps: [
    {
      id: 'fetch-profile',
      action: 'http.get',
      params: {
        url: 'https://api.example.com/users/{{ trigger.userId }}',
        headers: { Authorization: 'Bearer {{ env.EXAMPLE_API_KEY }}' },
      },
    },
    {
      id: 'send-welcome',
      action: 'email.send',
      params: {
        to: '{{ trigger.email }}',
        subject: 'Welcome to the platform',
        body: 'Hi {{ steps.fetch-profile.body.firstName }}, your account is ready.',
      },
    },
  ],
});
```

**Python:**
```python
workflow = nf.define_workflow('welcome-flow', {
    'trigger': {
        'type': 'event',
        'name': 'user.signup',
    },
    'steps': [
        {
            'id': 'fetch-profile',
            'action': 'http.get',
            'params': {
                'url': 'https://api.example.com/users/{{ trigger.userId }}',
            },
        },
        {
            'id': 'send-welcome',
            'action': 'email.send',
            'params': {
                'to': '{{ trigger.email }}',
                'subject': 'Welcome to the platform',
                'body': 'Hi {{ steps.fetch-profile.body.firstName }}, your account is ready.',
            },
        },
    ],
})
```

---

## Step 3: Register the workflow

Before a workflow can run, NexaFlow needs to register its definition in the control plane.

**Node.js:**
```js
await workflow.register();
console.log('Workflow registered:', workflow.id);
```

**Python:**
```python
workflow.register()
print(f"Workflow registered: {workflow.id}")
```

You only need to call `register()` once per version. If you update the definition, call `register()` again — NexaFlow creates a new version automatically.

---

## Step 4: Trigger a test run

Use `run()` to start a workflow manually with test input data. This is useful during development and testing.

**Node.js:**
```js
const result = await workflow.run({
  userId: 'usr_test_001',
  email: 'test@example.com',
});

console.log('Run status:', result.status);         // 'completed'
console.log('Run ID:', result.runId);              // 'run_abc123'
console.log('Step outputs:', result.stepOutputs); // { 'fetch-profile': {...}, 'send-welcome': {...} }
```

**Python:**
```python
result = workflow.run({
    'userId': 'usr_test_001',
    'email': 'test@example.com',
})

print(f"Run status: {result['status']}")         # completed
print(f"Run ID: {result['run_id']}")             # run_abc123
```

---

## Step 5: Check the execution log

You can retrieve the full execution history for any run by its ID.

**Node.js:**
```js
const log = await nf.runs.get(result.runId);
log.steps.forEach(step => {
  console.log(`${step.id}: ${step.status} (${step.durationMs}ms)`);
});
```

Expected output:
```
fetch-profile: completed (312ms)
send-welcome: completed (187ms)
```

---

## What's next

Now that your first workflow is running:

- Learn about [triggers](core-concepts.md#triggers) — schedule workflows with cron, or fire them from webhooks
- Read the [Error Handling](error-handling.md) guide to add retry policies and failure hooks
- Browse the full [API Reference](api-reference/README.md) for all available actions and options
