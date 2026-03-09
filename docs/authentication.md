# Authentication

The NexaFlow SDK authenticates all requests to the control plane using an API key. This guide explains how to manage keys, apply the principle of least privilege using key scopes, and handle credentials safely in production.

---

## API keys

You create and revoke API keys in the NexaFlow dashboard under **Settings > API Keys**. Keys are environment-scoped: a key created in your test environment only works against test-environment endpoints, and vice versa.

Keys follow this format: `nxf_live_<32-char-alphanum>` (production) or `nxf_test_<32-char-alphanum>` (test).

---

## Setting the API key

### Environment variable (recommended)

Pass the key through the `NEXAFLOW_API_KEY` environment variable. The SDK reads it automatically.

```bash
export NEXAFLOW_API_KEY=nxf_live_abc123...
```

```js
// No key passed explicitly — SDK reads NEXAFLOW_API_KEY
const nf = new NexaFlow();
```

### Constructor parameter

You can pass the key directly if your application manages secrets through a vault or secret manager:

```js
const nf = new NexaFlow({ apiKey: secretManager.get('nexaflow-key') });
```

**Do not hard-code API keys in source files or commit them to version control.** Rotate any key that was accidentally exposed immediately. See [Key rotation](#key-rotation).

---

## Key scopes

When creating a key, assign only the scopes your application needs.

| Scope | What it allows |
|---|---|
| `workflows:read` | List and inspect workflow definitions |
| `workflows:write` | Create, update, and delete workflow definitions |
| `runs:read` | Read execution history and run status |
| `runs:write` | Trigger runs, cancel runs, and replay failed runs |
| `admin` | Manage team members, billing, and all other settings |

A backend service that only triggers workflows and reads their status needs `runs:write` and `runs:read`. It does not need `workflows:write` or `admin`.

---

## Key rotation

Rotate your API key whenever:

- A key is exposed in logs, source control, or error messages
- A team member with key access leaves your organization
- You establish a regular rotation policy (recommended: every 90 days)

**To rotate a key:**

1. Create a new key in **Settings > API Keys**.
2. Update your application or secret manager to use the new key.
3. Deploy and verify the new key is working.
4. Revoke the old key.

Steps 2–4 should happen in sequence with minimal gap to avoid service interruption.

---

## Proxy configuration

If your environment routes outbound traffic through an HTTP proxy, set the standard `HTTPS_PROXY` environment variable:

```bash
export HTTPS_PROXY=http://proxy.internal:8080
```

The SDK respects the `HTTPS_PROXY` and `NO_PROXY` environment variables automatically in both Node.js and Python.

For mutual TLS (mTLS) environments or custom CA certificates, configure the SDK client with a custom HTTP agent:

```js
import https from 'https';
import { NexaFlow } from '@nexaflow/sdk';

const agent = new https.Agent({
  ca: fs.readFileSync('/etc/ssl/certs/internal-ca.pem'),
});

const nf = new NexaFlow({
  apiKey: process.env.NEXAFLOW_API_KEY,
  httpAgent: agent,
});
```

---

## Service accounts

For production deployments, create a dedicated service account key rather than using a personal API key. Service account keys are not tied to an individual user's account and are not revoked if that user leaves your organization.

To create a service account:

1. In the NexaFlow dashboard, go to **Settings > Service Accounts**.
2. Create a new service account and assign the minimum required scopes.
3. Generate an API key for that service account.
4. Store the key in your secret manager or environment configuration.
