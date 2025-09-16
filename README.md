# 🧭 MCP-Duty-Server

A structured API layer that enables **LLMs** to interact with the **PagerDuty API** seamlessly. Designed for programmatic use with standardized inputs and outputs, this server helps large language models manage incidents, teams, services, and more.

## 🔍 Overview

**MCP-Duty-Server** provides tool interfaces for LLMs to access PagerDuty's APIs and manage:

- Incidents
- Services
- Teams
- Users
- On-call schedules
- Escalation policies

It supports structured interactions with validation, automatic pagination, and standardized response formatting — optimized for use in LLM workflows.

---

## ⚙️ Setup

### 1. Environment Preparation

```bash
cd pagerduty-mcp-server
brew install uv
uv sync
```

### 2. Configure PagerDuty API Token

You can provide your API token using:

#### ✅ Option A: Environment Variable

```bash
export PAGERDUTY_API_TOKEN=your_api_token_here
```

#### ✅ Option B: `.env` File (Recommended)

```bash
echo "PAGERDUTY_API_TOKEN=your_api_token_here" > .env
```

The server automatically loads environment variables from `.env` if present.

---

## 🧠 Usage Modes

### 🖥️ As a Goose Extension (Desktop)

1. Open Goose > **Settings > Advanced Settings > Extensions > Add Custom Extension**
2. Set:

   - **Name**: `pagerduty-mcp-server`
   - **Type**: `StandardIO`
   - **Command**:

     ```bash
     uvx pagerduty-mcp-server
     ```

3. Save and enable the extension.

### 💻 As a Goose Extension (YAML / CLI)

```yaml
pagerduty:
  cmd: uvx
  args:
    - pagerduty-mcp-server
  type: stdio
  enabled: true
  env_keys:
    - PAGERDUTY_API_TOKEN
  timeout: 300
```

### 🤖 In Claude / Cursor

```json
{
  "mcpServers": {
    "pagerduty-mcp-server": {
      "command": "uvx",
      "args": ["pagerduty-mcp-server"],
      "env": {
        "PAGERDUTY_API_TOKEN": "<your_api_token>"
      }
    }
  }
}
```

### 🌐 Standalone Server

```bash
uv run path/to/repo/pagerduty-mcp-server/.venv/bin/pagerduty-mcp-server
```

---

## 📦 Response Structure

All responses follow a consistent schema:

```json
{
  "metadata": {
    "count": <int>,
    "description": "<summary>"
  },
  "<resource_type_plural>": [ ... ],
  "error": {
    "message": "<error message>",
    "code": "<error code>"
  }
}
```

### 🛑 Error Example

```json
{
  "metadata": {
    "count": 0,
    "description": "Error occurred while processing request"
  },
  "error": {
    "message": "Invalid user ID provided",
    "code": "INVALID_USER_ID"
  }
}
```

---

## 🧪 Validation Rules

- All IDs must match valid PagerDuty resource formats
- Date values must be ISO8601 timestamps
- Lists (e.g. `statuses`, `team_ids`) must have valid entries
- Invalid values in lists are silently ignored
- Empty strings / nulls are not accepted for required parameters

### Incident Status Filter

```json
"statuses": ["triggered", "acknowledged", "resolved"]
```

### Incident Urgency

```json
"urgency": ["high", "low"]
```

---

## 📉 Pagination & Limits

- Automatic pagination of long responses
- Server enforces PagerDuty rate limits
- Use `limit` to cap results (default is `{pagerduty-mcp-server.utils.RESPONSE_LIMIT}`)

---

## 👤 User Context Behavior

Many endpoints support `current_user_context: true` to filter results to the authenticated user.

When enabled:

- `user_ids` is not allowed (for all resources)
- `team_ids`/`service_ids` ignored (for incidents)
- Filters that conflict with user context are excluded
- `schedule_ids` can still be used (on-call queries)

> Useful for questions like: _"Who’s on call for my team next week?"_

---

## 🧪 Testing & Coverage

### Run All Tests

```bash
uv run pytest --cov=src --cov-report=term-missing
```

### Run Unit Tests Only

```bash
uv run pytest -m unit --cov=src --cov-report=term-missing
```

### Run Integration Tests (Requires API Token)

```bash
uv run pytest -m integration --cov=src --cov-report=term-missing
```

### Run Parser-Specific Tests

```bash
uv run pytest -m parsers --cov=src --cov-report=term-missing
```

### Filter by Submodule

```bash
uv run pytest -m escalation_policies --cov=src --cov-report=term-missing
```

---

## 🧪 Debug with MCP Inspector

```bash
npx @modelcontextprotocol/inspector uv run path/to/repo/pagerduty-mcp-server/.venv/bin/pagerduty-mcp-server
```

---

## 📚 Documentation

- Standardized response formatting
- All timestamps in ISO8601
- All lists pluralized (even for single results)
- Error responses always include `message` and `code`
- Tests tagged using `pytest` markers by function (`unit`, `integration`, `parsers`, etc.)

---

## 🔍 Example Queries

- "What incidents are currently assigned to me?"
- "Am I on call in the next 2 weeks?"
- "Who else is on the personalization team?"
