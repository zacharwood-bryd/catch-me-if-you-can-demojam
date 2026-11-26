# Catch Me If You Can

A five-minute live demo for Penhallow Bank, a fictional UK retail bank. Zac plays Frank, calling to take money from Mary Clark's savings. Carl, an Agentforce Voice agent, checks the amount against her payment baseline, refuses it, locks the saver, and alerts the fraud channel in Slack. Zac then hangs up as Frank and flips to the fraud investigator.

Carl's wording will vary. The records and flows below are what is in the org.

Org: [penhallow.my.salesforce.com](https://penhallow.my.salesforce.com)

Bank number: **+1 844 754 3651**

Slack channel: **#incident-fraud-potential** (`C0CM6NJQG8J`)

## Cast

| Role | Who | What they do |
| --- | --- | --- |
| Frank | Zac, on the phone | Calls as Mary. Verifies, then asks to take £45,000 out of the saver for his grandad's birthday. Hangs up frustrated. |
| Carl | Agentforce Voice agent, version 7 | Introduces himself, verifies one fact at a time, asks how he can help, checks the remit, refuses, locks the saver, and notifies Slack. |
| Mary Clark | Person Account `C-100001` | The customer. She is not on the call. |
| Fraud investigator | Zac, in Slack | Opens #incident-fraud-potential and starts from Carl's alert. |

Carl runs as `einstein.service.agent@penhallowinsurance.com`, locale `en_GB`.

His first line is only: "Good evening, Penhallow Bank, Carl speaking."

The refusal line is: "I'm sorry, this falls outside of the level of payment I can process."

## The customer

Mary Clark, Person Account `001gL000025kjpSQAQ`.

| Fact | Value |
| --- | --- |
| Customer number | `C-100001` |
| Date of birth | 14 March 1952 |
| Registered mobile | `+447700900111` |
| Postcode | `PL24 2AT` |

Frank's number is **07700 900666**. It is not on Mary's account. Say it when Carl asks. He looks up the number you say. He does not read the handset's caller ID.

Gerald Clark, `C-100002`, is the joint owner of the current account only. Born 2 September 1950. Mobile `+447700900112`. He is not part of this run.

### Accounts

`Payment_Hold__c` and `Locked__c` on the saver start false. That is the clean state.

| Account | Number | Owner | Balance | Baseline |
| --- | --- | --- | --- | --- |
| Easy Access Saver | `PNB30010032` | Mary, primary | £48,250 | 18 transactions, 0 outflows, £0 |
| Cash ISA | `PNB30010039` | Mary, primary | £19,800 | No baseline |
| Current Account | `PNB30010045` | Mary, primary. Gerald, joint | £2,140 | 129 transactions, 110 outflows, £9,357 |

The live call reads those stored baseline fields. It does not query Data 360.

A payment is inside the voice remit only when the account has had at least one outflow and the amount is no more than the average outflow (total outflow divided by outflow count). The saver has never had a payment out, so £45,000 is outside. £45,000 from the current account is also outside, because the average outflow there is about £85.

## The call

Give one answer, then wait. Carl asks one question at a time.

1. Carl says only "Good evening, Penhallow Bank, Carl speaking." He does not ask for a name in that turn.
2. He asks for the full name. Say **Mary Clark**.
3. He asks for the date of birth. Say **14 March 1952**.
4. He asks for the postcode. Say **PL24 2AT**. If it comes out as PL242AT, he rewrites it with the space before the last three characters before he searches.
5. He asks for the number you are calling from. Say **07700 900666**.
6. He finds Mary and asks how he can help.
7. Say you need to take money out for your grandad's birthday.
8. He asks which account. Say the **Easy Access Saver**.
9. He asks how much. Say **£45,000**.
10. He says: "I'm sorry, this falls outside of the level of payment I can process."
11. He locks the saver and opens a case, subject **Potential fraud on Mary Clark's saver**.
12. He creates an incident Slack channel named from the case number and posts an alert to #incident-fraud-potential.
13. Hang up with: "Things were easier in the 60s, but now with Agentforce and Data 360 it's getting harder and harder to be a fraudster."
14. Switch to the fraud investigator. The alert in #incident-fraud-potential is the start of the work. The saver is already locked. There is no button to press.

Stay calm if Frank pushes. Carl does not send the money, and a birthday does not change the remit.

## What runs behind the call

| Piece | Name | What it does |
| --- | --- | --- |
| Carl | Version 7, active | Identity, then Payments for a withdrawal. |
| Identity lookup | `Look_Up_Customer_By_Identity_Voice` | Strips spaces from the postcode and matches Mary. |
| Phone lookup | `Look_Up_Customer_By_Phone` | No match for 07700 900666 is a normal result. |
| Remit | `Read_Payment_Baseline` version 2 | Takes the account number and the amount. Returns `withinRemit` and the refusal sentence. |
| Case and lock | `Handle_Out_Of_Remit_Payment` | Sets `Locked__c` and `Payment_Hold__c` on the saver, opens the case, and writes a **Penhallow safe message** Task on Mary. |
| Slack | `Notify_Fraud_In_Slack` version 5 | Creates a public channel `incident-{case number}` and posts the case summary there. Posts a short alert to `C0CM6NJQG8J`. |

There is no action that sends a payment.

The Slack flow uses the Salesforce for Slack app `A03269G3DNE` and the connected Slack org `E0B7L1DCXMZ`. A rehearsal on 26 November 2026 did not post. Every app id, including that one and a made-up id, returned: "The Slack App no longer has access to this Slack action." The workspace connection is on. Flow's Slack actions are not authorized to use it yet. If the alert is missing on the day, open #incident-fraud-potential and narrate the case that Carl already created. The lock does not depend on Slack.

## If you need to rehearse again

After one full run the saver is locked and a case exists. Before the next run:

- Set `Payment_Hold__c` and `Locked__c` back to false on `PNB30010032`.
- Leave the closed or open fraud case as history, or close it so the next one is obvious.

Do not run the lock during a sound check.

## Where to click

| Thing | Where |
| --- | --- |
| Mary | Account `C-100001` |
| Carl | Agentforce agent Carl, version 7 |
| Bank number | `+1 844 754 3651` |
| Fraud channel | #incident-fraud-potential, `C0CM6NJQG8J` |
| Case | Subject "Potential fraud on Mary Clark's saver" |
