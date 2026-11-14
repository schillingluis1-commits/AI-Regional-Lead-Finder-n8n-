# AI Regional Lead Finder (n8n)

An n8n workflow that finds local companies in a chosen region and industry, reads their website, and uses a **locally hosted LLM (Ollama)** to rate how well each company fits an automation or AI consulting project. For good matches it writes a short outreach **draft** for manual review. Nothing is sent automatically.

Because the AI runs locally, no company data is sent to a third-party AI provider.

## What it does

1. Takes a region and a keyword (for example `Trier, Germany` + `metalbauer`) from one config node
2. Searches companies via the Google Places Text Search API
3. Skips companies that are already in your Google Sheet (no duplicates between runs)
4. Looks up each company's website via Place Details and downloads the page (2 second delay between requests)
5. Extracts the visible text (first 3,000 characters) and asks the local LLM to score the lead from 0 to 100, with a one-sentence reasoning
6. Logs every evaluated company to the sheet
7. For companies at or above the score threshold, checks they were not already contacted, then generates a short German outreach email draft
8. Saves the draft to the sheet for you to review. The workflow stops processing new companies while more than 14 drafts are waiting in the sheet (status `ready_to_send`)

## Workflow overview

![Workflow overview](screenshots/workflow-overview.png)

## Example output

![Example output](screenshots/example-output.png)

*(Sample data only.)*

## Requirements

- n8n (self-hosted with Docker or n8n Cloud)
- [Ollama](https://ollama.com) running locally with a model, default `llama3.1`
- A Google Cloud API key with the **Places API** enabled
- A Google account for Google Sheets (OAuth2 credential in n8n)

## Setup

1. In n8n: **Workflows → Import from file** → `workflow.json`
2. Create a Google Sheet with a tab named `CompanyList` and these header columns (the names must match exactly, including existing spelling):
   `place_id`, `name`, `address`, `website`, `score`, `reasoning`, `status `, `messaged`, `evaluated_at`, `email_subject`, `email_body`, `lat`, `lng`
   Note: `status ` has a trailing space in the workflow's lookup, so keep that header as is, or change both consistently.
3. Open the **Config** node and fill in:
   - `region`, `keyword`: what to search for
   - `batchTarget`: how many new companies per run
   - `scoreThreshold`: minimum score for a draft
   - `googleSheetId`: the ID from your sheet URL
   - `googlePlacesApiKey`: your key (see security note below)
   - `ollamaUrl`, `ollamaModel`: for Docker on the same host, `http://host.docker.internal:11434`
   - `senderName`, `senderCompany`: used in the generated draft
4. Select your own Google Sheets credential in the Sheets nodes
5. Run the workflow manually

## Security notes

- **Never commit real API keys or sheet IDs.** Keep them out of exported workflow files, or use n8n credentials and environment variables.
- Restrict your Google API key to the Places API and your server's IP in the Google Cloud console.

## Outreach and compliance

The workflow only creates drafts. You review each one before anything is sent. In Germany, unsolicited advertising emails to businesses are generally only allowed with at least presumed consent (UWG). Check the rules for your channel (email, contact form, letter, phone) before contacting anyone. This is not legal advice.

## Tech

n8n, Google Places API, Ollama (llama3.1), Google Sheets, Docker

## Possible extensions

- Add more criteria to the scoring prompt (company size, industry-specific signals)
- Send drafts to a review inbox instead of a sheet
- Run on a schedule instead of manually

---

Built by [YOUR_NAME] ([YOUR_COMPANY]).
