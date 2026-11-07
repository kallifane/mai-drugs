# mai-drugs

Skill `richiesta-medico`: gestisce le richieste di farmaci al medico di base. Scrive l'email per ricette e rinnovi e tiene traccia delle scorte dai ritiri.

Il repo è la skill: `SKILL.md` sta nella radice.

## Uso

Si parla in chat, senza comandi:

- "Devo chiedere la ricetta" → bozza dell'email al medico (non invia nulla)
- "Ho ritirato 2 scatole di X" → aggiorna le scorte e ricalcola la fine scorta
- "Quando finiscono le medicine?" → risposta in una riga

Alla prima volta la skill chiede lingua, tono, medico e farmaci. Profilo e scorte vengono salvati in `dati/`, che git ignora: i dati personali non finiscono nel repo.

## Backlog

- [ ] Installazione da Hermes: verificare formato skill e struttura di repo attesi, e che il percorso `dati/` funzioni anche lì.
- [ ] Struttura plugin (`plugin.json`, catalogo) se si vuole il marketplace.
- [ ] Nome e autore da definire.
- [ ] Licenza.
