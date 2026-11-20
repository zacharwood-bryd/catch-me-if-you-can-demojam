# Catch Me If You Can

A five-minute, one-person live demo for Penhallow Bank, a fictional UK retail bank. Zac plays Frank, a con man calling on Christmas Eve to empty Mary Clark's savings. Carl, an Agentforce Voice agent, stops him. Zac then switches to the fraud analyst in Slack and locks the account.

The dialogue below is the intended behaviour. Carl's wording will vary live. The records, flows, and Slack card described here are what is actually in the org and the Penhallow Bank workspace.

Org: [penhallow.my.salesforce.com](https://penhallow.my.salesforce.com)

Bank number: **+1 844 754 3651** (US toll-free)

Slack: **Penhallow Bank** workspace, channel **#penhallow-fraud**

## Cast

| Role | Who | What they do in the demo |
| --- | --- | --- |
| Frank | Zac, live on the phone | Calls as Mary. Gives her name, date of birth, and postcode. Asks to move the mobile onto the calling number, then to pay £45,000 to a new payee. |
| Carl | Agentforce Voice agent, active version 6 | Verifies the caller, refuses the number change and the payment, sends a code only to the mobile already on file, places a hold, and opens one fraud case. |
| Mary Clark | Seeded Person Account `C-100001` | The real customer. She never appears on the call. Her phone is represented by Tasks on her Account. |
| Gerald Clark | Seeded Person Account `C-100002` | Optional callback. Joint owner of the current account only. |
| Fraud analyst | Zac, in Slack | Reads the pre-posted card and clicks **Lock account and contact customer**. |

Carl runs as `einstein.service.agent@penhallowinsurance.com`, locale `en_GB`. His welcome line is: "Good evening, Penhallow Bank, Carl speaking. How can I help you?"

## The customer Carl is protecting

Mary Clark, Person Account `001gL000025kjpSQAQ`.

| Fact | Value |
| --- | --- |
| Customer number | `C-100001` |
| Date of birth | 14 March 1952 |
| Registered mobile | `+447700900111` |
| Postcode used for the name check | `PL24 2AT` (Par). This is the person mailing postcode. Billing postcode is empty. |

Frank's calling number in the story is **07700 900666** (`+447700900666`). It matches no Account phone. When Carl asks for the number, say **07700 900666**. Carl looks up the number you say. His identity step does not read the handset's real caller ID.

The other people already in the org, if a question comes up:

| Person | Customer number | Mobile | Why they exist |
| --- | --- | --- | --- |
| Gerald Clark | `C-100002` | `+447700900112` | Joint owner of the current account only. Born 2 September 1950. |
| Daniel Hosking | `C-100045` | `+447700900245` | The story says Frank's number once belonged to him. It is not his number now, and a phone lookup of `07700 900666` finds nobody. |
| Joan Rowe | `C-100105` | `+447700900970` | The customer Frank claimed to be on 28 October. |

### Accounts

Custom objects, built to look like Financial Services Cloud. `Status__c` is `Open` on all three. The lock flow does not change status. It sets the two flags.

| Account | Number | Salesforce id | Owner | Balance | Baseline stored on the account |
| --- | --- | --- | --- | --- | --- |
| Easy Access Saver 0032 | `PNB30010032` | `a4ogL000000VgxRQAS` | Mary, Primary Owner | £48,250 | 18 transactions, 0 outflows, £0 total outflow |
| Cash ISA 0039 | `PNB30010039` | `a4ogL000000VgxSQAS` | Mary, Primary Owner | £19,800 | No baseline stored |
| Current Account 0045 | `PNB30010045` | `a4ogL000000VgxTQAS` | Mary, Primary Owner. Gerald, Joint Owner | £2,140 | 129 transactions, 110 outflows, £9,357 total outflow |

`Payment_Hold__c` and `Locked__c` on the saver are both false. That is the clean state for a rehearsal.

The saver baseline is what Carl reads during the call. `Read_Payment_Baseline` reads `Baseline_Transaction_Count__c`, `Baseline_Outflow_Count__c`, and `Baseline_Total_Outflow__c` on `Financial_Account__c`. Those numbers came from the Account Payment Baseline insight. The live call does not query BigQuery.

### Payees already on Mary

All four were added at a branch. A payment to a grandson is a new payee.

| Payee | Id | Sort code | Last 4 | Added |
| --- | --- | --- | --- | --- |
| British Gas | `PY-00001` | 66-32-49 | 6648 | 28 Dec 2018 |
| South West Water | `PY-00002` | 86-23-97 | 9771 | 4 Aug 2012 |
| Cornwall Council Tax | `PY-00003` | 58-51-73 | 1007 | 24 Jul 2018 |
| TV Licensing | `PY-00004` | 37-27-84 | 0003 | 8 Oct 2012 |

### The prior call

VoiceCall `VC-00000001` (`0LQgL000004ErmvWAC`).

| Field | Value |
| --- | --- |
| Vendor call key | `CALL-000256` |
| When | 28 October 2026, 18:40 UTC |
| From | `+447700900666` |
| To | `+18447543651` |
| Type | Inbound |

Beat 9 quotes this record: the same number called on 28 October, claimed to be another customer (`C-100105`, Joan Rowe), and failed security twice. That sentence is written into the fraud case and the Slack card. Carl does not look the VoiceCall up mid-call.

## What is already running

Checked against the org on 18 November 2026. Saver unlocked, no hold, no fraud case on Mary, no phone-mock Tasks.

| Piece | Live name | What it does |
| --- | --- | --- |
| Voice channel | `Voice_Channel_18447543651`, type `PstnVoice`, key `+18447543651` | Inbound number. Routed to Carl through Agentforce Contact Center (Omni Flow), with the contact-centre queue as fallback. |
| Carl | Agentforce Service Agent, version 6 active (`Carl_6`) | Topics: Identity, Contact changes, Payments. He asks one question at a time. |
| Look up customer by phone | `Look_Up_Customer_By_Phone` | Searches Account phone fields. No match is a valid result. |
| Look up customer by identity | `Look_Up_Customer_By_Identity_Voice` | Matches name and date of birth, then the postcode with spaces ignored. Carl is told to write the postcode with the space, such as `PL24 2AT`. Returns whether the calling number is the registered mobile, and a sentence for him to say. |
| Read accounts and payees | `Read_Accounts_And_Payees` | Returns text summaries of accounts, ownership roles, and payees. |
| Read payment baseline | `Read_Payment_Baseline` | Reads the three baseline fields for one financial account number. |
| Send one-time code | `Send_One_Time_Code` | Writes a code Task on the Account. The only destination is the mobile already stored. |
| Place payment hold | `Place_Payment_Hold` | Sets `Payment_Hold__c` on the financial account. Does not send money and does not lock the account. |
| Create fraud case | `Create_Fraud_Case` | Opens one Case. Does not post to Slack. |
| Lock account | `Lock_Account_And_Contact_Customer` | Screen flow behind the Slack button. |

All eight flows are active, API version 68, system mode without sharing. There is no Apex.

There is no flow action that changes a mobile number, adds a payee, or sends a payment. Those refusals live in Carl's topic instructions.

## How Carl behaves

System instruction: he is calm and kind. He never changes a mobile number, never adds a payee, and never sends a payment. A code goes only to the mobile already on the account. If he cannot complete a request safely, he offers a branch visit with photo ID or a call back on the registered number. He asks one question at a time and waits for the answer.

### Identity

He greets the caller, then asks for one fact and stops. The order is name, then date of birth, then postcode, then the number they are calling from. If you have already given one, he skips it and asks for the next missing fact. He does not ask for all four in one turn.

A UK postcode always has a space before the last three characters. He rewrites `PL242AT` to `PL24 2AT` before he searches. The lookup also matches if the space is missing.

1. He stores the number you say.
2. He looks that number up. Finding nobody is a signal. He stays on the call and does not describe a failed search.
3. He matches name, date of birth, and postcode. For Mary that match sets the unregistered-number flag, because `07700 900666` is not `+447700900111`. He then says he has found the account, using her first name.
4. A request to change a phone goes to Contact changes. A request to add a payee or send money goes to Payments.

Give him one answer each time he asks:

- Name: Mary Clark
- Date of birth: 14 March 1952
- Postcode: `PL24 2AT`
- Number: `07700 900666`

### Contact changes

He cannot write a new number. If a code is needed, `Send_One_Time_Code` uses the mobile on the Account. He tells the caller the code went to the mobile ending **111**. The code itself is hidden from him, so he will not read it out.

If Frank says he no longer has that phone, Carl does not send the code anywhere else. He offers a branch visit with photo ID, or a call back on the registered number.

The first time he sends the code, or the first time he refuses the number change, he opens the fraud case once. He does not open a second case on the same call.

### Payments

He cannot add a payee and cannot send a payment. Before he answers he reads accounts, roles, and payees, then the baseline for the account Frank wants to pay from.

He places a hold when the payee is new, the baseline outflow count is zero, or the amount is nearly the whole balance. When the outflow count is zero he says that savings account has never had a payment out. He explains that the payment will not be sent.

If the fraud case is not already open, he opens it once before the hold.

Gerald is a joint owner of current account `PNB30010045` only. He is not an owner of Easy Access Saver `PNB30010032`. If the caller is Gerald, or says they are calling for his wife, Carl refuses a transfer from the saver. If the calling number is already on an open fraud case, he stops and passes the call to a colleague. The escalation line is: "I'll pass you to a colleague who can take this from here."

## The five-minute run

Twelve beats. The call is about 0:45 to 2:50. Slack is about 2:50 to 4:20. The close lands by 5:00.

Have these open before you dial, and leave Slack off the audience screen until beat 9:

1. Mary's Person Account, so the open can show the mobile, the three accounts, and the four payees.
2. The voice call transcript, once the call connects.
3. Penhallow Bank, `#penhallow-fraud`, ready to switch to.
4. Mary's Activity timeline, for the code Task and later the safe-message Task. The title card, queue, phone page, and closing slide are not built yet, so those beats are narration plus these Salesforce screens.

### 1. Cold open, 0:00 to 0:30

Say: "It's Christmas Eve. Penhallow Bank's phone lines are the busiest they'll be all year. Hold times are through the roof and the team are exhausted. And somewhere out there, someone's counting on exactly that."

Nothing live runs. The contact-centre queue slide is not built, so this beat is narration.

### 2. Set up the con, 0:30 to 0:45

Say: "Meet Mary. She's 74, banked with us for years, and has £48,000 in a savings account she has never once taken money out of. Tonight, I'm Frank, and I'm going to try to take it."

Show the Person Account: mobile ending 111, Easy Access Saver £48,250, Cash ISA £19,800, current account £2,140, and the four utility payees.

### 3. The call connects, 0:45 to 1:05

Dial **+1 844 754 3651** from the handset you will use on the day. Carl answers with the welcome line, then asks for the name. Wait. Then the date of birth. Wait. Then the postcode. Wait. Then the number. Give him one fact each time.

On screen: the live transcript.

Behind it:

- `Look_Up_Customer_By_Phone` on `07700 900666` returns no customer. That is signal 1.
- `Look_Up_Customer_By_Identity_Voice` matches `C-100001` and sets unregistered number to true, because the number you said is not the registered mobile.

### 4. Change the number, 1:05 to 1:30

Say: "I've got a new phone, so I need to update my mobile to this number."

Carl can help, and he does not make the change. Contact changes require a code to the registered mobile first. Signal 2 is a contact change from an unregistered number.

He may open the fraud case here, or at the next step. Either way he opens it once.

### 5. Move the money, 1:30 to 1:55

Say: "And while I've got you, I need to move £45,000 from my savings to my grandson's account before Christmas."

Carl should say, in his own words, that the savings account has never had a payment out, so he is taking extra care.

Behind it:

- `Read_Accounts_And_Payees` shows the three accounts, Mary as primary owner, Gerald only on the current account, and the four branch payees. A grandson is not among them. Signal 3.
- `Read_Payment_Baseline` for `PNB30010032` returns 18 transactions, 0 outflows, £0. Signal 4 is a first-ever outflow of nearly the whole £48,250 balance.

### 6. Step-up, 1:55 to 2:20

Say: "Sorry, which phone?" Then: "I told you, I don't have that phone any more."

Carl says the code has gone to the mobile ending 111. He does not offer to send it to `07700 900666`. He offers a branch visit with photo ID, or a call back on the registered number.

`Send_One_Time_Code` does this:

- Generates a six-digit code from the current time, hours, minutes, and seconds.
- Writes a completed Task on Mary, subject **Penhallow verification code**.
- Task description: `Penhallow Bank code <code>. Sent to the mobile ending 111. Didn't request this?`
- Returns the code to the flow. Carl's action hides the code and the message from him.

Show that Task when you want Mary's phone on screen. No SMS is sent. The Task is the phone.

`Create_Fraud_Case` opens the case if it is not already open. See the case section below. The Slack card is already waiting. This flow does not post it.

### 7. Pressure, 2:20 to 2:45

Say: "This is ridiculous, it's Christmas Eve, I've banked here for forty years, I want to make a complaint."

Carl stays calm. He still will not change the number or send the payment. He repeats the branch visit or the call back.

`Place_Payment_Hold` sets `Payment_Hold__c` on `PNB30010032`. `Locked__c` stays false until you click the Slack button. `Status__c` stays `Open`.

### 8. Frank hangs up, 2:45 to 2:50

Say: "And... Frank's gone." End the call.

The case already exists, so nothing in Slack is waiting for the call to finish. The voice transcript stays on the VoiceCall Salesforce creates for this live call. That live VoiceCall is separate from `VC-00000001`.

### 9. Switch to Slack, 2:50 to 3:30

Say: "Now I'm the fraud analyst. Here's what Carl sent me."

Open **Penhallow Bank**, **#penhallow-fraud**. The card is already there. It was posted before the demo, because the Slack token is not stored in Salesforce and the case flow cannot post.

The two cards already in the channels were posted under the previous customer name. Change the heading on both to **Suspected fraud on Mary Clark's saver** before the demo. A case Carl opens from here already uses that subject, and the lock flow closes a case with that subject.

The card says:

> **Suspected fraud on Mary Clark's saver**
>
> Frank, calling from 07700 900666, asked to move the mobile number and to pay almost the whole Easy Access Saver to a new payee.
>
> **Signals**
> 1. The calling number is not the registered mobile ending 111.
> 2. The payee is not already on the account.
> 3. Easy Access Saver PNB30010032 has never had a payment out. Outflow count 0. Balance £48,250.
> 4. The amount is nearly the whole balance.
>
> **Prior call:** 28 Oct, CALL-000256, VoiceCall VC-00000001.
>
> The payment is on hold and will not be sent.

Under it, a red button: **Lock account and contact customer**.

The same words are the Case description, plus one extra line with the lock link. The case is:

| Field | Value |
| --- | --- |
| Subject | Suspected fraud on Mary Clark's saver |
| Account | Mary Clark |
| Origin | Phone |
| Priority | High |
| Status | New, until the lock flow closes it |
| Description | The summary above, then `Lock the saver:` and the flow link |

The case text is fixed. It does not insert a live transcript snippet, and it does not change if Carl's wording on the call is different. The four signals and the 28 October call are always the ones above.

`#penhallow-fraud` and `#penhallow-fraud-fallback` exist only in the Penhallow Bank workspace. They are ordinary local channels. They do not appear in Norrhem Livs, Northbridge University, Support, or Meridian Group.

If the main card is missing, switch to **#penhallow-fraud-fallback**. It has the same card, with a context line: "Cued fallback. Use this if the live post does not arrive."

### 10. The human decision, 3:30 to 3:50

Say: "I want to lock this down and make sure Mary knows."

Click **Lock account and contact customer**. You need to be logged into the Penhallow org. The button is a link, not a Slack action:

`https://penhallow.lightning.force.com/flow/Lock_Account_And_Contact_Customer?financialAccountNumber=PNB30010032`

Opening that link runs the flow before the screen appears. The input defaults to `PNB30010032` even without the query string.

The flow then:

1. Finds `Financial_Account__c` where `Account_Number__c` is `PNB30010032`.
2. Sets `Locked__c` and `Payment_Hold__c` to true. Status stays `Open`.
3. Creates a completed Task on Mary, subject **Penhallow safe message**, description: "You don't need to do anything. We've locked your Easy Access Saver and stopped the payment. If you didn't ask for this, you're safe."
4. Finds the newest open Case on Mary with subject "Suspected fraud on Mary Clark's saver" and sets Status to **Closed**.
5. Shows a screen titled **Account locked**:
   - The Easy Access Saver is locked and the payment hold stays on.
   - Mary's phone, with the safe message.
   - Carl: "I've stopped that payment. You don't need to do anything else on this call. A colleague is looking after Mary."

The Slack message itself does not change when you click. Say that the lock is done, and show the flow screen or the two Tasks.

There is no Case status called "Fraud prevented". Closed is the status this org uses.

Salesforce for Slack is not connected, so the button cannot be a native Slack action yet. Connecting it is a manual step in the Penhallow Bank workspace if you want that later. The link already runs the flow.

### 11. Mary hears from the bank, 3:50 to 4:20

Say: "And Mary hears from us before Frank can try again."

Show the **Penhallow safe message** Task, or the flow screen. That is her phone for this build. Read the safe message, then Carl's line.

### 12. Close, 4:20 to 5:00

Say: "Carl was just as careful on call 10,000 on Christmas Eve as on call one. He knew Mary had never moved a penny from that account, straight from the bank's core data without copying it. Agentforce Voice, Data 360 and Slack, working as one. Frank didn't get away this time."

The closing slide is not built. The line about the data is the story of the baseline: 18 transactions and 0 outflows, stored on the saver from the Account Payment Baseline insight, and read by Carl through the flow.

## Optional beat: Frank calls back as Gerald

Adds about 45 seconds. If you include it, cut beat 11 to one line. Place it after beat 9, while the fraud case is still open, and before you click the lock button.

Call again from the same handset. Say you are Gerald Clark, calling for your wife, and ask to move the savings.

Carl refuses. Gerald is joint owner of `PNB30010045` only. He is not an owner of the saver. Because the calling number is already on an open fraud case, Carl escalates: "I'll pass you to a colleague who can take this from here."

A second Slack alert under the same case is not built. The escalation is on the call. The card already in `#penhallow-fraud` is the analyst's view.

## If something goes wrong on the day

| What happens | What you do |
| --- | --- |
| The call fails to connect, or the audio drops | Narrate beats 3 to 8 from this script, then carry on live in `#penhallow-fraud`. A recorded fallback of a clean call is not in the repo yet. |
| Carl wanders | Stay in character and ask the next scripted thing. The number change, then the £45,000, then "I don't have that phone." The guardrails matter more than his exact words. |
| The Slack card is not on screen | Switch to `#penhallow-fraud-fallback`. |
| You need to show the code and the Task is hard to find | On Mary's Account, open Activity and the completed Task **Penhallow verification code**. |
| Screen-share audio is unreliable | Join the video platform from a second device and dial from that handset. |

After one full click of the lock button the saver is locked, the hold is on, and the case is closed. A second rehearsal needs those cleared by hand until a reset flow exists:

- `Payment_Hold__c` and `Locked__c` back to false on `PNB30010032`
- Delete or ignore the closed fraud case, and the two Tasks **Penhallow verification code** and **Penhallow safe message**

Do not click the button during a sound check.

## What this build does not include yet

- Title card, contact-centre queue, a phone web page, and the closing slide. The phone today is the Task on Mary's Account, and the lock screen.
- A live Slack post from `Create_Fraud_Case`. The card is pre-posted.
- A Slack button that calls Salesforce inside Slack. The button opens the flow URL.
- A rehearsal reset flow.
- Mapping your real handset's caller ID onto Frank. Say `07700 900666` when Carl asks. `07700 900666` is an Ofcom drama number and is not spoofed onto the handset.
- A live Data 360 query during the call. Carl reads the baseline fields on the financial account.
- A transcript snippet inside the case. The case and the Slack card use the fixed summary.

## Where to click

| Thing | Where |
| --- | --- |
| Mary | Account `C-100001` in the Penhallow org |
| Prior call | VoiceCall `VC-00000001` |
| Carl | Agentforce agent Carl, version 6 |
| Bank number | `+1 844 754 3651` |
| Fraud card | Penhallow Bank workspace, `#penhallow-fraud` |
| Cued copy | `#penhallow-fraud-fallback` in the same workspace |
| Lock | The button on the card, or the Lock Account And Contact Customer flow with `financialAccountNumber` `PNB30010032` |
