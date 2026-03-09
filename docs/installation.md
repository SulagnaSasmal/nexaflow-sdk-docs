# Installation

This page covers requirements and setup for the NexaFlow SDK in Node.js and Python environments.

---

## Prerequisites

Before you install the SDK, you need:

- A NexaFlow account — [sign up at nexaflow.dev](https://nexaflow.dev)
- An API key from your NexaFlow dashboard (Settings > API Keys)
- One of the supported runtimes:

| Runtime | Minimum version | Recommended |
|---|---|---|
| Node.js | 18 LTS | 20 LTS |
| Python | 3.10 | 3.12 |

---

## Node.js

Install the SDK using npm or yarn:

```bash
npm install @nexaflow/sdk
```

```bash
yarn add @nexaflow/sdk
```

Verify the installation:

```bash
node -e "const { NexaFlow } = require('@nexaflow/sdk'); console.log('NexaFlow SDK ready');"
```

### TypeScript support

The SDK ships with TypeScript declarations. No additional packages are needed.

```ts
import { NexaFlow, WorkflowDefinition, RunResult } from '@nexaflow/sdk';
```

---

## Python

Install the SDK using pip:

```bash
pip install nexaflow
```

Or add it to your `requirements.txt`:

```
nexaflow>=2.0.0
```

Verify the installation:

```bash
python -c "import nexaflow; print('NexaFlow SDK ready')"
```

### Virtual environments

The SDK has no native dependencies, so it works in any standard Python virtual environment, including venv, Poetry, and Conda.

---

## Configure your API key

The SDK reads your API key from the `NEXAFLOW_API_KEY` environment variable by default. Set it before running your application:

**Linux / macOS:**
```bash
export NEXAFLOW_API_KEY=nxf_live_your_key_here
```

**Windows (PowerShell):**
```powershell
$env:NEXAFLOW_API_KEY = "nxf_live_your_key_here"
```

You can also pass the key directly to the client constructor if your environment requires it — see [Authentication](authentication.md) for details and security guidance.

---

## Verify connectivity

After setting your API key, confirm the SDK can reach the control plane:

**Node.js:**
```js
import { NexaFlow } from '@nexaflow/sdk';

const nf = new NexaFlow({ apiKey: process.env.NEXAFLOW_API_KEY });
const status = await nf.ping();
console.log(status); // { ok: true, region: 'us-east-1', latencyMs: 42 }
```

**Python:**
```python
from nexaflow import NexaFlow

nf = NexaFlow(api_key=os.environ["NEXAFLOW_API_KEY"])
status = nf.ping()
print(status)  # {'ok': True, 'region': 'us-east-1', 'latency_ms': 38}
```

If the call fails, see the [Troubleshooting](#troubleshooting) section below.

---

## Troubleshooting

**`NEXAFLOW_API_KEY is not set` error**  
The SDK requires the environment variable to be set before the client is instantiated. Check that the variable is exported in the same shell session where your application starts.

**`AuthenticationError: invalid key`**  
API keys are environment-scoped: a key created in your test environment will not work against production endpoints and vice versa. Confirm the key matches the environment you're targeting.

**`ConnectionError: timeout`**  
Check that your network or firewall allows outbound HTTPS (port 443) to `api.nexaflow.dev`. If you're in a restricted network, contact your network administrator or see the [proxy configuration](authentication.md#proxy-configuration) section of the Authentication guide.

**SDK version mismatch warning**  
If you see a warning that your SDK version is behind the control plane API version, run `npm update @nexaflow/sdk` or `pip install --upgrade nexaflow`. Using an outdated SDK is supported but may mean newer features aren't available.
