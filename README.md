# Gmail AI Customer-Support Automation (n8n)

An n8n workflow that monitors a Gmail support inbox, answers common questions from a Google Sheets FAQ, uses Google Gemini only to answer from approved FAQ content, and escalates urgent or unresolved requests to a human with a trackable ticket.

**Author:** Ayesha Imran
**Internship:** Logitrix Solutions
**Instructor:** Mohsin

## Features
- Polls Gmail every minute for unread Inbox mail (skips own mail and Promotions/Social/Forums/Updates)
- Ignores no-reply, bounce and system senders; never replies to its own escalation emails (loop-safe)
- FAQ keyword matching from a Google Sheet
- Gemini 2.5 Flash classifies "is this a support request?" and answers **only** from the FAQ (returns `NO_ANSWER` otherwise)
- Urgent / sensitive keywords (human, agent, urgent, refund, billing, complaint, hacked, security, chargeback, cancel) go straight to a person
- Ticket logging in Google Sheets, agent notification by email, holding reply with a ticket ID
- Fail-safe: if Gemini is unavailable, keyword and urgent rules are used so no email is lost
- Personal / newsletter mail is left untouched and unread

## Workflow
```
Gmail Trigger -> Get FAQs -> Extract Fields -> Ignore automated?
  -> FAQ Match -> Build AI Request -> Gemini AI -> Parse AI Answer -> Is support email?
      -> Urgent? --yes--> Prepare Ticket -> Log Ticket -> Notify Agent -> Reply Holding -> Mark Read
      -> Has answer? --yes--> Send Reply -> Mark Read
                    --no---> Prepare Ticket (as above)
```

## Files
| File | Purpose |
|---|---|
| `Gmail_Support_Bot_FINAL.json` | Importable n8n workflow |
| `faq_sample.csv` | Starter FAQ rows |

## Setup
1. **Google Sheet** with two tabs:
   - `FAQs`: `keywords | question | answer` (import `faq_sample.csv`)
   - `Tickets`: `ticket_id | created_at | channel | customer_id | customer_name | message | reason | status`
2. **n8n credentials:** Gmail OAuth2, Google Sheets OAuth2, and Google Gemini API (free key from aistudio.google.com).
3. In n8n: **Workflows -> Import from file** -> select `Gmail_Support_Bot_FINAL.json`.
4. Set your Google Sheet ID in the `Get FAQs` and `Log Ticket` nodes, your support email in `Notify Agent`, and assign credentials to any red node.
5. Activate the workflow.

## Testing
| Test | Expected result |
|---|---|
| "How do I reset my password?" | Threaded FAQ reply, email marked read |
| Support question not in FAQ | Ticket logged, agent emailed, holding reply with ticket ID |
| "I want a refund, urgent" | Ticket logged, agent emailed, holding reply |
| Mail from a no-reply address | Ignored |
| Personal / university mail | No reply, stays unread |
| Gemini unavailable | Fallback to keyword/urgent rules |

## Security
No tokens or passwords are stored in the workflow file. Keep all secrets in n8n Credentials, and do not share screenshots that show customer data or keys.

## Limitations / Future work
- Google Sheets is for small volumes; move tickets to a real help-desk database for production.
- Free Gemini tier has rate limits.
- Possible additions: sentiment/priority, multilingual FAQ, ticket dashboard.
