# Gmail Multi-Label Workflow Architecture

```mermaid
flowchart TD
    A[Gmail Trigger] --> B[Normalize Email Metadata]
    B --> C[Loop Over Items<br/>Batch Size = 1]
    C --> D[Check message_id in Email_Log]
    D --> E{Already processed?}

    E -- Yes --> C
    E -- No --> F{Has attachment?}

    F -- No --> G[Prepare no-attachment record]
    G --> H[(Google Sheets: Email_Log)]
    H --> C

    F -- Yes --> I[Find/Create label folder]
    I --> J[Find/Create date folder]
    J --> K[Find/Create sender folder]
    K --> L[Search existing filenames]
    L --> M[Split attachments]
    M --> N[Collision-safe filename]
    N --> O[(Google Drive)]
    O --> P[Summarize uploaded attachments]
    P --> H

    C -->|Done| Q[Batch complete]
```

## Storage convention

```text
attachments/<matched-label>/<YYYY-MM-DD>/<sender-name>/
```

## Design notes

- Gmail `message_id` is the idempotency key.
- Trigger batches are serialized one email at a time.
- Drive folder find-or-create branches converge through capture/edit nodes.
- Attachment filename comparisons are case-insensitive.
- One email produces one log row even when it contains several attachments.
