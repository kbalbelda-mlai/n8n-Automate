# n8n-Automate — Setup

This guide prepares the two exported workflows in this repository for use in another n8n environment.

> The repository contains **workflow definitions and controlled test fixtures**, not reusable account credentials. Connect your own accounts and replace environment-specific resource identifiers after import.

---

## 1. Requirements

You need:

- an n8n Cloud workspace or compatible n8n instance;
- a Gmail account;
- Google Drive;
- Google Sheets;
- a Telegram account and bot;
- Google Gemini access;
- permission to create OAuth/API credentials for the services above.

The project was validated on **n8n Cloud**.

---

## 2. Import the Workflows

Import:

```text
workflows/gmail-multilabel/gmail-multilabel.json
workflows/telegram-receipt/telegram-receipt.json
```

In n8n, use the workflow menu and choose **Import from File**.

Official reference:

https://docs.n8n.io/build/manage-workflows/export-and-import/

After import, keep both workflows **unpublished** until the credentials and resource mappings below are complete.

---

# Workflow 1 — Gmail Multi-Label Automation

## 3. Create the Gmail Labels

The current workflow is designed around four labels:

```text
n8n-testing_label01
n8n-testing_label02
n8n-testing_label03
n8n-testing_label04
```

You may keep these names or replace them with your own.

The Gmail Trigger currently searches for the four labels using an OR-style Gmail search query.

### Important: replace account-specific Gmail label IDs

The `normalize_email_metadata` node contains a mapping from Gmail's internal `Label_*` identifiers to the readable label names.

Those internal IDs are **specific to the Gmail account used to build the workflow**.

After connecting your Gmail credential:

1. create or select your four labels;
2. fetch a test email containing each label;
3. inspect the trigger's `labelIds`;
4. replace the four hard-coded `Label_*` keys in `normalize_email_metadata`.

The readable label values can remain:

```text
n8n-testing_label01
n8n-testing_label02
n8n-testing_label03
n8n-testing_label04
```

unless you intentionally rename them.

---

## 4. Connect Gmail

Create or select a Gmail OAuth credential in:

```text
Gmail[Trigger] - n8n-Automate
```

The current trigger configuration uses:

- Event: **Message Received**
- Poll interval: **5 minutes**
- Simplified output: **off**
- Read status: **read and unread**
- Attachment download: **on**
- Attachment prefix: `attachment_`

The 5-minute interval was used for formal batch testing. You may change it for your own environment.

Official Gmail Trigger documentation:

https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.gmailtrigger/

---

## 5. Create the Gmail Log Sheet

Create a Google Sheet with these headers:

```text
message_id
received_timestamp
sender_name
sender_email
subject
matched_label
has_attachment
attachment_count
drive_folder
processing_status
```

Then reconnect the appropriate spreadsheet/sheet in:

```text
check_message_already_logged
log_email_to_sheet
```

`message_id` is used as the duplicate-processing key.

---

## 6. Create the Gmail Drive Root

Create an attachment root folder, for example:

```text
n8n-Automate/
└── gmail-multilabel/
    └── attachments/
```

Reconnect that `attachments` folder in the Google Drive nodes that search/create the label hierarchy.

The workflow creates:

```text
attachments/<matched-label>/<received-date>/<sender-name>/
```

If an attachment name already exists in the destination sender folder, the workflow generates:

```text
file.pdf
file_2.pdf
file_3.pdf
```

rather than overwriting the existing file.

---

# Workflow 2 — Telegram Receipt Processing

## 7. Create a Telegram Bot

Create a Telegram bot using **BotFather**, then create/select the Telegram credential in n8n.

Reconnect the credential in:

```text
telegram_trigger
download_receipt_photo
```

The workflow accepts photos sent directly to the bot.

When using the Telegram Trigger, avoid running the test webhook and published production webhook at the same time with the same bot. Telegram supports one registered webhook per bot, so switching between test and production can replace the previous webhook.

Official references:

https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.telegramtrigger/

https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.telegram/

---

## 8. Connect Google Gemini

Create/select the Google Gemini credential in:

```text
gemini_analyze_image
```

The current export is configured for:

```text
models/gemini-3.6-flash
```

If that model is unavailable in your n8n environment, choose a compatible Gemini model that supports image analysis.

Official Google Gemini node documentation:

https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.googlegemini/

---

## 9. Create the Receipt Log Sheet

Create a Google Sheet with these headers:

```text
submission_id
submission_timestamp
telegram_user
merchant_name
receipt_date
receipt_time
receipt_number
total_amount
currency
merchant_tin
merchant_address
vat_amount
discount_amount
payment_method
drive_file
quality_status
processing_status
```

Reconnect the spreadsheet/sheet in:

```text
check_submission_already_logged
log_receipt_to_sheet
```

The duplicate-processing key is:

```text
submission_id = <telegram_chat_id>_<telegram_message_id>
```

---

## 10. Create the Telegram Drive Structure

Create:

```text
n8n-Automate/
└── telegram-receipt/
    ├── config/
    └── receipts/
```

Reconnect the account-specific Google Drive folders in:

```text
search_receipt_prompt
search_submission_date_folder
create_submission_date_folder
upload_original_receipt
```

Original photos are stored as:

```text
receipts/<submission-date>/receipt_<submission_id>.jpg
```

The storage date is the **Telegram submission date**, not necessarily the date printed on the receipt.

Official Google Drive node documentation:

https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googledrive/

---

## 11. Configure the External Receipt Prompt

Inside:

```text
telegram-receipt/config/
```

create a Google Doc or text-readable file named:

```text
receipt_extraction_prompt
```

The workflow searches for this file by name, downloads it, converts a Google Doc to plain text, and uses it as the Gemini prompt.

If the file is missing or empty, the workflow can use its built-in fallback prompt.

The prompt is designed to interpret Philippine receipt terminology such as:

- official receipt / sales invoice;
- TIN / VAT REG TIN;
- VATable sales / VAT amount;
- senior citizen / PWD discount;
- cash / card / GCash / Maya;
- PHP / ₱; and
- final transaction total versus cash tendered, change, subtotal, or VAT.

---

## 12. Timezone

Both workflows normalize operational timestamps to:

```text
Asia/Manila
```

If using the workflows in another location, update the DateTime expressions in:

```text
normalize_email_metadata
normalize_telegram_submission
```

and any downstream code that assumes Philippine time.

---

## 13. Publish the Workflows

After reconnecting credentials and resources:

1. run a manual test;
2. confirm Google Drive and Google Sheets output;
3. publish the workflow.

Trigger-based n8n workflows run automatically only after they are published.

Official reference:

https://docs.n8n.io/build/understand-workflows/save-and-publish-workflows/

---

# Public Repository Safety

## 14. Sanitize Workflow Exports Before Publishing

n8n workflow JSON exports can contain environment-specific metadata even when credential secrets themselves aren't embedded.

Before public release, review both JSON files and remove or replace:

- credential IDs;
- Google Spreadsheet document IDs;
- Google Drive folder IDs;
- cached Google Drive / Sheets URLs;
- Telegram webhook IDs;
- n8n workflow IDs;
- n8n `versionId`;
- n8n `instanceId`.

Do **not** publish:

- bot tokens;
- Google API keys;
- OAuth tokens;
- credential exports;
- `.env` files containing secrets.

After sanitizing, import the public JSON once into a temporary workflow to verify that the structure remains valid and that n8n simply asks the user to reconnect the required resources.

---

## 15. Screenshot Privacy

Before making the repository public, review the screenshots for:

- real sender email addresses;
- Gmail message IDs;
- Telegram usernames;
- Telegram chat IDs embedded in `submission_id`;
- Drive filenames that include Telegram chat IDs.

Crop, blur, or redact these values when they are not necessary to demonstrate the workflow.

---

## 16. Optional Gmail Auto-Label Rules

For repeatable testing, Gmail filters can assign labels from subject tokens.

Example:

```text
N8N-L01 → n8n-testing_label01
N8N-L02 → n8n-testing_label02
N8N-L03 → n8n-testing_label03
N8N-L04 → n8n-testing_label04
```

A subject containing two tokens can intentionally receive two monitored labels.

---

## 17. Official References

- n8n workflow import/export  
  https://docs.n8n.io/build/manage-workflows/export-and-import/

- n8n publishing  
  https://docs.n8n.io/build/understand-workflows/save-and-publish-workflows/

- Gmail Trigger  
  https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.gmailtrigger/

- Google Drive  
  https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googledrive/

- Google Sheets  
  https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/

- Telegram  
  https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.telegram/

- Telegram Trigger  
  https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.telegramtrigger/

- Google Gemini  
  https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.googlegemini/
