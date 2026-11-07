---
name: mai-drugs
description: Manages medicine requests to a GP / family doctor. Sends the prescription or renewal request to the doctor (email, SMS or WhatsApp) at the right time before the stock runs out, and tracks stock by logging the boxes picked up. Use when the user says things like "I need to ask my doctor for a prescription", "write to the doctor for my meds", "renew my prescription", "I'm running out of pills", "I picked up my medicines", "the doctor gave me N boxes" — or in Italian "devo chiedere la ricetta al medico", "scrivi al dottore per le medicine", "rinnovo ricetta", "finisco le pillole", "ho ritirato le medicine". Works in any language.
---

# Doctor medicine requests

The user tells the skill how much stock is left. The skill works out when the stock ends and **sends the request to the doctor by itself, the configured number of working days before that date**, on the channel the user chose (email, SMS or WhatsApp). For the user's own medicines only.

So the message does not go out when the user talks to the skill, but on the **send date**: run-out date minus the advance (working days). Sending right away is only for when the send date is today or already past.

The skill is an agent: it **sends** the message itself to the doctor's address or number saved in the profile, using whatever sending tool is available for that channel (a mail connector for email, a messaging tool for SMS / WhatsApp). It does not hand the text back for the user to send.

Be honest about the outcome:

- Say "sent" only after the sending tool confirmed it. Then report in one line: channel, recipient, date and time.
- If no tool is available for the saved channel, or the send fails, **do not claim it was sent**. Say what is missing or what failed, show the message and the recipient so the user can send it by hand, and log nothing as sent.

## How the user talks to the skill

Plain chat, no commands. Examples:

- "Mi restano 12 compresse di X" / "I have 12 pills of X left" → **Request** mode: compute the run-out and the send date, schedule the send
- "Devo chiedere la ricetta adesso" / "Send the request now" → **Request** mode, sent immediately
- "Ho ritirato 2 scatole di X" / "I picked up 2 boxes of X" → **Pickup** mode
- "Quando finiscono le medicine?" / "When do I run out?" → read the stock and answer in one line

## Data files

Personal data is never stored in this skill's shared files. It lives in the `dati/` folder next to this file (git-ignored, so it is not published with the skill):

- `dati/profilo.md` — created on first use from `profilo.example.md`
- `dati/scorte.md` — created on first use from `scorte.example.md`

If `dati/` or a file in it is missing, create it from the example. Read both files on every use. Never delete history rows: append, don't overwrite.

### First use (onboarding)

If `dati/profilo.md` does not exist, ask the user these questions (with `AskUserQuestion` when available, otherwise in chat), then save the answers:

1. Language and tone of the message (default: the user's own language, formal).
2. Doctor's name and form of address.
3. **Channel**: email, SMS or WhatsApp.
4. **Where to send it**, depending on the channel: the doctor's email address, or the phone number for SMS / WhatsApp. Some chat clients open a menu when the user types `@`, which makes an email address hard to type. Tell the user they can write `chiocciola` or `at` (or `[at]`) in place of `@`, or paste the address. Convert it to `@`, then repeat the full address back and ask the user to confirm it before saving.
5. **Doctor's turnaround**: how many working days the doctor needs to issue the prescription (default 2).
6. **Advance**: how many working days before the stock runs out the user wants to send the request. Default 2. It must be larger than the turnaround, otherwise the prescription is ready the very day the stock ends with no margin: if the answer is not larger, say so and suggest turnaround + 1.
7. How the user wants to receive the prescription (email, SMS, pick-up at the office).
8. Regular medicines: name, dosage, units per day, units per box.
9. **Send time** of day (default 09:00).
10. **Sending**: send automatically on the send date (default), or show the message and ask for confirmation first.

Save the answers, then continue with the request. Never leave an unanswered item silently out of the message: if something is missing, ask.

## Request mode

1. Work out which medicines are involved and how much stock is left of each (the user gives it, e.g. "12 pills left", or it comes from `dati/scorte.md`). Save it in `dati/scorte.md`.
2. If a required datum is missing (medicine name, dosage, units left), ask. Never invent names, dosages or quantities.
3. Compute the run-out date: units left ÷ units per day, counted from today.
4. Compute the **send date**: run-out date minus the advance from the profile, counting working days (weekends and holidays the user mentions do not count). The send time is the one in the profile (default 09:00). If the doctor's turnaround leaves no margin, say so and suggest an earlier send date.
5. Check the "Requests sent" table in `dati/scorte.md`: if a request for the same medicines is already sent or scheduled and no pickup has been logged since, tell the user and ask before creating another.
6. If the send date is in the future: **schedule** the send for that date and time with the scheduling tool the agent has (cron, scheduler, one-off job or reminder with an action). The scheduled job must carry the full message, the channel and the recipient, so it can send without further input. Record it in "Scheduled sends" in `dati/scorte.md` (send date and time, job id, channel, recipient, medicines, run-out date).
7. If the send date is today or already passed, send now, marked urgent, and say so in one line.
8. If the profile says "confirm first", show the message and ask for a yes at the moment of sending, not only at scheduling.
9. Reply in one line: "Scheduled for [date, time] on [channel] to [recipient], stock runs out [date]". Add a line only if data was missing or the advance leaves no margin.

If no scheduling tool is available, **do not pretend**: say that the send cannot be scheduled here, give the send date, and offer to send now or to remind the user on that date if a reminder tool exists.

When a scheduled send fires, send the message with the available tool and report as described above: "sent" only after the tool confirmed. Move the row from "Scheduled sends" to "Requests sent" with the real date and time. If the send fails, tell the user at once.

### Email structure (channel: email)

- **Subject**: short and clear, e.g. `Prescription request – [Full name]` / `Richiesta ricetta – [Nome Cognome]`
- **Greeting**: in the profile's language and tone.
- **Body** (3–5 lines):
  - who you are and what you ask (renewal / new prescription);
  - bulleted list of medicines with **name and dosage only** (the doctor already knows the schedule);
  - when the stock runs out;
  - how to receive the prescription, if set in the profile.
- **Closing**: thanks and full name.

### SMS / WhatsApp structure

No subject, two or three lines: greeting, request with the medicines (name and dosage only), run-out date, thanks and full name. Example: "Buongiorno Dottore, sono Mario Rossi: le chiedo il rinnovo della ricetta per [Farmaco A] [dosaggio]. Le scorte finiscono il [data]. Grazie, Mario Rossi".

Example email (Italian, formal):

> **Oggetto**: Richiesta rinnovo ricetta – Mario Rossi
>
> Buongiorno Dottoressa Bianchi,
>
> le scrivo per chiederle il rinnovo della ricetta per i seguenti farmaci:
>
> - [Farmaco A] [dosaggio]
> - [Farmaco B] [dosaggio]
>
> Le scorte terminano indicativamente il [data]. Mi sarebbe gradito ricevere il numero di ricetta elettronica via email.
>
> La ringrazio per la disponibilità.
> Cordiali saluti,
> Mario Rossi

## Pickup mode

When the user says they collected the medicines:

1. Ask (if not stated) how many boxes of each medicine the doctor gave and the pickup date.
2. Update `dati/scorte.md`: add boxes × units per box to the units left, and append a row to the pickup history.
3. Recompute the run-out date and the send date. Cancel any scheduled send still pending for those medicines (they are no longer needed) and schedule the new one as in Request mode.
4. Reply in one line with the new run-out date and the new send date.

## Rules

- Concise, direct, no filler.
- Never send to an address or number other than the one saved in the profile.
- One request per message; several medicines go in a single list.
- No medical advice: never suggest medicines or dosage changes. Report any new symptoms or side effects the user mentions to the doctor as stated, without interpreting them.
- Posologia / units per day are used only to compute stock, never written in the message.
