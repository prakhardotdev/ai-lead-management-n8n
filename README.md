# AI-Powered Lead Management System

An n8n workflow that captures leads via webhook, validates them, prevents duplicates, qualifies them with AI, stores them in HubSpot CRM, sends a personalized email, and routes sales alerts by lead category.

![Workflow canvas](docs/canvas.png)

## Problem
Small businesses receive leads from forms and websites but follow up manually. Leads get lost, duplicated, or answered late, and sales teams can't tell which leads matter most.

## Solution
An end-to-end automation:
Capture -> Validate -> Deduplicate -> Understand (AI) -> Store (CRM) -> Communicate (Email) -> Route (Alerts)

## Architecture

1. **Input & Validation**: Webhook receives the lead, fields are cleaned, and required fields plus email format are checked. Invalid leads go to a manual review email and never reach the AI.
2. **Duplicate Check**: HubSpot is searched by email. Existing leads only get their data refreshed. No AI call, no second email.
3. **AI Qualification**: Gemini scores the lead (0-100), assigns Hot/Warm/Cold, and gives a one-sentence reason as structured JSON. A second validation checks score range and category. Invalid output goes to manual review.
4. **CRM + Email**: The lead and AI results are saved to HubSpot. A second AI step writes a personalized email, sent via Gmail.
5. **Routing**: A Switch node sends an alert based on category (Hot / Warm / Cold).

## Tech Stack
n8n (self-hosted, Docker), HubSpot CRM, Google Gemini, Gmail, PowerShell for test requests.

## Key Design Decisions
- **Validation before AI**: saves API quota and avoids garbage in the CRM.
- **Idempotency**: the same email submitted twice creates one contact and sends one email.
- **AI output is validated**, not trusted.
- **Business rule**: existing leads are updated but not emailed again.

## Test Cases

| # | Test | Result |
|---|------|--------|
| 1 | New lead (Hot) | Full flow, CRM, email, Hot alert |
| 2 | Warm lead | Routed to Warm |
| 3 | Cold lead | Routed to Cold |
| 4 | Missing budget | Stopped at validation |
| 5 | Hindi requirement | Valid structured output |
| 6 | Same email twice | One contact, no second email |
| 7 | Missing email | Stopped at validation |
| 8 | Empty requirement | Stopped at validation |
| 9 | Invalid email (`rahul@abc`) | Stopped by regex check |
| 10 | Invalid AI output (score 150) | Sent to manual review |
| 11 | Very high budget | Valid score and category |
| 12 | Random text requirement | Handled without false high intent |

## Bugs Found During Testing
- Empty budget became `0` and passed validation. Fixed with a "greater than 0" check.
- Invalid email reached HubSpot and errored. Fixed with a regex check.
- Name appeared as "Rahul Sharma Sharma". Fixed by splitting first and last name.

## Known Limitations
- Runs on localhost. Needs hosting for production.
- Gemini free tier (20 requests/day) is too low for real traffic.
- Lead emails currently go to a test address.
- Single-field duplicate check (email only).

## Next Steps
Hosted deployment, paid API key with retry and error alerts, human approval before sending emails, and a dashboard for lead metrics.

## Author
Prakhar | github.com/prakhardotdev | linkedin.com/in/prakhardotdev
