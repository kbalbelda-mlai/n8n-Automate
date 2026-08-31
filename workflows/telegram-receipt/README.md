# Telegram Receipt Processing Automation

Processes receipt photos sent through Telegram, extracts structured receipt information with Google Gemini, stores the original image in Google Drive, and logs the extracted record to Google Sheets.

## Workflow

```text
Telegram Trigger
→ Validate photo input
→ Normalize submission metadata
→ Duplicate submission check
→ Download highest-resolution Telegram photo
→ Load external extraction prompt
   └─ Fallback to built-in prompt if unavailable
→ Gemini image analysis
→ Parse structured JSON
→ Restore submission + image context
→ Assess receipt quality
→ Find/Create submission-date folder
→ Upload original image to Google Drive
→ Prepare final record
→ Append to Receipt_Log
```

## Main capabilities

- Accepts receipt photos through a Telegram bot.
- Uses the highest-resolution Telegram photo variant available.
- Uses Google Gemini for multimodal receipt extraction.
- Supports an externally editable prompt with a built-in fallback.
- Extracts common Philippine receipt and invoice fields.
- Separates the printed receipt date from the Telegram submission date.
- Groups original images in Google Drive by submission date.
- Retains the original image even when extraction quality is poor.
- Assigns a deterministic submission identifier using Telegram chat ID + message ID.
- Prevents reprocessing of an already logged `submission_id`.
- Writes one Google Sheets record per processed photo.

## Extracted fields

The final `Receipt_Log` schema is:

| Field | Description |
|---|---|
| `submission_id` | Unique Telegram submission identifier |
| `submission_timestamp` | Time the image was received |
| `telegram_user` | Telegram sender identifier/name |
| `merchant_name` | Extracted business or merchant name |
| `receipt_date` | Transaction date printed on the receipt |
| `receipt_time` | Transaction time, when available |
| `receipt_number` | Receipt / invoice / reference number |
| `total_amount` | Final amount payable or paid |
| `currency` | Normalized currency code, typically PHP |
| `merchant_tin` | Merchant TIN when readable |
| `merchant_address` | Merchant address when readable |
| `vat_amount` | VAT amount when available |
| `discount_amount` | Discount amount when available |
| `payment_method` | Cash, card, GCash, Maya, etc. when identifiable |
| `drive_file` | Logical Drive path of the stored image |
| `quality_status` | Extraction-quality classification |
| `processing_status` | Workflow completion state |

## Receipt interpretation

The Gemini prompt is designed for Philippine receipt terminology and differentiates values such as:

- `TOTAL`, `TOTAL DUE`, `AMOUNT DUE`, `GRAND TOTAL`, and `NET TOTAL`
- cash tendered versus final transaction total
- official receipt / sales invoice / transaction numbers
- merchant TIN versus customer TIN
- VAT, VATable sales, VAT-exempt sales, and zero-rated sales
- senior citizen / PWD / promotional discounts
- cash, credit/debit card, GCash, Maya, and other payment references

The model is instructed to return `null` rather than invent unreadable information.

## Quality handling

The workflow evaluates three core fields:

- `merchant_name`
- `receipt_date`
- `total_amount`

Possible quality outcomes are:

```text
SUCCESS
REVIEW_REQUIRED
UNRECOGNIZABLE
```

A poor-quality or non-receipt image can still complete the automation successfully. The original image is retained in Drive for audit even if the extracted receipt content is incomplete or unrecognizable.

## Google Drive organization

Original images are grouped by **Telegram submission date**, not by the transaction date printed on the receipt:

```text
telegram-receipt/
└── receipts/
    └── <YYYY-MM-DD>/
        ├── receipt_<submission_id>.jpg
        └── ...
```

## Prompt configuration

The operational extraction prompt is stored externally so it can be edited without changing the workflow itself.

Conceptual location:

```text
telegram-receipt/config/receipt_extraction_prompt
```

If the external prompt is unavailable or empty, the workflow can fall back to a built-in default prompt.

## Setup

1. Import the final `telegram-receipt.json` into n8n.
2. Connect your own Telegram, Google Gemini, Google Drive, and Google Sheets credentials.
3. Create a Telegram bot with BotFather and configure the Telegram Trigger.
4. Create the `Receipt_Log` sheet using the schema above.
5. Create the Drive roots used for `telegram-receipt/config` and `telegram-receipt/receipts`.
6. Add the external extraction prompt or retain the built-in fallback.
7. Publish/activate the workflow.

## Validation evidence

The final repository can use these screenshots:

- `docs/screenshots/telegram-workflow-overview.png`
- `docs/screenshots/telegram-test-clear-extraction.png`
- `docs/screenshots/telegram-test-degraded.png`
- `docs/screenshots/telegram-test-unrecognizable.png`
- `docs/screenshots/telegram-test-receipt-log.png`
- `docs/screenshots/telegram-test-drive-output.png`

See `docs/testing/telegram-receipt-testing.md` for the formal test summary.
