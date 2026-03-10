# NexaFlow SDK Documentation

NexaFlow is a workflow automation SDK for Node.js and Python. It lets you build, run, and monitor multi-step automated workflows from your own application — without managing infrastructure.

This repository contains the full developer reference for the NexaFlow SDK, including installation, core concepts, an API reference, and runnable examples.

---

## What you can do with this SDK

- **Define workflows as code** — declare triggers, steps, and conditional branches in your preferred language
- **Run workflows synchronously or asynchronously** — fire-and-forget jobs or awaited pipelines
- **Handle retries and failures** — built-in retry policies, dead-letter queues, and error hooks
- **Observe execution in real time** — structured execution logs and a status polling API

---

## Documentation

| Section | Description |
|---|---|
| [Overview](docs/overview.md) | Architecture, use cases, and key concepts |
| [Installation](docs/installation.md) | Prerequisites and setup |
| [Quickstart](docs/quickstart.md) | Build and run your first workflow in 10 minutes |
| [Authentication](docs/authentication.md) | API keys, scopes, and secure credential storage |
| [Core Concepts](docs/core-concepts.md) | Workflows, triggers, steps, and actions explained |
| [Error Handling](docs/error-handling.md) | Retry policies, failure hooks, and dead-letter queues |
| [API Reference](docs/api-reference/) | Full method and class reference |
| [Migration Guide](docs/migration-guide.md) | Upgrading from v1 to v2 |
| [Changelog](docs/changelog.md) | Version history |

---

## Supported languages

| Language | Package | Minimum version |
|---|---|---|
| Node.js | `@nexaflow/sdk` | Node.js 18 LTS |
| Python | `nexaflow` | Python 3.10 |

---

## Quick example (Node.js)

```js
import { NexaFlow } from '@nexaflow/sdk';

const nf = new NexaFlow({ apiKey: process.env.NEXAFLOW_API_KEY });

const workflow = nf.defineWorkflow('send-welcome-email', {
  trigger: { type: 'event', name: 'user.created' },
  steps: [
    {
      id: 'lookup-user',
      action: 'db.query',
      params: { query: 'SELECT * FROM users WHERE id = :userId' },
    },
    {
      id: 'send-email',
      action: 'email.send',
      params: {
        to: '{{ steps.lookup-user.email }}',
        template: 'welcome-v2',
      },
    },
  ],
});

await workflow.run({ userId: '12345' });
```

---

## Repository layout

```
nexaflow-sdk-docs/
├── docs/
│   ├── overview.md
│   ├── installation.md
│   ├── quickstart.md
│   ├── authentication.md
│   ├── core-concepts.md
│   ├── error-handling.md
│   ├── migration-guide.md
│   ├── changelog.md
│   └── api-reference/
│       ├── README.md
│       ├── client.md
│       ├── workflows.md
│       ├── triggers.md
│       └── actions.md
└── examples/
    ├── node/
    └── python/
```

---

## Contributing

See the contributing guide for branch conventions, commit style, and how to submit a documentation improvement.

---

## License

MIT

---

## Status

**Phase 6 — SDK & Library Documentation — Complete**

| Section | Status |
|---------|--------|
| Overview & architecture | Complete |
| Installation (Node.js + Python) | Complete |
| Quickstart | Complete |
| Authentication & API key scopes | Complete |
| Core concepts | Complete |
| Error handling (retry policies, DLQ) | Complete |
| API Reference — Client (`NexaFlow` class) | Complete |
| API Reference — Workflows (`defineWorkflow`, `WorkflowInstance`) | Complete |
| API Reference — Triggers (event, schedule, webhook) | Complete |
| API Reference — Actions (HTTP, email, Slack, DB, data, workflow) | Complete |
| Migration guide (v1 → v2) | Complete |
| Changelog (v1.0 → v2.3) | Complete |
| Node.js examples | Complete |
| Python examples | Complete |

## Future Enhancements

- HTML documentation portal (rendered from Markdown using SpecFlow or MkDocs)
- Dark / light mode support if converted to HTML
- Python SDK full type reference (auto-generated)
- Rate limits and quota documentation
- On-premises / self-hosted deployment guide
- `@nexaflow/sdk` TypeScript declaration file reference
