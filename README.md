# AI Email Support Agent — n8n Portfolio Demo

## Summary

An n8n-based, **draft-only** customer-support email workflow for a fictional e-commerce business. Incoming Gmail messages are filtered, checked for duplicates against Google Sheets, classified using an OpenAI model, and routed to either a general inquiry response or an order lookup with email-address matching. Drafts are saved for human review and logging is updated by Gmail `messageId`.

## Core workflow

1. Gmail Trigger → Get a message → Normalize email fields.
2. Check Activity Log by `messageId`. Skip if already logged.
3. Mark new message `Processing` in the log.
4. Classify the incoming message with an OpenAI model; extract category, intent, and order number.
5. If an order is needed, query an Orders sheet and check that the sender address matches the stored customer email.
6. Create a Gmail **draft only** for verified orders, unverified customers, missing orders, or general inquiries.
7. Update the existing log record by `messageId` with draft ID and review outcome.

## Dependencies

- n8n (workflow was exported from n8n 2.43.7)
- Gmail OAuth2 credential with appropriate account access
- Google Sheets OAuth2 credential
- OpenAI API credential
- One spreadsheet with an `Orders` tab and an `Activity Log` tab

### `Orders` columns used by the workflow

`orderNumber`, `customerEmail`, `customerName`, `orderStatus`, `carrier`, `trackingNumber`, `estimatedDelivery`.

### `Activity Log` columns used by the workflow

`processedAt`, `messageId`, `threadId`, `senderEmail`, `subject`, `category`, `intent`, `orderNumber`, `outcome`, `draftId`, `reviewStatus`.

## Template setup

**This sanitized JSON is not immediately runnable**. After importing it into n8n:

1. Reconnect Gmail, Google Sheets, and OpenAI credentials (all credential references have been removed).
2. Select your own Google spreadsheet in each Sheets node and select the correct `Orders` or `Activity Log` tab. All original spreadsheet identifiers and URLs were removed.
3. Review `Mark Email Processing`: `outcome = Processing` and `reviewStatus = Processing` are explicitly mapped in this portfolio copy. The source export was missing the `reviewStatus` value; correct your live workflow too.
4. The `AI-TEST` Gmail subject filter remains enabled. Change it only after testing in a dedicated account.
5. Configure a separate error-handling workflow if required. Its reference was removed from this template and it is **not included** in this file.
6. Recreate a separate Processing Monitor workflow as needed (not included in this file) to identify stale `Processing` records.
7. Test all branches and validate no message-send operations exist before enabling the workflow. This workflow currently creates Gmail drafts only.

## Sanitization performed

- Removed pinned Gmail message data (including real email addresses).
- Removed credential references and credential IDs (no API keys were found in the original export).
- Replaced spreadsheet ID and cached URLs with placeholders.
- Removed original workflow instance ID, workflow ID, version ID, error-workflow reference, and node webhook IDs.
- Marked this template as inactive / unpublished.
- Preserved node prompts, conditions, connections, and draft-only behavior.
- Corrected missing `reviewStatus = Processing` mapping in the template only.

## Security and reliability caveats

**This is a tested portfolio demonstration, not a production-certified customer-support system.**

- Sender-address matching is a basic check, **not strong identity verification**. Sender headers can be forged, and sensitive order information should not be exposed without more robust account authentication/authorization.
- Spreadsheet read/check/write actions are not atomic. The production concurrency limit of 1 used in the local environment mitigates some overlap but doesn't guarantee exactly-once drafts, especially after partial failures or external changes.
- The main workflow does not automatically recover stuck email states. An additional external monitor and an error-handler workflow were built separately; they are absent from this export.
- The final logging steps overwrite `processedAt` with the completion time; storing start and completion timestamps separately would improve observability.
- Classification JSON is parsed without schema-validation/error-fallback logic; malformed model output can fail the run.
- Avoid sensitive customer data in test logs, exports, AI inputs, and portfolio screenshots; define retention, consent, and access policies before real use.
- Never share real API credentials or a copy of the persistent n8n volume.

## Demonstrated test scenarios

In local tests, the user confirmed successful automated runs for: general inquiry, verified order draft, mismatched customer email, missing order, and duplicate email suppression. These are functional demonstration results, not guarantees under production traffic.

## Project Screenshots

### 1. AI Email Agent Workflow

The n8n workflow handles incoming emails, AI classification, duplicate detection, order verification, Gmail draft creation, and activity logging.

![AI Email Agent Workflow](screenshots/Workflow%20Image.png)

### 2. Google Sheets Activity Log

An illustrative view of the Google Sheets tracking system used to record email classifications, order inquiries, draft outcomes, and review statuses.

![Activity Log](screenshots/Activity%20Flow%20Image.png)

### 3. AI-Generated Gmail Draft

An illustrative example showing how a customer inquiry is processed and transformed into a personalized email draft for human review before sending.

![AI-Generated Email Draft](screenshots/Email%20Draft%20Image.png)

