---
name: richiesta-medico
description: Manages medicine requests to a GP / family doctor. Drafts the prescription or renewal email and tracks stock by logging the boxes picked up. Use when the user says things like "I need to ask my doctor for a prescription", "write to the doctor for my meds", "renew my prescription", "I'm running out of pills", "I picked up my medicines", "the doctor gave me N boxes" — or in Italian "devo chiedere la ricetta al medico", "scrivi al dottore per le medicine", "rinnovo ricetta", "finisco le pillole", "ho ritirato le medicine". Works in any language.
---

# Doctor medicine requests

Prepares the email asking the doctor for a prescription, and keeps track of how long the medicines will last. For the user's own medicines only.

The skill **drafts** the email in chat and never sends anything: the user sends it.

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

If `dati/profilo.md` does not exist, collect the profile fields with `AskUserQuestion` (language and tone, doctor's name and form of address, regular medicines with dosage / units per day / units per box, preferred way to receive the prescription). Defaults: lead time 2 working days, formal tone, the user's own language. Save the answers, then continue with the request.

## Request mode

1. Work out which medicines are needed (even just "the usual ones") and whether it is a renewal or a first request.
2. If a required datum is missing (medicine name, dosage), ask. Never invent names, dosages or quantities.
3. From `dati/scorte.md`, compute the run-out date: units left ÷ units per day. If the user gives a date, use it.
4. Check the lead time: the request must go out **at least the configured number of working days (default 2) before** the stock runs out. Give the latest date to send the email. If that window is already passed, mark it urgent in subject and body and tell the user in one line.
5. Write the email. **Do not state the number of boxes**: the doctor decides.
6. Deliver only the email. Add a single extra line only if data is missing or the lead time has been exceeded.

### Email structure

- **Subject**: short and clear, e.g. `Prescription request – [Full name]` / `Richiesta ricetta – [Nome Cognome]`
- **Greeting**: in the profile's language and tone.
- **Body** (3–5 lines):
  - who you are and what you ask (renewal / new prescription);
  - bulleted list of medicines with **name and dosage only** (the doctor already knows the schedule);
  - when the stock runs out;
  - how to receive the prescription, if set in the profile.
- **Closing**: thanks and full name.

Example (Italian, formal):

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
- One request per email; several medicines go in a single list.
- No medical advice: never suggest medicines or dosage changes. Report any new symptoms or side effects the user mentions to the doctor as stated, without interpreting them.
- Posologia / units per day are used only to compute stock, never written in the email.
