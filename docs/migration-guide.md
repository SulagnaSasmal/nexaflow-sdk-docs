# Migration Guide: v1 to v2

NexaFlow v2 is a major version with breaking changes to the SDK API. This guide lists every breaking change and tells you exactly how to update your code.

**Estimated migration time:** 30–90 minutes depending on workflow complexity.

---

## What changed and why

Version 2 standardizes the SDK API across Node.js and Python, removes several deprecated patterns from v1, and aligns the authentication model with the rest of the NexaFlow platform. The core concepts — workflows, triggers, steps, actions — are unchanged.

---

## Breaking changes

### 1. Constructor signature

**v1:**
```js
const nf = new NexaFlow('nxf_live_your_key_here');
```

**v2:**
```js
const nf = new NexaFlow({ apiKey: 'nxf_live_your_key_here' });
```

The constructor now accepts an options object, not a bare string. Update all client instantiation calls.

---

### 2. `workflow.execute()` renamed to `workflow.run()`

**v1:**
```js
const result = await workflow.execute({ userId: '123' });
```

**v2:**
```js
const result = await workflow.run({ userId: '123' });
```

The method signature is otherwise identical. This rename aligns the Node.js and Python SDKs.

---

### 3. Step result access

In v1, step outputs were accessed directly by step ID:

**v1:**
```js
result['send-email']  // A step output
```

**v2:**
```js
result.stepOutputs['send-email']
```

Step outputs are now namespaced under `result.stepOutputs` to avoid collisions with other result fields.

---

### 4. Event emission — `nf.emit()` moved to `nf.events.emit()`

**v1:**
```js
await nf.emit('user.signup', { userId: '123' });
```

**v2:**
```js
await nf.events.emit('user.signup', { userId: '123' });
```

---

### 5. Callback-style error handlers removed

v1 allowed an `onError` callback function on step definitions. v2 replaces this with a declarative `onError` block.

**v1:**
```js
{
  id: 'charge-card',
  action: 'http.post',
  params: { ... },
  onError: async (err, context) => {
    await notify(err);
  },
}
```

**v2:**
```js
{
  id: 'charge-card',
  action: 'http.post',
  params: { ... },
  onError: {
    action: 'slack.post',
    params: { channel: '#billing-alerts', message: 'Charge failed: {{ error.message }}' },
  },
}
```

If your v1 `onError` callback contained complex logic, refactor it into a custom action and reference it in the v2 `onError` block.

---

### 6. `workflow.deploy()` replaced by `workflow.register()`

**v1:**
```js
await workflow.deploy();
```

**v2:**
```js
await workflow.register();
```

The behavior is identical — the rename clarifies that this operation registers the definition, not deploys a running process.

---

### 7. API key scopes are now required for write operations

v1 API keys had implicit read+write access. v2 keys require explicit scopes.

If you created your API key before upgrading the SDK, generate a new key in **Settings > API Keys** with the appropriate scopes (`runs:write`, `workflows:write`, etc.). v1-style keys without a scope declaration will behave as read-only in the v2 control plane.

---

## Non-breaking changes

These additions are new in v2 and do not require changes to existing code:

- **Dead-letter queues** — configure via `deadLetterQueue` in the workflow definition
- **`nf.runs.replay()`** — replay failed runs programmatically
- **Parallel blocks** — run steps concurrently with the `parallel` array syntax
- **Webhook trigger HMAC verification** — now enforced when a `secret` is provided
- **Python SDK** — fully supported in v2; v1 Python support was experimental

---

## Step-by-step migration checklist

1. Update the SDK: `npm install @nexaflow/sdk@^2` or `pip install 'nexaflow>=2.0.0'`
2. Update all constructor calls from `new NexaFlow(key)` to `new NexaFlow({ apiKey: key })`
3. Rename all `.execute()` calls to `.run()`
4. Update result access from `result[stepId]` to `result.stepOutputs[stepId]`
5. Move `nf.emit()` calls to `nf.events.emit()`
6. Convert `onError` callback functions to declarative `onError` action blocks
7. Rename `.deploy()` calls to `.register()`
8. Generate new API keys with explicit scopes if you're using keys created before the upgrade
9. Run your test suite against the NexaFlow test environment before deploying to production
