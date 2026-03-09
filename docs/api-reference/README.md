# API Reference

This reference covers all classes, methods, and options exposed by the NexaFlow SDK.

---

## Sections

| Reference | Contents |
|---|---|
| [Client](client.md) | `NexaFlow` constructor options, top-level methods (`ping`, `events`) |
| [Workflows](workflows.md) | `defineWorkflow`, `WorkflowInstance` methods (`register`, `run`, `pause`, `resume`, `versions`) |
| [Triggers](triggers.md) | Trigger type schemas and options for event, schedule, and webhook triggers |
| [Actions](actions.md) | Built-in action reference — parameters, return values, and error codes |

---

## Template syntax

Many workflow parameters support the `{{ }}` template syntax. Templates are evaluated at execution time using the values in the run context.

| Expression | Resolves to |
|---|---|
| `{{ trigger.<field> }}` | A field from the trigger payload |
| `{{ steps.<stepId>.<field> }}` | An output field from a completed step |
| `{{ env.<VAR> }}` | An environment variable configured in the NexaFlow dashboard |
| `{{ run.id }}` | The current run's unique ID |
| `{{ run.workflowId }}` | The workflow ID |

Templates support dot notation for nested fields: `{{ steps.fetch-user.body.address.city }}`.

For data transformation beyond simple field access, use the `data.transform` action with a JSONata expression.

---

## Error codes

All NexaFlow SDK errors include a `code` property for programmatic handling.

| Code | Meaning |
|---|---|
| `authentication_error` | Invalid or revoked API key |
| `permission_denied` | Key does not have the required scope |
| `workflow_not_found` | Workflow ID does not exist or belongs to another environment |
| `run_not_found` | Run ID does not exist |
| `invalid_definition` | Workflow definition failed validation |
| `template_error` | Template expression could not be evaluated at runtime |
| `step_timeout` | Step exceeded its `timeoutMs` limit |
| `action_error` | Built-in action returned an error |
| `rate_limit_exceeded` | Too many API requests; back off and retry |
| `internal_error` | Unexpected error in the NexaFlow control plane |

Catch errors by code:

```js
try {
  await workflow.run(context);
} catch (err) {
  if (err.code === 'rate_limit_exceeded') {
    await sleep(err.retryAfterMs);
    await workflow.run(context);
  } else {
    throw err;
  }
}
```

---

## Pagination

List endpoints return paginated results. Pass `limit` and `cursor` to control page size and position:

```js
const page1 = await nf.runs.list({ workflowId: 'welcome-flow', limit: 50 });
const page2 = await nf.runs.list({ workflowId: 'welcome-flow', limit: 50, cursor: page1.nextCursor });
```

`nextCursor` is `null` when you've reached the last page.
