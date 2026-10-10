# AI Sales Agent (n8n + Gemini + Google Sheets + Gmail)

Four importable workflows that automate cold outreach end to end.

| File | What it does |
|---|---|
| `1-outbound.json` | Weekdays 09:00. Reads leads with `status = new`, fetches each website, asks the AI model to score fit and write a personalized email, then sends it (or saves a Gmail draft) and logs the result. |
| `2-reply-handler.json` | Polls Gmail every minute. Matches replies to leads, classifies them (interested / question / not_now / unsubscribe / other), replies or alerts you, updates the sheet. |
| `3-follow-ups.json` | Weekdays 10:00. Sends up to 2 short follow-ups in the same thread to leads that have not replied after 3 days. |
| `4-error-alert.json` | Emails you when any workflow fails. |

## Lead status flow

`new` -> `contacted` (or `drafted`, `disqualified`) -> `interested` / `needs_review` / `follow_up_later` / `do_not_contact`

## Setup (about 20 minutes)

1. **Google Sheet**: create a sheet tab named `Leads`, import `leads-template.csv` (keep the header row exactly as is). Keep emails lowercase.
2. **n8n**: run n8n Cloud (free trial) or self-host with Docker. Import the four JSON files (Workflows > Import from file).
3. **Credentials** (create once, then select them in the nodes that show a warning):
   - *Google Sheets OAuth2* and *Gmail OAuth2* (Google Cloud console: enable the Gmail and Sheets APIs, create OAuth client).
   - *Header Auth* named "Gemini API Key": header name `x-goog-api-key`, value = a free API key from aistudio.google.com (no billing needed).
4. **Sheet ID**: paste your Google Sheet ID into every Google Sheets node (workflows 1, 2, 3).
5. **Config node** (first node in workflows 1-3, keep them identical): fill in `senderName`, `senderCompany`, `offer`, `icp`, `bookingLink`, `ownerEmail`. Defaults: 5 emails/day while testing (20 later), minimum fit score 7, `autoSend = false`.
6. In workflow 4, change the recipient address. In workflows 1-3: Settings > Error workflow > choose "Sales Agent 4 - Error alert".

## Testing order

1. Put 3 to 5 real leads (or your own other email addresses) in the sheet with `status = new`.
2. Run workflow 1 manually with `autoSend = false`. Check the Gmail drafts and the sheet. Tune `offer` / `icp` until drafts sound right.
3. Set `autoSend = true`, run again on a couple of test addresses you own, then reply from them to test workflow 2.
4. Activate all workflows only after steps 2 and 3 look right.

## About the free Gemini tier

- The workflows call Google's Gemini API. The model name lives in the **Config** node (`model`, default `gemini-flash-latest`). If you get a 404, copy a current Flash model ID from aistudio.google.com and paste it there.
- Free-tier limits are low (roughly 10-15 requests/minute and a few hundred to ~1,500/day). The default 20 leads/day fits comfortably.
- On the free tier Google may use your prompts to improve its models, and free use is meant for testing and learning. Do not send sensitive customer data, and switch to a paid key before using it commercially.

## Going fully automated safely

- Review about 20 drafts before turning `autoSend` on.
- Send from a separate domain or mailbox, with SPF/DKIM/DMARC set up. Start at 10-20 emails a day.
- Every email carries an unsubscribe line and replies containing "unsubscribe" set `do_not_contact`. Keep that, and respect local anti-spam laws (CAN-SPAM, GDPR).
- Replies the AI cannot answer from your `offer` text go to you, not to the lead.
- If the AI's response cannot be parsed, the lead stays `new` and is retried the next day.

## Ideas for later

Swap the Fetch Website step for Apollo/Hunter enrichment, add a LinkedIn or WhatsApp channel, store a knowledge base (pricing, FAQs) in a Google Doc and feed it into the reply prompt, or add a dashboard sheet with reply and meeting rates.
