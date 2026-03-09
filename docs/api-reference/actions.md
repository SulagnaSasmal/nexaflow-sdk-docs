# Actions

NexaFlow ships with a set of built-in actions you can use in your workflow steps without writing any integration code.

---

## HTTP actions

### `http.get`

Makes a GET request to a URL.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `url` | string | Yes | Request URL; supports templates |
| `headers` | object | No | HTTP headers as key-value pairs |
| `queryParams` | object | No | Query string parameters |
| `timeoutMs` | integer | No | Request timeout; overrides step-level timeout |
| `followRedirects` | boolean | No | Default: `true` |

**Returns:** `{ statusCode, headers, body }` where `body` is the parsed JSON response (or a string for non-JSON responses).

---

### `http.post`

Makes a POST request with a JSON or form body.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `url` | string | Yes | Request URL |
| `body` | object \| string | No | Request body; serialized as JSON by default |
| `headers` | object | No | HTTP headers |
| `contentType` | `'json'` \| `'form'` | No | Default: `'json'` |
| `timeoutMs` | integer | No | Request timeout |

**Returns:** Same as `http.get`.

**Error codes:**

| Code | Meaning |
|---|---|
| `http_4xx` | Server returned a 4xx status code |
| `http_5xx` | Server returned a 5xx status code |
| `http_timeout` | Request exceeded the timeout |
| `http_connection_error` | Could not connect to the host |

---

## Email actions

### `email.send`

Sends an email using your configured email provider (configured in **Settings > Integrations**).

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `to` | string \| string[] | Yes | Recipient email address(es) |
| `subject` | string | Yes | Email subject line |
| `body` | string | No | Plain-text body |
| `html` | string | No | HTML body; used instead of `body` if both are provided |
| `template` | string | No | Template ID from your email provider; overrides `body` and `html` |
| `templateData` | object | No | Variables to inject into the template |
| `from` | string | No | Sender address; defaults to your configured default sender |
| `cc` | string \| string[] | No | CC addresses |
| `bcc` | string \| string[] | No | BCC addresses |

**Returns:** `{ messageId, provider, acceptedAt }`

---

## Messaging actions

### `slack.post`

Posts a message to a Slack channel using your configured Slack integration.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `channel` | string | Yes | Channel name (e.g., `#alerts`) or channel ID |
| `message` | string | Yes | Message text; supports Slack mrkdwn format |
| `blocks` | object[] | No | Slack Block Kit blocks; used instead of `message` if provided |
| `unfurlLinks` | boolean | No | Default: `false` |

**Returns:** `{ ts, channel }` — the Slack message timestamp and channel.

---

## Database actions

### `db.query`

Runs a read-only SQL query against your connected database.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Parameterized SQL query |
| `params` | object | No | Named parameters to bind (`:paramName` syntax) |
| `connection` | string | No | Connection name if multiple databases are configured |

**Returns:** `{ rows, rowCount }`

**Security note:** Always use parameterized queries with `:paramName` syntax. Never interpolate user-provided values directly into the query string.

---

### `db.update`

Runs a write query (INSERT, UPDATE, DELETE) against your connected database.

**Parameters:** Same as `db.query`.

**Returns:** `{ affectedRows, lastInsertId }`

---

## Data actions

### `data.transform`

Applies a [JSONata](https://jsonata.org/) expression to transform data within the run context.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input` | any | Yes | The data to transform; supports templates |
| `expression` | string | Yes | A JSONata expression |

**Returns:** The result of evaluating `expression` against `input`.

**Example — extract email addresses from an array of users:**

```js
{
  id: 'extract-emails',
  action: 'data.transform',
  params: {
    input: '{{ steps.fetch-users.rows }}',
    expression: '$[email != ""].(email)',
  },
}
```

---

## Workflow actions

### `workflow.trigger`

Starts another registered workflow as a sub-process.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workflowId` | string | Yes | ID of the workflow to start |
| `context` | object | No | Trigger payload for the child workflow |
| `waitForCompletion` | boolean | No | Default: `false`. Set to `true` to block until the child run finishes. |

**Returns:** `{ runId, status }`. If `waitForCompletion` is `false`, `status` is `'queued'`.
