# Contribuire

Grazie per l'interesse verso questo stack. Questo repository è nato come **materiale di studio personale** su un'architettura LLM locale open source, ma i contributi che lo migliorano sono benvenuti.

## Come contribuire

1. **Apri una issue** prima di lavori consistenti, così ne discutiamo insieme.
2. **Forka** il repository e crea un branch descrittivo (`fix/vram-note`, `docs/troubleshooting-cuda`).
3. **Fai le tue modifiche** seguendo le convenzioni sotto.
4. **Apri una Pull Request** con una descrizione chiara di cosa cambia e perché.

## Convenzioni

### Documentazione
- Mantieni lo **stile a livelli** (L1–L4) coerente con l'architettura esistente.
- Usa **etichette epistemiche** quando affermi qualcosa non verificato:
  - `[Inferenza]` — deduzione plausibile
  - `[Non verificato]` — da confermare alla fonte
- I dati tecnici (VRAM, requisiti, versioni) vanno **verificati alla fonte ufficiale** e datati, perché cambiano rapidamente.

### Comandi e configurazioni
- Testa i comandi prima di proporli.
- Segnala esplicitamente le differenze **Windows / Linux / macOS** dove rilevanti.
- Non includere **mai** chiavi, token o percorsi personali negli esempi: usa placeholder.

### Sicurezza e compliance
- Nessun contributo deve introdurre invii di dati verso cloud esterni senza dichiararlo chiaramente.
- Le modifiche che toccano la sezione agente devono preservare il principio `mode = "interactive"` come default sicuro.

## Cosa NON committare

Rispetta il `.gitignore`: mai modelli, chiavi, vector DB, o documenti riservati. In caso di dubbio, chiedi nella issue.

## Licenza dei contributi

Contribuendo, accetti che il tuo contributo sia rilasciato sotto la stessa licenza **MIT** del repository.
