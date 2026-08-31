# Telegram Receipt Workflow Architecture

```mermaid
flowchart TD
    A[Telegram Trigger] --> B{Photo received?}
    B -- No --> Z[End]
    B -- Yes --> C[Normalize submission]
    C --> D[Check submission_id in Receipt_Log]
    D --> E{Already processed?}
    E -- Yes --> Z
    E -- No --> F[Download highest-resolution photo]

    F --> G[Load external extraction prompt]
    G --> H{Prompt available?}
    H -- Yes --> I[Use external prompt]
    H -- No --> J[Use fallback prompt]
    I --> K[Merge prompt + Telegram context]
    J --> K

    K --> L[Gemini image analysis]
    L --> M[Parse structured JSON]
    M --> N[Restore image + submission context]
    N --> O[Assess receipt quality]

    O --> P[Find/Create submission-date folder]
    P --> Q[(Google Drive: Original Image)]
    Q --> R[Prepare final record]
    R --> S[(Google Sheets: Receipt_Log)]
```

## Core design decisions

### Submission-date storage

Drive grouping is based on the Telegram submission date:

```text
receipts/<submission-date>/receipt_<submission_id>.jpg
```

This is intentionally independent of the transaction date printed on the receipt.

### External prompt with fallback

The Gemini prompt is stored externally for easier editing, while a built-in fallback keeps the workflow operational if the external prompt is unavailable.

### Audit retention

The original Telegram image is uploaded regardless of extraction quality. A degraded or unrecognizable receipt therefore remains available for later human review.

### Idempotency

`submission_id` is used as the duplicate-processing key before Gemini analysis and Drive upload.

### Structured extraction

The AI output is parsed into a fixed schema before writing to the final log, preventing free-form model text from being written directly into the operational record.
