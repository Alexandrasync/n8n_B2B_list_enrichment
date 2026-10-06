# B2B Outreach — n8n Workflows

> Русская версия: [README.ru.md](README.ru.md)

A set of [n8n](https://n8n.io) workflows for cold B2B outreach prep: filtering companies by relevance and generating personalized email fragments.

The current prompts target agencies and communities that organize conferences, forums, and corporate events (business photography services). You can adapt the prompts for another niche.

## Project layout

```text
├── README.md
├── README.ru.md
├── .gitignore
└── workflows/
    ├── Валидация базы.json
    └── Персонализация сообщений.json
```

| File | Purpose |
|------|---------|
| `workflows/Валидация базы.json` | Fetch company websites from Google Sheets, clean HTML, score relevance with an LLM (YES/NO) |
| `workflows/Персонализация сообщений.json` | Brand-friendly company name, dative case form, personalized paragraph for the email |

## How it works

### 1. Lead list validation (`Валидация базы`)

1. Reads rows from Google Sheets.
2. For each company, requests the site from the `Основной сайт` column.
3. Cleans HTML and truncates text (`Limit Text`, up to ~8000 characters).
4. Via Ollama (`gemma4:31b`) answers strictly **ДА** (YES) or **НЕТ** (NO) — whether the company may need a business photographer.
5. Writes results back to the sheet (`Limit Text`, relevance flag, site open errors).

### 2. Message personalization (`Персонализация сообщений`)

1. Takes rows that already have `Limit Text` filled in.
2. **GigaChat** — clean everyday company name (no LLC/ООО and no quotes).
3. **GigaChat** — company name in the dative case for email copy.
4. **Ollama** — personalized paragraph (1–2 sentences, ~120–180 characters) based on company activity.
5. Updates the corresponding Google Sheets columns.

Recommended order: validation first, then personalization.

## Requirements

- [n8n](https://docs.n8n.io/hosting/) (self-hosted or cloud)
- Google Sheets + OAuth2 credentials in n8n
- [Ollama](https://ollama.com/) with the `gemma4:31b` model (or change the model in the nodes)
- [GigaChat](https://www.npmjs.com/package/n8n-nodes-gigachat) community node (`n8n-nodes-gigachat`) — personalization workflow only
- n8n LangChain Ollama node (`@n8n/n8n-nodes-langchain`)

## Quick start

### 1. Import

1. Open n8n → **Workflows** → **Import from File**.
2. Import both JSON files from `workflows/`.

### 2. Connect credentials

After import, re-link credentials (the export contains names/ids only, no secrets):

| Credential | Used in |
|------------|---------|
| Google Sheets OAuth2 | Both workflows |
| Ollama API | Both workflows |
| GigaChat | Personalization only |

### 3. Google Sheets

The JSON uses placeholders — after import, pick your own document and sheet in every Google Sheets node:

| Placeholder | What to set |
|-------------|-------------|
| `YOUR_GOOGLE_SHEETS_DOCUMENT_ID` | Spreadsheet ID from the URL (`/d/<ID>/edit`) |
| `YOUR_SHEET_GID_OR_NAME` / `YOUR_SHEET_NAME` | Worksheet with your data |
| `YOUR_SPREADSHEET_NAME` | Display name of the spreadsheet (optional) |

Expected columns (names matter — mappings depend on them):

| Column | Role |
|--------|------|
| `Наименование` | Row key (matching column) |
| `Основной сайт` | URL used for validation |
| `Limit Text` | Cleaned site text / LLM context |
| `Подходит ли компания, да/нет` | Validation result |
| `Ошибка в открытии сайта` | HTTP / open errors |
| `Общеупотребительное название компании` | Brand name without legal form |
| `Название компании в дательном падеже` | Form used in email copy |
| `Персонализированный параграф` | Ready personalization paragraph |

Extra columns (email, phone, director, etc.) may exist in the sheet — the workflows do not necessarily fill them.

### 4. Run

Both workflows are started manually (**Execute workflow**). Processing runs in batches with a `Wait` pause to avoid overloading the LLM and target sites.

## Adapting to your niche

- Edit the prompt in the `Message a model_cloud` node in the validation workflow (YES/NO criterion).
- Edit the personalized-paragraph prompt in the personalization workflow.
- Swap the Ollama / GigaChat model in node parameters if needed.

## Security

The JSON files contain **no** passwords or API keys. Google Sheets IDs are replaced with placeholders. After import, re-bind credentials in n8n; keep secrets only in Credentials / `.env` (do not commit them).

## License

You may use and modify these workflows for your own projects. Add a `LICENSE` file if needed (MIT, Apache-2.0, etc.).
