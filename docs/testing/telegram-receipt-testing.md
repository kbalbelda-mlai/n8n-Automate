# Telegram Receipt Workflow — Testing


## Test cases

| ID | Input | Purpose | Expected behavior | Result |
|---|---|---|---|---|
| T01 | Clear synthetic coffee-shop receipt | Baseline extraction | Structured fields extracted; image stored; one log row | PASS |
| T02 | Clear synthetic supermarket receipt | Layout variation | Different receipt format extracted and logged correctly | PASS |
| T03 | Partially degraded / folded receipt | Degraded-image handling | Workflow completes; readable fields extracted where possible; original image archived | PASS |
| T04 | Heavily degraded / folded receipt | Difficult computer-vision case | Workflow completes without discarding the source image; quality handling applied | PASS |
| T05 | Handwritten words on paper | Non-receipt / invalid-input case | Classified as `UNRECOGNIZABLE`; image retained; record logged | PASS |

## Result summary

- **Formal cases executed:** 5
- **Passed:** 5
- **Failed:** 0

## T01 — Clear receipt

The first synthetic receipt represents a clean Philippine-style receipt with merchant details, date/time, receipt number, total, VAT, and payment method.

The test validates the complete happy path:

```text
Telegram photo
→ Gemini extraction
→ structured JSON
→ Drive upload
→ Receipt_Log row
```

## T02 — Different clear layout

The second image uses a different retail receipt layout. This validates that the extraction prompt is not tied to one fixed visual template.

## T03 — Partially degraded receipt

The third test introduces realistic image-quality issues such as folds and wrinkles, uneven lighting, mild fading / lower contrast, and slight perspective distortion.

The goal is not perfect transcription. The pass criterion is that the workflow completes normally, preserves the original image, and records whatever structured information can be reliably extracted.

## T04 — Heavily degraded receipt

The fourth receipt is intentionally more difficult for computer vision while remaining somewhat human-readable.

The pass criterion is graceful handling:

- no workflow crash;
- no requirement that every field be present;
- source image retained in Drive; and
- final record still produced with the appropriate quality assessment.

## T05 — Handwritten non-receipt

The fifth input is handwritten words on paper rather than an actual receipt.

Observed outcome:

```text
quality_status = UNRECOGNIZABLE
```

This is considered the correct behavior because the system should not invent merchant, transaction, or payment information from an image that does not reasonably represent a receipt.


### Workflow overview

`docs/screenshots/telegram-workflow-overview.png`

### Clear extraction

`docs/screenshots/telegram-test-clear-extraction.png`


### Unrecognizable input

`docs/screenshots/telegram-test-unrecognizable.png`

### Final Receipt_Log

`docs/screenshots/telegram-test-receipt-log.png`

### Drive output

`docs/screenshots/telegram-test-drive-output.png`
