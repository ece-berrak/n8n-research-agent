# Notion Database Schema

The Notion database the agent writes to has this column structure:

| Column | Type | Filled by | Purpose |
|---|---|---|---|
| **Topic** | Title (default page title) | AI | Short title for each research finding |
| **TLDR** | Text | AI | One-line summary |
| **Findings** | Text | AI | Detailed findings in markdown |
| **Sources** | Text | AI | Source URL(s) from web search |
| **Created** | Created time | Notion (auto) | When the row was added |

## Setup steps

### 1. Create the database

In Notion, create a new page → add a database (table view). Name it whatever you like; the n8n workflow will reference it by name.

Add columns matching the table above. **Important details:**

- `Topic` should be the database's **default Title column** (the primary column every Notion DB has). If you don't already have one called `Topic`, you can rename the default `Name` column to `Topic`.
- `Created` is a special type. When adding the column, choose "Created time" — Notion will auto-fill it for every new row.
- All other columns are "Text" type.

### 2. Connect the database to your Notion integration

n8n connects to Notion via an Integration. Each database needs to be explicitly shared with the integration to be accessible.

In the database:
1. Click the `•••` menu (top right of the database)
2. **Connections** → **Connect to** → select your n8n integration
3. Confirm

Without this, n8n's Notion node will say "database not found" or similar.

### 3. Configure the Notion tool in n8n

In the AI Agent → Notion tool node:

- **Resource**: `Database Page`
- **Operation**: `Create`
- **Database**: select your database from the dropdown
- **Title** (page title field): use the expression below for `Topic`
- **Properties**: add one entry per non-Title column

### 4. Use these expressions

All property values use `$fromAI()` — this tells the agent to generate the value at runtime. Important syntax notes:

- Wrap the expression in `={{ ... }}`
- Use **single quotes** inside `$fromAI()` (double quotes can break n8n parsing)
- Make sure the field is in **expression mode** (the `fx` button on the left of the input must be active/highlighted)

#### Title (Notion page title)

```
={{ $fromAI('topic', 'Short title of the research finding', 'string') }}
```

#### TLDR

```
={{ $fromAI('tldr', 'One-line summary', 'string') }}
```

#### Findings

```
={{ $fromAI('findings', 'Detailed findings in markdown', 'string') }}
```

#### Sources

```
={{ $fromAI('sources', 'A source URL from web search. Always provide a valid URL starting with https://', 'string') }}
```

## Common pitfalls

- **Notion returns `Invalid date`** for the `Created` column — don't pass a date from n8n at all; let Notion auto-fill it via the "Created time" column type.
- **Notion returns `expected to be rich_text`** — the column is Rich Text but the AI returned malformed data. Check that the `$fromAI` expression is in expression mode (fx).
- **Properties save as literal `$fromAI(...)` strings** — expression mode isn't active. Click the `fx` icon on the left of the value field.
- **`Invalid parameter key`** — a property name contains characters n8n's tool framework doesn't allow. Avoid `;` `:` `?` `/` and other punctuation in Notion column names. Use plain letters, numbers, spaces, hyphens.
