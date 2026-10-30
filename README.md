# AI Support Ticket Triage
**Demo video:** [Watch the demo](https://www.loom.com/share/e670de263f924fa39d605af233eea209)

<img width="1287" height="565" alt="Workflow canvas for the AI support ticket triage system" src="https://github.com/user-attachments/assets/e7af7193-7d65-4c16-91b3-418d3555b59a" />

*The full workflow: two intake channels feed an AI classifier, a rule-based router sends high-urgency tickets to Slack, and every ticket is logged to Airtable.*

## The problem
Support requests rarely arrive through one channel. Some come from a web form, others from another tool or a website widget, and someone has to read each one to decide what needs attention first. When that decision is manual, urgent requests get buried behind routine ones, and nobody knows when something has slipped through.

## What I built
A workflow that takes tickets from two intake channels (a form and a webhook) and converts them into one consistent format. An AI model then reads each ticket and returns a category (billing, technical or general), an urgency level (high, medium or low) and a one-sentence reason. High-urgency tickets alert the team in Slack straight away. Every ticket is logged to Airtable with its final status, so the team has a complete record.

## How it works
**Intake:** the form and webhook payloads arrive in different shapes, so each is mapped to the same set of fields before anything else happens.
**Classify:** the AI returns a response that must match a strict JSON schema, with category and urgency limited to fixed allowed values. This keeps the output predictable enough to route on.
**Route:** a rule-based switch sends high-urgency tickets to Slack, and every ticket to Airtable.
**Safety net:** if the classification comes back as anything unexpected, the ticket is flagged "needs manual review" and logged instead of disappearing.

<img width="1189" height="208" alt="Slack alert for a high-urgency ticket" src="https://github.com/user-attachments/assets/7c0a9006-8df0-4040-b5ad-48fd66b175ff" />

*A high-urgency ticket triggers an instant alert to the team's Slack channel, with the category and urgency.*

<img width="1382" height="359" alt="Airtable log of classified support tickets" src="https://github.com/user-attachments/assets/c1a8c6d5-3803-4db7-86d0-7ac597eb9217" />

*Every ticket is logged with its category, urgency, reasoning and final status.*

## Design decisions
- **Urgency reflects business impact, not just tone.** The classifier is instructed to rate urgency by business impact and time sensitivity. In testing, an angry complaint about something cosmetic (a font that was too small) was rated medium rather than high.
- **Branch around the AI step, then recombine.** The AI node returns only its own output and drops the original ticket fields (name, email, description). Rather than hoping a setting would preserve them, I branch the original data around the AI node and combine it back afterwards, so nothing is lost.
- **Rules decide the routing, AI only classifies.** The switch that routes tickets is plain logic, so a given classification always produces the same routing.
- **Log once, with the final status.** Tickets are written to Airtable after routing, so each row has its correct final status and no row needs correcting later.

## Results
I tested the workflow on 9 tickets I wrote by hand, covering all three urgency levels, all three categories and both intake channels.
- **Urgency:** 9 out of 9 matched the label I set in advance. Urgency drives the routing, so this is the number that matters most.
- **Category:** 8 out of 9. The one miss was a payment-gateway failure submitted under "Billing" and classified as "Technical". The ticket really was ambiguous, and the AI's reasoning was sensible.
- **Safety net:** I forced an unrecognised urgency value, and the ticket was correctly flagged for manual review.
- **Speed:** roughly 2 to 7 seconds per ticket.
- **Cost:** a full day of testing, covering more than ten tickets, cost under one cent in OpenAI usage, so each ticket costs well under a tenth of a cent.
The test set is small and I wrote it myself, so I treat these figures as a sanity check, not a benchmark.

## Limitations and next steps
- **The AI classifies from the text of the ticket,** not from the category the customer picked. That is usually an advantage, but a next step is to store both and flag tickets where they disagree.
- **Urgency levels aren’t defined in detail in the prompt,** so ambiguous tickets rely on the model’s judgment. A next step is to add explicit definitions for each level and test on borderline cases.
- **There is no error-notification workflow yet,** so a failed run would not alert anyone automatically.
- **Customers do not yet receive an automatic acknowledgment.**
- **There is no duplicate-ticket detection.**

## Tools
n8n (self-hosted), OpenAI (gpt-4.1-mini, structured JSON output), Airtable, Slack.

## What's in this repo

- `README.md`: this case study

The workflow export is available on request.

## Portfolio 
https://mattreilly98.github.io
