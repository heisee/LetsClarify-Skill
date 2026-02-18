# Let's Clarify

[![smithery badge](https://smithery.ai/badge/letsclarify/letsclarify)](https://smithery.ai/server/letsclarify/letsclarify)

Human-in-the-loop form collection for AI agents. When your workflow needs structured human input before proceeding — approvals, decisions, reviews, data collection — Let's Clarify handles it.

**Website:** https://letsclarify.ai

## Installation

Add the skill to your Claude Code project:

```bash
claude install github:heisee/LetsClarify-Skill
```

Then set your API key:

```bash
export LETSCLARIFY_API_KEY="lc_..."
```

## How it works

1. **Your agent defines a form** — text fields, dropdowns, radios, checkboxes, file uploads
2. **Let's Clarify generates unique URLs** — one per human recipient
3. **Humans fill out the form** in their browser (hosted page or embedded widget)
4. **Your agent gets structured JSON back** via polling or webhook

No SDK. No dependencies. Just HTTP.

## Setup

Register for an API key:

```bash
curl -X POST https://letsclarify.ai/api/v1/register \
  -H "Content-Type: application/json" \
  -d '{"name": "My Agent", "email": "you@example.com"}'
```

Store the returned `api_key` (starts with `lc_`). It is shown only once.

## Example: Approval workflow

```bash
# 1. Create a form
curl -X POST https://letsclarify.ai/api/v1/forms \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $LETSCLARIFY_API_KEY" \
  -d '{
    "title": "Approve deployment to production",
    "context_markdown": "## Release v2.4.1\n- Fix auth bug\n- Add caching",
    "recipient_count": 2,
    "retention_days": 7,
    "schema": [
      { "id": "decision", "type": "radio", "label": "Decision", "required": true,
        "options": [
          { "value": "approve", "label": "Approve" },
          { "value": "reject", "label": "Reject" }
        ]},
      { "id": "notes", "type": "textarea", "label": "Notes" }
    ]
  }'

# Response: { "form_token": "xK9m2...", "recipients": ["uuid-1", "uuid-2"], ... }

# 2. Send links to humans
#    https://letsclarify.ai/f/{form_token}/{recipient_uuid}

# 3. Poll for results
curl -H "Authorization: Bearer $LETSCLARIFY_API_KEY" \
  "https://letsclarify.ai/api/v1/forms/{form_token}/results"
```

## Embed widget

Instead of sending links, you can embed forms directly into any web page:

```html
<script src="https://letsclarify.ai/embed.js"></script>
<div data-letsclarify-form="{form_token}"
     data-letsclarify-recipient="{recipient_uuid}">
</div>
```

The widget auto-renders the form, handles validation, file uploads, and submission. CSS is injected automatically.

## Field types

| Type | Description |
|------|-------------|
| `text` | Single-line text input |
| `textarea` | Multi-line text |
| `select` | Dropdown |
| `radio` | Radio buttons |
| `checkbox` | Single boolean |
| `checkbox_group` | Multiple checkboxes |
| `file` | File upload (PDF, images, etc.) |

All types support optional validation (`min_length`, `max_length`, `pattern`, `min_items`, `max_items`).

## API endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/v1/register` | No | Get an API key |
| DELETE | `/api/v1/register` | Yes | Delete your API key |
| POST | `/api/v1/forms` | Yes | Create a form |
| POST | `/api/v1/forms/:token/recipients` | Yes | Add more recipients (max 10,000) |
| GET | `/api/v1/forms/:token/summary` | Yes | Submission counts |
| GET | `/api/v1/forms/:token/results` | Yes | Full results (paginated, filterable) |
| DELETE | `/api/v1/forms/:token` | Yes | Delete form (requires X-Delete-Token) |
| GET | `/api/v1/embed/:token/:uuid` | No | Get form config (embed) |
| POST | `/api/v1/embed/:token/:uuid` | No | Submit form (embed) |

Auth: `Authorization: Bearer lc_...`

## Use cases

- **Approvals** — deploy gates, budget sign-offs, PR reviews
- **Data collection** — onboarding forms, surveys, intake forms
- **Document review** — attach context as Markdown, collect structured feedback with file uploads
- **Human-in-the-loop AI** — any step where an agent needs a human decision before proceeding

## Full API reference

See [`skills/letsclarify/SKILL.md`](skills/letsclarify/SKILL.md) for the complete API documentation.

## License

MIT — see [LICENSE](LICENSE)
