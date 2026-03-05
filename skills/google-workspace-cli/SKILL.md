---
name: google-workspace-cli
description: Interact with all Google Workspace APIs via the `gws` CLI — Drive, Gmail, Calendar, Sheets, Docs, Slides, Chat, Tasks, Admin, Meet, Forms, Keep, and more. Use when the user needs to manage Google Drive files, send or read Gmail, create or query Calendar events, read or write Sheets/Docs/Slides, manage Chat spaces, administer Workspace users/groups, or automate any Google Workspace workflow. Also use when configuring the gws MCP server for AI agent integrations. Triggers on Google Workspace, gws, Google Drive, Gmail, Google Calendar, Google Sheets, Google Docs, Google Slides, Google Chat, Google Tasks, Google Meet, Google Forms, Google Keep, Google Admin, Workspace CLI, gws CLI, gws mcp, Google API, Workspace automation.
---

# Google Workspace CLI (`gws`)

One CLI for **all** of Google Workspace — Drive, Gmail, Calendar, Sheets, Docs, Slides, Chat, Tasks, Admin, Meet, Forms, Keep, and every other Workspace API. Built for humans and AI agents. Structured JSON output. 100+ agent skills included.

> **Note:** This is not an officially supported Google product.

**Repository:** https://github.com/googleworkspace/cli

## How It Works

`gws` does NOT ship a static list of commands. It reads Google's own [Discovery Service](https://developers.google.com/discovery) at runtime and builds its entire command surface dynamically. When Google adds a new API endpoint or method, `gws` picks it up automatically — zero updates needed.

## Prerequisites

- **Node.js 18+** — for `npm install` (or download a pre-built binary from [GitHub Releases](https://github.com/googleworkspace/cli/releases))
- **A Google Cloud project** — required for OAuth credentials
- **A Google account** with access to Google Workspace

## Installation

```bash
# Install globally via npm (recommended — bundles native binaries, no Rust needed)
npm install -g @googleworkspace/cli

# Verify installation
gws --version
```

Alternative installation methods:
```bash
# From GitHub Releases (pre-built binaries)
# Download from: https://github.com/googleworkspace/cli/releases

# Build from source (requires Rust toolchain)
cargo install --git https://github.com/googleworkspace/cli --locked

# Nix flake
nix run github:googleworkspace/cli
```

## Authentication

### Quick Setup (recommended — requires gcloud CLI)

```bash
gws auth setup       # one-time: creates a Cloud project, enables APIs, logs you in
gws auth login       # subsequent logins with scope selection
```

### Scoped Login (for unverified/testing OAuth apps, limited to ~25 scopes)

```bash
# Select only the services you need to stay under the scope limit
gws auth login -s drive,gmail,sheets
gws auth login -s drive,gmail,calendar,docs,chat
```

### Multiple Accounts

```bash
gws auth login --account work@corp.com
gws auth login --account personal@gmail.com

gws auth list                                    # list registered accounts
gws auth default work@corp.com                   # set the default

gws --account personal@gmail.com drive files list  # one-off override
export GOOGLE_WORKSPACE_CLI_ACCOUNT=personal@gmail.com  # env var override
```

### Manual OAuth Setup (no gcloud)

1. Open [Google Cloud Console](https://console.cloud.google.com/) → create or select a project
2. Go to **OAuth consent screen** → configure as External (testing mode is fine)
3. Add your account under **Test users**
4. Go to **Credentials** → Create OAuth client → Type: **Desktop app**
5. Download the client JSON → save to `~/.config/gws/client_secret.json`
6. Run `gws auth login`

### Headless / CI

```bash
# On a machine with a browser:
gws auth export --unmasked > credentials.json

# On the headless machine:
export GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE=/path/to/credentials.json
gws drive files list   # just works
```

### Service Account (server-to-server)

```bash
export GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE=/path/to/service-account.json
gws drive files list

# For Domain-Wide Delegation:
export GOOGLE_WORKSPACE_CLI_IMPERSONATED_USER=admin@example.com
```

### Pre-obtained Access Token

```bash
export GOOGLE_WORKSPACE_CLI_TOKEN=$(gcloud auth print-access-token)
```

### Auth Precedence

| Priority | Method | Source |
|:---------|:-------|:-------|
| 1 | Access token | `GOOGLE_WORKSPACE_CLI_TOKEN` |
| 2 | Credentials file | `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` |
| 3 | Per-account encrypted credentials | `gws auth login --account EMAIL` |
| 4 | Plaintext credentials | `~/.config/gws/credentials.json` |

Account resolution: `--account` flag > `GOOGLE_WORKSPACE_CLI_ACCOUNT` env var > default in `accounts.json`.

## Command Structure

The universal pattern for ALL gws commands:

```
gws <service> <resource> <method> [--params '{ JSON }'] [--json '{ JSON }'] [flags]
```

### Global Flags

| Flag | Description |
|:-----|:------------|
| `--params '{ JSON }'` | URL/query parameters as JSON |
| `--json '{ JSON }'` | Request body as JSON |
| `--dry-run` | Preview the HTTP request without executing |
| `--page-all` | Auto-paginate, one JSON line per page (NDJSON) |
| `--page-limit <N>` | Max pages to fetch (default: 10) |
| `--page-delay <MS>` | Delay between pages (default: 100ms) |
| `--upload <path>` | Multipart file upload |
| `--account <email>` | Use a specific authenticated account |
| `--sanitize <template>` | Model Armor response sanitization |

### Introspecting Schemas

```bash
# See the full request/response schema for any method
gws schema drive.files.list
gws schema gmail.users.messages.send
gws schema calendar.events.insert
```

## Core Services — Commands & Examples

### Google Drive

```bash
# List files (paginated)
gws drive files list --params '{"pageSize": 10}'

# List ALL files (auto-paginate as NDJSON)
gws drive files list --params '{"pageSize": 100}' --page-all

# Search for files
gws drive files list --params '{"q": "name contains '\''report'\'' and mimeType = '\''application/pdf'\''", "pageSize": 20}'

# Get file metadata
gws drive files get --params '{"fileId": "FILE_ID"}'

# Upload a file
gws drive files create --json '{"name": "report.pdf", "parents": ["FOLDER_ID"]}' --upload ./report.pdf

# Create a folder
gws drive files create --json '{"name": "Project Docs", "mimeType": "application/vnd.google-apps.folder"}'

# Move a file to a folder
gws drive files update --params '{"fileId": "FILE_ID", "addParents": "FOLDER_ID", "removeParents": "OLD_PARENT_ID"}'

# Share a file
gws drive permissions create --params '{"fileId": "FILE_ID"}' --json '{"role": "writer", "type": "user", "emailAddress": "user@example.com"}'

# Download a file (export Google Docs as PDF)
gws drive files export --params '{"fileId": "FILE_ID", "mimeType": "application/pdf"}'

# Delete a file
gws drive files delete --params '{"fileId": "FILE_ID"}'

# List shared drives
gws drive drives list --params '{"pageSize": 10}'

# Create a shared drive
gws drive drives create --params '{"requestId": "unique-id"}' --json '{"name": "Team Drive"}'
```

### Gmail

```bash
# List messages in inbox
gws gmail users messages list --params '{"userId": "me", "maxResults": 10}'

# Search messages
gws gmail users messages list --params '{"userId": "me", "q": "from:boss@company.com is:unread", "maxResults": 20}'

# Get a specific message
gws gmail users messages get --params '{"userId": "me", "id": "MESSAGE_ID"}'

# Send an email
gws gmail users messages send --params '{"userId": "me"}' --json '{
  "raw": "<BASE64_ENCODED_RFC2822_MESSAGE>"
}'

# List labels
gws gmail users labels list --params '{"userId": "me"}'

# Create a label
gws gmail users labels create --params '{"userId": "me"}' --json '{"name": "Important/Projects"}'

# Modify message labels
gws gmail users messages modify --params '{"userId": "me", "id": "MESSAGE_ID"}' --json '{"addLabelIds": ["LABEL_ID"], "removeLabelIds": ["INBOX"]}'

# Trash a message
gws gmail users messages trash --params '{"userId": "me", "id": "MESSAGE_ID"}'

# List drafts
gws gmail users drafts list --params '{"userId": "me"}'

# Create a draft
gws gmail users drafts create --params '{"userId": "me"}' --json '{
  "message": {"raw": "<BASE64_ENCODED_RFC2822_MESSAGE>"}
}'

# Set vacation auto-reply
gws gmail users settings updateVacation --params '{"userId": "me"}' --json '{
  "enableAutoReply": true,
  "responseSubject": "Out of Office",
  "responseBodyPlainText": "I am out of office until March 10.",
  "restrictToContacts": false,
  "restrictToDomain": false
}'
```

### Google Calendar

```bash
# List upcoming events
gws calendar events list --params '{"calendarId": "primary", "timeMin": "2026-03-05T00:00:00Z", "maxResults": 10, "singleEvents": true, "orderBy": "startTime"}'

# Get a specific event
gws calendar events get --params '{"calendarId": "primary", "eventId": "EVENT_ID"}'

# Create an event
gws calendar events insert --params '{"calendarId": "primary"}' --json '{
  "summary": "Team Standup",
  "description": "Daily standup meeting",
  "start": {"dateTime": "2026-03-06T09:00:00-05:00", "timeZone": "America/New_York"},
  "end": {"dateTime": "2026-03-06T09:30:00-05:00", "timeZone": "America/New_York"},
  "attendees": [{"email": "alice@example.com"}, {"email": "bob@example.com"}]
}'

# Update an event
gws calendar events update --params '{"calendarId": "primary", "eventId": "EVENT_ID"}' --json '{
  "summary": "Updated Standup",
  "start": {"dateTime": "2026-03-06T10:00:00-05:00"},
  "end": {"dateTime": "2026-03-06T10:30:00-05:00"}
}'

# Delete an event
gws calendar events delete --params '{"calendarId": "primary", "eventId": "EVENT_ID"}'

# List calendars
gws calendar calendarList list

# Check free/busy
gws calendar freebusy query --json '{
  "timeMin": "2026-03-06T08:00:00Z",
  "timeMax": "2026-03-06T18:00:00Z",
  "items": [{"id": "alice@example.com"}, {"id": "bob@example.com"}]
}'

# Create a recurring event
gws calendar events insert --params '{"calendarId": "primary"}' --json '{
  "summary": "Weekly Review",
  "recurrence": ["RRULE:FREQ=WEEKLY;BYDAY=FR"],
  "start": {"dateTime": "2026-03-06T16:00:00-05:00", "timeZone": "America/New_York"},
  "end": {"dateTime": "2026-03-06T17:00:00-05:00", "timeZone": "America/New_York"}
}'
```

### Google Sheets

**Important:** Sheets ranges use `!` which bash interprets as history expansion. Always wrap values in single quotes.

```bash
# Create a spreadsheet
gws sheets spreadsheets create --json '{"properties": {"title": "Q1 Budget"}}'

# Read cells
gws sheets spreadsheets values get --params '{"spreadsheetId": "SPREADSHEET_ID", "range": "Sheet1!A1:C10"}'

# Write cells
gws sheets spreadsheets values update --params '{"spreadsheetId": "SPREADSHEET_ID", "range": "Sheet1!A1", "valueInputOption": "USER_ENTERED"}' --json '{"values": [["Name", "Score"], ["Alice", 95], ["Bob", 87]]}'

# Append rows
gws sheets spreadsheets values append --params '{"spreadsheetId": "SPREADSHEET_ID", "range": "Sheet1!A1", "valueInputOption": "USER_ENTERED"}' --json '{"values": [["Charlie", 92]]}'

# Get spreadsheet metadata
gws sheets spreadsheets get --params '{"spreadsheetId": "SPREADSHEET_ID"}'

# Clear a range
gws sheets spreadsheets values clear --params '{"spreadsheetId": "SPREADSHEET_ID", "range": "Sheet1!A1:C10"}'

# Batch update (add sheet, format cells, etc.)
gws sheets spreadsheets batchUpdate --params '{"spreadsheetId": "SPREADSHEET_ID"}' --json '{
  "requests": [
    {"addSheet": {"properties": {"title": "March Data"}}},
    {"repeatCell": {
      "range": {"sheetId": 0, "startRowIndex": 0, "endRowIndex": 1},
      "cell": {"userEnteredFormat": {"textFormat": {"bold": true}}},
      "fields": "userEnteredFormat.textFormat.bold"
    }}
  ]
}'
```

### Google Docs

```bash
# Create a document
gws docs documents create --json '{"title": "Meeting Notes"}'

# Get document content
gws docs documents get --params '{"documentId": "DOC_ID"}'

# Update document (insert text)
gws docs documents batchUpdate --params '{"documentId": "DOC_ID"}' --json '{
  "requests": [
    {"insertText": {"location": {"index": 1}, "text": "Hello, World!\n"}}
  ]
}'
```

### Google Slides

```bash
# Create a presentation
gws slides presentations create --json '{"title": "Q1 Review"}'

# Get presentation
gws slides presentations get --params '{"presentationId": "PRES_ID"}'

# Add a slide
gws slides presentations batchUpdate --params '{"presentationId": "PRES_ID"}' --json '{
  "requests": [
    {"createSlide": {"slideLayoutReference": {"predefinedLayout": "TITLE_AND_BODY"}}}
  ]
}'
```

### Google Chat

```bash
# List spaces
gws chat spaces list

# Send a message to a space
gws chat spaces messages create --params '{"parent": "spaces/SPACE_ID"}' --json '{"text": "Deploy complete."}'

# Get a message
gws chat spaces messages get --params '{"name": "spaces/SPACE_ID/messages/MSG_ID"}'

# List messages in a space
gws chat spaces messages list --params '{"parent": "spaces/SPACE_ID", "pageSize": 25}'

# Create a space
gws chat spaces create --json '{"displayName": "Project Alpha", "spaceType": "SPACE"}'
```

### Google Tasks

```bash
# List task lists
gws tasks tasklists list

# Create a task list
gws tasks tasklists insert --json '{"title": "Sprint 42"}'

# List tasks in a list
gws tasks tasks list --params '{"tasklist": "TASKLIST_ID"}'

# Create a task
gws tasks tasks insert --params '{"tasklist": "TASKLIST_ID"}' --json '{"title": "Review PR #123", "due": "2026-03-07T00:00:00Z"}'

# Complete a task
gws tasks tasks update --params '{"tasklist": "TASKLIST_ID", "task": "TASK_ID"}' --json '{"status": "completed"}'
```

### Google Meet

```bash
# Create a meeting space
gws meet spaces create --json '{}'

# Get meeting space info
gws meet spaces get --params '{"name": "spaces/SPACE_ID"}'
```

### Google Forms

```bash
# Create a form
gws forms forms create --json '{"info": {"title": "Feedback Survey"}}'

# Get form
gws forms forms get --params '{"formId": "FORM_ID"}'

# List responses
gws forms forms responses list --params '{"formId": "FORM_ID"}'
```

### Google Admin (Directory)

```bash
# List users
gws admin users list --params '{"domain": "example.com", "maxResults": 100}'

# Get a user
gws admin users get --params '{"userKey": "user@example.com"}'

# Create a user
gws admin users insert --json '{
  "primaryEmail": "newuser@example.com",
  "name": {"givenName": "Jane", "familyName": "Doe"},
  "password": "TempP@ssw0rd!"
}'

# List groups
gws admin groups list --params '{"domain": "example.com"}'

# Add member to group
gws admin members insert --params '{"groupKey": "group@example.com"}' --json '{"email": "user@example.com", "role": "MEMBER"}'
```

### Google Keep

```bash
# List notes
gws keep notes list

# Get a note
gws keep notes get --params '{"name": "notes/NOTE_ID"}'
```

### Apps Script

```bash
# List projects
gws apps-script projects list

# Deploy a project
gws apps-script projects deployments create --params '{"scriptId": "SCRIPT_ID"}' --json '{"versionNumber": 1}'
```

## Workflow Helpers (Shortcut Commands)

`gws` ships higher-level helper commands for the most common multi-step operations:

```bash
# Upload a file to Drive with automatic metadata
gws drive-upload ./report.pdf

# Append a row to a sheet
gws sheets-append --spreadsheet-id ID --range 'Sheet1!A1' --values '[["Name", "Score"]]'

# Read sheet values
gws sheets-read --spreadsheet-id ID --range 'Sheet1!A1:C10'

# Send an email (simplified)
gws gmail-send --to user@example.com --subject "Hello" --body "Hi there"

# Triage inbox — show unread summary
gws gmail-triage

# Show upcoming calendar agenda
gws calendar-agenda

# Insert a calendar event quickly
gws calendar-insert --summary "Lunch" --start "2026-03-06T12:00:00" --end "2026-03-06T13:00:00"

# Append text to a Google Doc
gws docs-write --document-id DOC_ID --text "New paragraph here"

# Send a Chat message
gws chat-send --space "spaces/SPACE_ID" --text "Hello team!"

# Today's standup report (meetings + open tasks)
gws workflow-standup-report

# Meeting prep (agenda, attendees, linked docs)
gws workflow-meeting-prep

# Convert email to task
gws workflow-email-to-task --message-id MSG_ID

# Weekly digest (meetings + unread count)
gws workflow-weekly-digest
```

## MCP Server Integration

`gws mcp` starts a [Model Context Protocol](https://modelcontextprotocol.io/) server over stdio, exposing Google Workspace APIs as structured tools for any MCP-compatible client.

```bash
# Start MCP server for specific services
gws mcp -s drive                  # Drive only
gws mcp -s drive,gmail,calendar   # multiple services
gws mcp -s all                    # all services (many tools!)

# Include workflow and helper tools
gws mcp -s drive,gmail -w -e
```

### MCP Client Configuration

**VS Code / Copilot / Claude Desktop / Cursor:**
```json
{
  "mcpServers": {
    "gws": {
      "command": "gws",
      "args": ["mcp", "-s", "drive,gmail,calendar,sheets,docs"]
    }
  }
}
```

**Gemini CLI Extension:**
```bash
gws auth setup
gemini extensions install https://github.com/googleworkspace/cli
```

> **Tip:** Each service adds roughly 10–80 tools. Keep the list to what you actually need to stay under your client's tool limit (typically 50–100 tools).

### MCP Flags

| Flag | Description |
|:-----|:------------|
| `-s, --services <list>` | Comma-separated services to expose, or `all` |
| `-w, --workflows` | Also expose workflow tools |
| `-e, --helpers` | Also expose helper tools |

## Advanced Usage

### Dry Run (preview requests without executing)

```bash
gws drive files list --params '{"pageSize": 5}' --dry-run
```

### Pagination

```bash
# Auto-paginate everything as NDJSON
gws drive files list --params '{"pageSize": 100}' --page-all

# Limit pages
gws drive files list --params '{"pageSize": 100}' --page-all --page-limit 5

# Delay between pages (rate limiting)
gws drive files list --params '{"pageSize": 100}' --page-all --page-delay 200
```

### Piping & Processing Output

All output is structured JSON. Pipe to `jq` for processing:

```bash
# Get just file names
gws drive files list --params '{"pageSize": 100}' --page-all | jq -r '.files[].name'

# Get unread email subjects
gws gmail users messages list --params '{"userId": "me", "q": "is:unread", "maxResults": 5}' | jq '.messages[].id'

# Count events this week
gws calendar events list --params '{"calendarId": "primary", "timeMin": "2026-03-02T00:00:00Z", "timeMax": "2026-03-08T00:00:00Z", "singleEvents": true}' | jq '.items | length'
```

### Multipart Uploads

```bash
gws drive files create --json '{"name": "report.pdf"}' --upload ./report.pdf
```

### Model Armor (Response Sanitization)

Scan API responses for prompt injection before they reach your agent:

```bash
gws gmail users messages get --params '{"userId": "me", "id": "MSG_ID"}' \
  --sanitize "projects/P/locations/L/templates/T"
```

| Environment Variable | Description |
|:---------------------|:------------|
| `GOOGLE_WORKSPACE_CLI_SANITIZE_TEMPLATE` | Default Model Armor template |
| `GOOGLE_WORKSPACE_CLI_SANITIZE_MODE` | `warn` (default) or `block` |

## Agent Decision Guide

Use this table to decide which `gws` command to run based on what the user is asking:

| User Intent | Service | Example Command |
|:------------|:--------|:----------------|
| List, search, upload, download, share files | `drive` | `gws drive files list` |
| Create folders, manage permissions | `drive` | `gws drive files create`, `gws drive permissions create` |
| Read, send, search, label emails | `gmail` | `gws gmail users messages list/send` |
| Create drafts, manage filters | `gmail` | `gws gmail users drafts create` |
| View, create, update, delete calendar events | `calendar` | `gws calendar events list/insert/update/delete` |
| Check availability / free-busy | `calendar` | `gws calendar freebusy query` |
| Read, write, append spreadsheet data | `sheets` | `gws sheets spreadsheets values get/update/append` |
| Create spreadsheets, format cells | `sheets` | `gws sheets spreadsheets create/batchUpdate` |
| Create, read, edit documents | `docs` | `gws docs documents create/get/batchUpdate` |
| Create, edit presentations | `slides` | `gws slides presentations create/batchUpdate` |
| Send messages, manage chat spaces | `chat` | `gws chat spaces messages create` |
| Manage tasks and to-do lists | `tasks` | `gws tasks tasks list/insert/update` |
| Create meeting links | `meet` | `gws meet spaces create` |
| Create forms, read responses | `forms` | `gws forms forms create`, `gws forms forms responses list` |
| Manage users, groups, devices | `admin` | `gws admin users list`, `gws admin groups list` |
| Manage notes | `keep` | `gws keep notes list` |
| Run/deploy Apps Script projects | `apps-script` | `gws apps-script projects list` |
| Audit logs and usage reports | `admin-reports` | `gws admin-reports activities list` |
| Manage security alerts | `alertcenter` | `gws alertcenter alerts list` |
| Manage identity and groups | `cloudidentity` | `gws cloudidentity groups list` |
| Subscribe to Workspace events | `events` | `gws events subscriptions create` |
| Manage Google Vault (eDiscovery) | `vault` | `gws vault matters list` |
| Manage Workspace licenses | `licensing` | `gws licensing licenseAssignments list` |
| Manage Google Classroom | `classroom` | `gws classroom courses list` |

## Environment Variables Reference

| Variable | Description |
|:---------|:------------|
| `GOOGLE_WORKSPACE_CLI_TOKEN` | Pre-obtained OAuth access token |
| `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` | Path to credentials JSON (service account or exported) |
| `GOOGLE_WORKSPACE_CLI_ACCOUNT` | Default account email |
| `GOOGLE_WORKSPACE_CLI_IMPERSONATED_USER` | User to impersonate (domain-wide delegation) |
| `GOOGLE_WORKSPACE_CLI_SANITIZE_TEMPLATE` | Default Model Armor template |
| `GOOGLE_WORKSPACE_CLI_SANITIZE_MODE` | `warn` or `block` |

## Troubleshooting

| Error | Fix |
|:------|:----|
| "Access blocked" or 403 during login | Add yourself as a test user in OAuth consent screen |
| "Google hasn't verified this app" | Click Advanced → Continue (safe for personal use) |
| Too many scopes error | Use `gws auth login -s drive,gmail,sheets` to select fewer services |
| `gcloud` CLI not found | Install gcloud or set up OAuth manually in Cloud Console |
| `redirect_uri_mismatch` | Re-create OAuth client as Desktop app type |
| `accessNotConfigured` | Enable the required API at the URL shown in the error, then retry |
| Stale credentials | Run `gws auth login` to re-authenticate |

## Resources

- **GitHub Repo**: https://github.com/googleworkspace/cli
- **npm Package**: https://www.npmjs.com/package/@googleworkspace/cli
- **Skills Index**: https://github.com/googleworkspace/cli/blob/main/docs/skills.md
- **Google Discovery API**: https://developers.google.com/discovery
- **Google Workspace APIs**: https://developers.google.com/workspace
