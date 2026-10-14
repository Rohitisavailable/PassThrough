# PassThrough

**Turn shoot expenses into invoice-ready client reimbursables.** PassThrough is a local-first tool for freelance photographers and videographers who pay for parking, travel, props, rush prints, and other client costs, then forget to bill them back. Drop a receipt, attach it to a shoot and client, check the extracted facts, approve the billable line, and export a CSV for your invoice.

## Backboard and open-weight model

PassThrough uses Backboard with the configured open-weight vision model (Qwen3.5 9B through OpenRouter by default) to extract receipt fields. It does not use a local AI model in Backboard mode: receipt images and scanned PDF pages go to Backboard directly, while searchable PDF text is extracted locally and sent. Client/project labels and a saved receipt copy stay on this computer. Backboard model usage may incur charges.

## Run locally on Windows

Install Python 3.12. This checkout already has a ready `.venv`; skip the venv creation command below here. On a fresh checkout, create it with `py -3.12 -m venv .venv`. Then run in PowerShell:

```powershell
cd C:\Users\tfroh\FreelanceGuard
# Skip this command if .venv already exists
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python app.py
```

Open <http://127.0.0.1:8001/>. The server binds to loopback. RapidOCR downloads its small OCR model files on first use; that download contains model weights, not your receipts.

Add your key to `.env` as `BACKBOARD_API_KEY=...`. Set `AI_PROVIDER=backboard`, `BACKBOARD_PROVIDER=openrouter`, `BACKBOARD_MODEL=qwen/qwen3.5-9b`, and `PORT=8001` there too. The app reads `.env` on startup; non-empty process environment variables take precedence. `.env` is excluded from Git. Receipt image/text content leaves this computer for Backboard; review Backboard and upstream model billing/privacy terms before using real client receipts.

## Deploy to Render

This app needs a **Web Service**, not a Static Site. The included `render.yaml` configures a free, single-user FastAPI demo with HTTPS Basic Auth and Backboard. Connect this GitHub repository in Render and create a Blueprint from `render.yaml`. In the prompted secret fields, enter `BACKBOARD_API_KEY` and a unique `APP_PASSWORD` of at least 16 characters. Keep both in Render's environment settings; never commit them. The username is `owner`.

The Free plan has ephemeral storage. Receipts and expense records can disappear after a restart or deploy, so use this for a demo with sample data only, not real client documents. The service is protected by browser Basic Auth; use a strong unique password and only access it over the Render HTTPS URL. The public `/healthz` endpoint reveals no receipt data.


## Workflow

1. Upload a receipt photo (JPG, PNG, WEBP) or PDF up to 12 MB. Backboard reads images; searchable PDFs are extracted locally, while scanned PDFs are rendered and sent to Backboard.
2. Enter the shoot/project and client.
3. The configured model suggests merchant, date, total, currency, and a short expense line. Unknown fields remain blank; the model is told not to calculate or convert amounts.
4. Review and correct every field. Mark the expense billable, then approve it. Only approved billable expenses appear in the CSV.
5. Download invoice lines and use them in your existing invoicing tool.

## Trust and data handling

Every model extraction starts in review; nothing is automatically approved or invoiced. The server validates dates, currency codes, and positive decimal amounts in code. Amounts in different currencies stay separate. Exact duplicate files are rejected. Receipt images, extracted fields, and approval state are saved in `data/` on this computer; deleting an expense also deletes its saved receipt. Do not expose this local app to an untrusted network. Review the source image before billing a client.

Scanned receipts can be hard to read, the model may miss or misread a field, and this first version does not connect to accounting software or calculate tax treatment. It creates a reviewable invoice-line CSV; it does not send invoices or payments.

## Stack

Python, FastAPI, SQLite, pdfplumber, pypdfium2, Backboard API, and Qwen3.5 9B via OpenRouter. The open-weight model reads receipt images and extracts invoice fields without per-merchant templates. Deterministic code owns money parsing, duplicate checks, review approval, storage, and export.
