# Gmail Multi-Label Automation

Automates Gmail intake across multiple monitored labels, logs email metadata to Google Sheets, and stores attachments in Google Drive using a structured label/date/sender hierarchy.

## Workflow

```text
Gmail Trigger
→ Normalize metadata
→ Loop Over Items (one email at a time)
→ Duplicate check
→ Attachment check
   ├─ No attachment → Log
   └─ Attachment(s)
      → Find/Create label folder
      → Find/Create date folder
      → Find/Create sender folder
      → Check existing filenames
      → Split attachments
      → Collision-safe rename
      → Upload to Google Drive
      → Summarize
      → Log
```

## Main capabilities

- Monitors four Gmail labels from one Gmail Trigger.
- Preserves multiple monitored labels found on the same message.
- Processes multiple messages returned during one polling cycle sequentially.
- Prevents duplicate processing using Gmail `message_id`.
- Detects emails with and without attachments.
- Supports multiple attachments within one message.
- Groups Drive uploads by monitored label, received date, and sender.
- Prevents filename overwrites using `_2`, `_3`, and later suffixes.
- Writes one Google Sheets record per email.

## Monitored labels

- `n8n-testing_label01`
- `n8n-testing_label02`
- `n8n-testing_label03`
- `n8n-testing_label04`

## Logged fields

| Field | Description |
|---|---|
| `message_id` | Gmail message identifier |
| `received_timestamp` | Timestamp normalized to Asia/Manila |
| `sender_name` | Parsed sender display name |
| `sender_email` | Parsed sender address |
| `subject` | Email subject |
| `matched_label` | One or more monitored labels |
| `has_attachment` | Whether attachments were found |
| `attachment_count` | Number of attachments |
| `drive_folder` | Logical Drive storage path |
| `processing_status` | `NO_ATTACHMENT` or `ATTACHMENTS_SAVED` |

## Drive organization

```text
attachments/
└── <matched-label>/
    └── <YYYY-MM-DD>/
        └── <sender-name>/
            └── files
```

For emails that match multiple monitored labels, the combined `matched_label` value is retained as the grouping value rather than duplicating the attachment into several label folders.

## Collision handling

If a filename already exists in the destination sender folder:

```text
report.pdf
report_2.pdf
report_3.pdf
```

The existing file is not overwritten.

## Setup

1. Import the final `gmail-multilabel.json` into n8n.
2. Connect your own Gmail, Google Drive, and Google Sheets credentials.
3. Create the four monitored Gmail labels or update the label mappings.
4. Create an `Email_Log` sheet with the fields listed above.
5. Configure the Google Drive `attachments` root folder.
6. Publish/activate the workflow.

## Validation evidence

The final repository uses these screenshots:

- `docs/screenshots/gmail-workflow-overview.png`
- `docs/screenshots/gmail-test-clean-state.png`
- `docs/screenshots/gmail-test-email-log.png`
- `docs/screenshots/gmail-test-drive-output.png`

See `docs/testing/gmail-multilabel-testing.md` for the formal test matrix.
