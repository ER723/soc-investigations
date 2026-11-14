# SOC Investigations

Tier 1 SOC practice work by Anggasukmana Adisastra: email, alert and log investigations written up as short reports, the way a ticket would be handed to the next analyst.

## Reports

| ID | Report | Type | Verdict |
|---|---|---|---|
| Investigation 01 | [Phishing email posing as PayPal](reports/investigation-01.md) | Email header and body analysis (LetsDefend practice case) | Phishing, high confidence |
| Alert triage 01 | [Repeated failed sudo attempts](reports/alert-triage-01.md) | Alert triage in my own OSSEC lab | Benign (a test I ran); found and fixed a detection gap |

## What each report contains

- A one-paragraph summary and a verdict
- What was examined, and a timeline
- Evidence, with indicators of compromise defanged (for example `example[.]com`, `hxxps://`)
- Recommended actions, and what an analyst would escalate
- **What I did not do**, so the limits of the work are clear

## Scope and honesty notes

- Investigation 01 is a practice sample from a training exercise (an old email), not a live incident. I did not open its link or landing page.
- Alert triage 01 was done on a lab VM I own, using failed logins I triggered myself.
- Where I inferred something instead of confirming it, the report says so.

## Tools used

OSSEC HIDS and Kali Linux (lab), Mozilla Thunderbird and raw email headers, `grep` and `auth.log`.
