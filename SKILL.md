---
name: mai-drugs
description: Manages medicine requests to a GP / family doctor. Sends the prescription or renewal request to the doctor (email, SMS or WhatsApp) and tracks stock by logging the boxes picked up. Use when the user says things like "I need to ask my doctor for a prescription", "write to the doctor for my meds", "renew my prescription", "I'm running out of pills", "I picked up my medicines", "the doctor gave me N boxes" — or in Italian "devo chiedere la ricetta al medico", "scrivi al dottore per le medicine", "rinnovo ricetta", "finisco le pillole", "ho ritirato le medicine". Works in any language.
---

# Doctor medicine requests

Writes **and sends** the message asking the doctor for a prescription, on the channel the user chose (email, SMS or WhatsApp), and keeps track of how long the medicines will last. For the user's own medicines only.

The skill is an agent: it **sends** the message itself to the doctor's address or number saved in the profile, using whatever sending tool is available for that channel (a mail connector for email, a messaging tool for SMS / WhatsApp). It does not hand the text back for the user to send.

Be honest about the outcome:

- Say "sent" only after the sending tool confirmed it. Then report in one line: channel, recipient, date and time.
- If no tool is available for the saved channel, or the send fails, **do not claim it was sent**. Say what is missing or what failed, show the message and the recipient so the user can send it by hand, and log nothing as sent.

## How the user talks to the skill

Plain chat, no commands. Examples:

- "Devo chiedere la ricetta" / "I need to renew my prescription" → **Request** mode
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
9. **Sending**: send automatically when the user asks for the request (default), or show the message and ask for confirmation first.

Save the answers, then continue with the request. Never leave an unanswered item silently out of the message: if something is missing, ask.

## Request mode

1. Work out which medicines are needed (even just "the usual ones") and whether it is a renewal or a first request.
2. If a required datum is missing (medicine name, dosage), ask. Never invent names, dosages or quantities.
3. From `dati/scorte.md`, compute the run-out date: units left ÷ units per day. If the user gives a date, use it.
4. Check the advance: the message must go out **at least the configured number of working days before** the stock runs out (weekends and holidays the user mentions do not count). Give the latest date to send it. If the doctor's turnaround leaves no margin, say so and suggest an earlier date. If the window has already passed, mark the message urgent and tell the user in one line.
5. Write the message in the saved channel's format (see below). **Do not state the number of boxes**: the doctor decides.
6. Check the "Requests sent" table in `dati/scorte.md`: if a request for the same medicines was already sent and no pickup has been logged since, tell the user and ask before sending another.
7. Send it to the saved email / number on the saved channel. If the profile says "confirm first", show the message and wait for a yes before sending.
8. Reply in one line with the outcome (see above) and, if the advance was exceeded or data was missing, one extra line about it. Append a row to the "Requests sent" table in `dati/scorte.md`.

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
3. Reply in one line with the new run-out date and the latest date for the next request.

## Rules

- Concise, direct, no filler.
- Never send to an address or number other than the one saved in the profile.
- One request per message; several medicines go in a single list.
- No medical advice: never suggest medicines or dosage changes. Report any new symptoms or side effects the user mentions to the doctor as stated, without interpreting them.
- Posologia / units per day are used only to compute stock, never written in the message.
