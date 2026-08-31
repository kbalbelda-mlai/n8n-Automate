# n8n-Automate — Testing

This document summarizes the final controlled validation of both workflows. Workflow-specific reports are available under [`docs/testing/`](docs/testing/).

---

## Test Environment

- **Automation platform:** n8n Cloud
- **Gmail workflow trigger:** Gmail Trigger
- **Gmail formal polling interval:** 5 minutes
- **Receipt workflow trigger:** Telegram Trigger
- **AI extraction:** Google Gemini
- **Storage:** Google Drive
- **Operational logs:** Google Sheets
- **Timezone:** Asia/Manila
- **Test data:** controlled Gmail attachments, synthetic receipt / invoice fixtures, and one handwritten non-receipt input

---

# Workflow 1 — Gmail Multi-Label Automation

## Objective

Validate:

- automatic processing across four monitored Gmail labels;
- messages with and without attachments;
- multiple attachments;
- multiple monitored labels on one message;
- several messages returned in the same polling interval;
- collision-safe Drive storage; and
- one Google Sheets row per email.

## Final Test Matrix

| ID | Scenario | Labels | Attachments | Pass criterion | Result |
|---|---|---|---:|---|---|
| **G01** | No attachment | L01 | 0 | One log row, no file upload, `NO_ATTACHMENT` | **PASS** |
| **G02** | Single PDF | L02 | 1 | One upload and one log row | **PASS** |
| **G03** | Multiple mixed attachments | L03 | 2 | Both files stored, one log row | **PASS** |
| **G04** | Fourth-label routing | L04 | 2 | L04 detected and both files stored | **PASS** |
| **G05** | Multi-label, no attachment | L01 + L03 | 0 | Both monitored labels retained | **PASS** |
| **G06** | Multi-label with attachment | L02 + L04 | 1 | Both labels retained and file stored | **PASS** |
| **G07** | Batch polling | L01 | 1 | Correct processing alongside other messages returned in the same poll | **PASS** |
| **G08** | Filename collision | L02 | 1 | Existing file preserved; later file receives a numeric suffix | **PASS** |
| **G09** | Three attachments | L03 | 3 | Three files stored and only one email row created | **PASS** |

### Gmail result

```text
Formal cases: 9
Passed:       9
Failed:       0
```

The final workflow uses **Loop Over Items with batch size 1** so each email completes its duplicate check and processing path before the next item from the trigger batch is released.

### Filename collision behavior

The collision test validates patterns such as:

```text
gmail_multiple_label_test_A.pdf
gmail_multiple_label_test_A_2.pdf
```

rather than overwriting the existing file.


Detailed report:

[`docs/testing/gmail-multilabel-testing.md`](docs/testing/gmail-multilabel-testing.md)

---

# Workflow 2 — Telegram Receipt Processing

## Objective

Validate:

- live photo intake through Telegram;
- structured Gemini extraction;
- more than one document layout;
- degraded-image handling;
- invalid / non-receipt handling;
- Drive archival by submission date; and
- Google Sheets logging.

## Final Test Fixtures

The current repository contains:

| ID | Fixture | Test purpose |
|---|---|---|
| **T01** | `examples/telegram/T01.png` — Bayani Mart retail receipt | Clear baseline receipt |
| **T02** | `examples/telegram/T02.png` — Brightbuild Facility Services service invoice | Alternate invoice/document layout |
| **T03** | `examples/telegram/T03.png` — folded / unevenly lit Ministop receipt | Partially degraded receipt |
| **T04** | `examples/telegram/T04.png` — heavily folded / low-contrast Southpoint receipt | Difficult but somewhat human-readable receipt |
| **T05** | `examples/telegram/T05.png` — handwritten words on lined paper | Non-receipt / invalid image |

## Final Test Matrix

| ID | Scenario | Pass criterion | Result |
|---|---|---|---|
| **T01** | Clear retail receipt | Structured transaction fields logged and original image stored | **PASS** |
| **T02** | Clean service invoice | Alternate layout extracted and archived | **PASS** |
| **T03** | Partially degraded receipt | Workflow completes, reliable fields are retained, original image is archived | **PASS** |
| **T04** | Heavily degraded receipt | Workflow completes gracefully and retains the source image for audit | **PASS** |
| **T05** | Handwritten non-receipt | No fabricated receipt; classified as `UNRECOGNIZABLE` and archived | **PASS** |

### Telegram result

```text
Formal cases: 5
Passed:       5
Failed:       0
```

The Telegram test suite intentionally evaluates workflow behavior rather than requiring every degraded image to produce perfect transcription. For T03 and T04, the important pass conditions are graceful processing, structured output where reliable, logging, and retention of the original image.

T05 is the negative control. Its handwritten text is not a receipt, and the observed `UNRECOGNIZABLE` result is the intended outcome.


Detailed report:

[`docs/testing/telegram-receipt-testing.md`](docs/testing/telegram-receipt-testing.md)

---

# Combined Result

| Workflow | Cases | Passed | Failed |
|---|---:|---:|---:|
| Gmail Multi-Label | 9 | 9 | 0 |
| Telegram Receipt | 5 | 5 | 0 |
| **Total** | **14** | **14** | **0** |

The controlled validation demonstrates that the two workflows perform their primary end-to-end requirements across normal inputs and selected edge cases.

---

## Evidence Interpretation

The repository intentionally keeps documentation evidence concise:

- **workflow screenshots** show the final n8n architecture;
- **Google Sheets screenshots** show persisted operational results;
- **Google Drive screenshots** show the resulting storage structure;
- **Gemini output screenshots** show structured extraction and invalid-input handling;
- **examples/** contains the controlled test inputs used for repeatable validation.

The public repository does not need every development or debugging execution. The test report describes the final validated implementation.

---

## Known Test Boundaries

### Gmail

- The final controlled suite used one sending account.
- Sender-folder naming is dynamic, but the final nine-case run did not use several independent sender identities.
- Gmail's internal label IDs are account-specific and must be reconfigured by an importing user.

### Telegram

- The controlled suite focuses on image submissions rather than text-only bot interaction.
- T01–T04 are controlled test fixtures rather than production receipts.
- Extraction quality naturally depends on image legibility and the available Gemini model.
- The workflow is designed to retain source images so uncertain results can be reviewed manually.

---

## Reproducing the Tests

### Gmail

1. Create the four monitored labels.
2. Configure Gmail filters or assign labels manually.
3. Publish the workflow.
4. Send test messages using the fixtures under `examples/gmail/`.
5. For the batch test, deliver several labeled messages between two poll cycles.
6. For the collision test, reuse a filename in the same label/date/sender destination.
7. Compare the final Sheet and Drive outputs against the test matrix above.

### Telegram

1. Publish the Telegram workflow.
2. Open the configured Telegram bot.
3. Send `T01.png` through `T05.png` one at a time.
4. Wait for each production execution to complete.
5. Confirm one final log row and one stored original image per submission.
6. Confirm that T05 is not treated as a valid receipt.

---

## Related Documentation

- [`README.md`](README.md)
- [`SETUP.md`](SETUP.md)
- [`workflows/gmail-multilabel/README.md`](workflows/gmail-multilabel/README.md)
- [`workflows/telegram-receipt/README.md`](workflows/telegram-receipt/README.md)
- [`docs/architecture/gmail-multilabel-flow.md`](docs/architecture/gmail-multilabel-flow.md)
- [`docs/architecture/telegram-receipt-flow.md`](docs/architecture/telegram-receipt-flow.md)
