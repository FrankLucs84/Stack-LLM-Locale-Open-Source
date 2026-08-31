# 🧠 Stack LLM Locale Open Source — RAG + Agente

> Un sistema completo per eseguire un **RAG di qualità in locale** e, all'occorrenza, un **agente operativo** — tutto **open source, gratuito e local-first**. Nessun dato verso cloud esterni.

![License](https://img.shields.io/badge/license-MIT%20components-blue)
![Local First](https://img.shields.io/badge/local--first-yes-success)
![Hardware](https://img.shields.io/badge/target-RTX%204060%208GB%20%C2%B7%2032GB%20RAM-orange)

---

## 📑 Indice

- [Cos'è questo stack](#-cosè-questo-stack)
- [Architettura a livelli](#-architettura-a-livelli)
- [La via singola raccomandata](#-la-via-singola-raccomandata)
- [Requisiti hardware](#-requisiti-hardware)
- [Guida di installazione](#-guida-di-installazione)
  - [Fase 1 — L1 Infrastruttura (Ollama)](#fase-1--l1-infrastruttura-ollama--modelli)
  - [Fase 2 — L3 RAG (AnythingLLM)](#fase-2--l3-rag-anythingllm)
  - [Fase 3 — L2 Knowledge (book-to-skill)](#fase-3--l2-knowledge-book-to-skill)
  - [Fase 4 — L3 Agente (OpenWorker, opzionale)](#fase-4--l3-agente-openworker-opzionale)
- [Architettura finale](#-architettura-finale)
- [Metodi di context engineering](#-metodi-di-context-engineering-l4)
- [Troubleshooting](#-troubleshooting)
- [Note di compliance](#-note-di-compliance)
- [Riferimenti](#-riferimenti)

---

## 🎯 Cos'è questo stack

Un'architettura a strati dove **ogni livello alimenta quello sopra**. La stessa base (runtime + modello) serve due "teste" intercambiabili:

- **Testa RAG** → interroga i tuoi documenti con citazioni
- **Testa Agente** → esegue task operativi

Il principio: si installa il runtime **una volta**, poi si montano sopra prima il RAG, poi — all'occorrenza — l'agente, senza rifare nulla. Entrambe le teste puntano allo **stesso endpoint OpenAI-compatibile** (`localhost:11434`).

---

## 🏛 Architettura a livelli

```
┌──────────────────────────────────────────────────────────────┐
│  L4 · METODO / CONTEXT ENGINEERING                          │
│  Come far lavorare insieme i pezzi (RAG, Wiki, GraphRAG…)    │
├──────────────────────────────────────────────────────────────┤
│  L3 · APPLICAZIONE / AGENTI                                 │
│  AnythingLLM (RAG)   ·   OpenWorker / Hermes (agenti)       │
├──────────────────────────────────────────────────────────────┤
│  L2 · CONTESTO / KNOWLEDGE                                  │
│  book-to-skill  ·  Obsidian / Tolaria  ·  OpenViking        │
├──────────────────────────────────────────────────────────────┤
│  L1 · INFRASTRUTTURA / MODELLO                             │
│  Ollama (runtime)   +   Qwen3 (modello)                     │
└──────────────────────────────────────────────────────────────┘
```

| Livello | Funzione | Domanda a cui risponde |
|---|---|---|
| **L1** | Serve il modello | Come faccio girare un LLM in locale? |
| **L2** | Nutre il modello di conoscenza | Come do al modello cosa sapere? |
| **L3** | Fa agire il modello | Come lo interrogo (RAG) e come lo faccio operare (agente)? |
| **L4** | Integra tutto in una pratica | Come faccio lavorare insieme i pezzi? |

---

## ⭐ La via singola raccomandata

Per iniziare senza dispersione, **un elemento per livello** (tailoring: complessità adeguata al contesto):

| Livello | Scelta primaria | Estensioni future |
|---|---|---|
| **L1** | Ollama + Qwen3 8B | LM Studio, llama.cpp, modelli più grandi |
| **L2** | book-to-skill | Obsidian / Tolaria (LLM Wiki curata) |
| **L3** | AnythingLLM (RAG) | OpenWorker / Hermes (agente pieno) |
| **L4** | RAG vettoriale (in AnythingLLM) | GraphRAG, Agent Memory, ACE |

> **Risultato:** solo **3 installazioni reali** (Ollama, AnythingLLM, book-to-skill), perché L4 è già dentro AnythingLLM. È il "sistema minimo che funziona".

---

## 💻 Requisiti hardware

### Macchina di riferimento

| Componente | Valore | Impatto |
|---|---|---|
| **GPU / VRAM** | RTX 4060 — **8 GB** | ⚠️ Il vincolo chiave |
| RAM sistema | 32 GB | ✅ Abbondante |
| CPU | i7-11700F (8 core) | ✅ Buona per fallback |
| Storage | NVMe (modelli) | ✅ Load veloce |

### ⚠️ Regola d'oro della VRAM

> Se un modello **entra** nella VRAM → 20-30+ token/sec.
> Se **sfora** e Ollama scarica strati su RAM via PCIe → la velocità **crolla di 5-30×** (da ~40 a ~8 tok/sec o meno).
> **Conseguenza: scegliere modelli che stanno negli 8 GB.**

### Cosa regge una RTX 4060 8GB

| Uso | Modello | VRAM | Esito |
|---|---|---|---|
| RAG generativo | `qwen3:8b` (Q4) | ~5 GB | ✅ Ottimo |
| Coding / agente leggero | `qwen3-coder:8b` (Q4) | ~5 GB | ✅ Ottimo |
| Embedder RAG | `nomic-embed-text` | ~0.3 GB | ✅ Sempre |
| Agente pieno | `qwen3:30b-a3b` (Q4) | ~18 GB | ❌ Non in VRAM |

### Strategia a due velocità

- **Profilo A — Veloce** (`qwen3:8b` / `qwen3-coder:8b`): in VRAM, uso quotidiano, RAG + coding.
- **Profilo B — Potente** (`qwen3:30b-a3b`): fuori VRAM, lento (~5-8 tok/sec), solo task agentici non interattivi.

> 💡 **Consiglio:** parti col Profilo A (copre l'obiettivo primario). Attiva il B solo quando un task reale lo richiede.

---

## 🚀 Guida di installazione

> **Nota Windows:** i percorsi in stile `~/.claude/` vanno tradotti in `%USERPROFILE%\.claude\`. I comandi `ollama` e `git` sono identici su tutte le piattaforme.

### Fase 1 — L1 Infrastruttura (Ollama + modelli)

**Obiettivo:** runtime attivo con endpoint OpenAI-compatibile su `localhost:11434`.

#### 1.1 Installare Ollama

Scarica da [ollama.com](https://ollama.com), installa, poi verifica:

```bash
ollama --version
# atteso: ollama version 0.x.x
```

#### 1.2 Scaricare i modelli

```bash
# --- Profilo A: veloce, in VRAM ---
ollama pull qwen3:8b            # RAG generativo (~5 GB)
ollama pull qwen3-coder:8b      # variante coding (~5 GB)

# --- Embedder per il RAG (INDISPENSABILE) ---
ollama pull nomic-embed-text    # ~275 MB

# --- Profilo B: potente, occasionale (opzionale) ---
ollama pull qwen3:30b-a3b       # MoE ~18 GB, lento su 8GB VRAM
```

#### 1.3 Verificare

```bash
ollama list
ollama run qwen3:8b "Ciao, rispondi in una riga"
curl http://localhost:11434/v1/models
```

✅ **Checkpoint L1:** se `ollama list` mostra i modelli e `curl` restituisce un JSON, la fondazione è pronta. **Non procedere finché questo non funziona.**

---

### Fase 2 — L3 RAG (AnythingLLM)

**Obiettivo:** il tuo "RAG ottimo in locale". [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) (MIT) si collega a Ollama e gestisce ingestione, vector DB e citazioni.

#### 2.1 Installare

**Opzione Desktop (consigliata Windows):** scarica da [anythingllm.com](https://anythingllm.com).

**Opzione Docker:**

```bash
docker run -d -p 3001:3001 \
  --add-host=host.docker.internal:host-gateway \
  -v anythingllm_storage:/app/server/storage \
  mintplexlabs/anythingllm
```

#### 2.2 Configurare su Ollama

| Impostazione | Valore |
|---|---|
| LLM Provider | `Ollama` |
| Ollama Base URL | `http://localhost:11434` *(Docker: `http://host.docker.internal:11434`)* |
| Chat Model | `qwen3:8b` |
| Embedding Provider | `Ollama → nomic-embed-text` |
| Vector Database | `LanceDB` (default, locale) |

#### 2.3 Primo test RAG

1. Crea un **Workspace** (es. `Deliverable-PMI`)
2. Carica 2-3 documenti di test
3. Fai una domanda specifica e **verifica che la risposta citi i chunk corretti**

✅ **Checkpoint L3-RAG:** se le risposte citano correttamente i documenti, hai il RAG ottimo funzionante.

> ⚠️ Se le citazioni sono imprecise: aumenta il numero di chunk recuperati, o passa a `qwen3:14b`. **Mai** modelli sotto i 7B per il RAG (allucinano).

---

### Fase 3 — L2 Knowledge (book-to-skill)

**Obiettivo:** trasformare documenti e libri tecnici in skill riusabili dagli agenti. Elaborazione **100% locale**.

#### 3.1 Prerequisiti

```bash
python3 --version              # atteso 3.10+
pip3 install docling           # PDF tecnici (tabelle, codice)
pip3 install pypdf ebooklib beautifulsoup4   # fallback + EPUB
```

#### 3.2 Installare

```bash
git clone https://github.com/virgiliojr94/book-to-skill.git \
  ~/.claude/skills/book-to-skill

# verifica estrattori disponibili
python3 ~/.claude/skills/book-to-skill/scripts/extract.py --check
```

#### 3.3 Generare una skill

```bash
/book-to-skill ./PMBOK-cap7.pdf
# genera: SKILL.md + chapters/ + glossary.md + patterns.md + cheatsheet.md
# poi interroghi on-demand: /pmbok-cap7 <argomento>
```

> 📉 **Efficienza:** 24-51× meno token rispetto a versare il libro nel contesto.

---

### Fase 4 — L3 Agente (OpenWorker, opzionale)

**Obiettivo:** l'agente operativo "all'occorrenza". **Attiva solo dopo** che L1-L3-RAG sono solidi.

#### 4.1 Installare

```bash
git clone https://github.com/andrewyng/openworker
cd openworker
bash packaging/setup_dev_env.sh   # Python 3.10+, Node 20+, Rust
```

#### 4.2 Configurazione (`config.toml`)

Percorso: `%APPDATA%\coworker\config.toml` (Windows) · `~/.config/coworker/config.toml` (Linux/Mac)

```toml
model = "ollama:qwen3-coder:8b"
mode = "interactive"          # conferma su ogni scrittura — NON cambiare all'inizio
max_iterations = 100
allowed_commands = ["ls","cat","pwd","grep","git status","git diff"]
host = "127.0.0.1"
port = 8765

# Per task pesanti, cambia a runtime:
# model = "ollama:qwen3:30b-a3b"   (lento su 8GB VRAM)
```

#### 4.3 Verifica del tool calling

> **Test decisivo:** che il modello **esegua davvero i tool**, non solo li descriva. Con `qwen3-coder:8b` funziona. Assegna un task reale a basso rischio in un workspace sacrificabile.

🔒 **Sicurezza — non negoziabile:** mantieni `mode = "interactive"` nelle prime sessioni. L'agente ha accesso a shell e file: la conferma su ogni scrittura è la difesa contro prompt injection. Sali ad `auto` solo su workspace sacrificabili.

---

## 🔗 Architettura finale

```
  AnythingLLM (:3001)  ─┐
                        ├──►  Ollama (:11434)  ──►  modelli on-demand
  OpenWorker (:8765)  ──┘         (localhost)        qwen3:8b / coder / 30b-a3b

  book-to-skill  ──►  genera skill locali  ──►  usate da OpenWorker
```

Un solo Ollama, due teste. Ollama carica il modello giusto al momento giusto. Tutto sulla macchina.

### Sequenza minima

| # | Fase | Comando chiave | Esito |
|---|---|---|---|
| 1 | Ollama + modelli | `ollama pull qwen3:8b` | Endpoint attivo |
| 2 | AnythingLLM | installa + config Ollama | RAG funzionante |
| 3 | book-to-skill | `git clone` + `--check` | Skill da doc |
| 4 | OpenWorker *(opz.)* | `setup_dev_env.sh` | Agente pronto |

---

## 🧩 Metodi di context engineering (L4)

Il RAG vettoriale è il punto di partenza, ma non l'unico metodo. In produzione **si combinano**:

| Metodo | Punto di forza | Quando usarlo |
|---|---|---|
| **RAG vettoriale** | Semplice, veloce | Q&A su documenti *(default)* |
| **GraphRAG** | Ragionamento multi-hop (+35% precisione) | Conoscenza con relazioni forti |
| **Agent Memory** | Continuità tra sessioni | Agenti con utenti ricorrenti |
| **LLM Wiki** | Curata, versionabile | Knowledge base di dominio |
| **ACE / Agentic** | Auto-evolutivo | Frontiera sperimentale |

> ⚠️ Tutti degradano se i dati sottostanti sono obsoleti o non governati. **La governance dei dati viene prima del metodo.**

---

## 🛠 Troubleshooting

<details>
<summary><b>Ollama è lentissimo (pochi token/sec)</b></summary>

**Causa più probabile:** il modello non entra in VRAM e gira su CPU, oppure Ollama non usa la GPU.

- Verifica che il modello scelto stia negli 8 GB (usa `qwen3:8b`, non `qwen3:30b-a3b`).
- Controlla i log all'avvio di Ollama: deve rilevare la GPU NVIDIA.
- Aggiorna i driver NVIDIA e verifica il supporto CUDA.
- Con `ollama ps` controlli se il modello è caricato su GPU o CPU.
</details>

<details>
<summary><b>Out-of-memory quando carico un modello</b></summary>

- Il modello è troppo grande per la VRAM. Scendi di taglia (14B → 8B → 4B).
- Riduci la context window nelle impostazioni.
- Chiudi altre applicazioni che usano la GPU.
</details>

<details>
<summary><b>AnythingLLM non si collega a Ollama</b></summary>

- **Desktop:** usa `http://localhost:11434`.
- **Docker:** usa `http://host.docker.internal:11434` e aggiungi `--add-host=host.docker.internal:host-gateway` al `docker run`.
- Verifica che Ollama sia in esecuzione: `curl http://localhost:11434/v1/models`.
</details>

<details>
<summary><b>Il RAG allucina o cita male</b></summary>

- Aumenta il numero di chunk recuperati nel workspace.
- Verifica che l'embedder `nomic-embed-text` sia configurato (non solo il modello chat).
- Passa a `qwen3:14b` per risposte più affidabili (a costo di velocità).
- Non usare modelli sotto i 7B per il RAG.
</details>

<details>
<summary><b>L'agente "risponde ma non fa niente"</b></summary>

- **Il modello non fa tool calling.** È il fallimento più comune.
- Usa un modello con function calling nativo (`qwen3-coder:8b`, `qwen3:30b-a3b`).
- Se usi llama.cpp come backend, il flag `--jinja` è indispensabile per abilitare i tool.
</details>

<details>
<summary><b>Errore CUDA / GPU non rilevata su Windows</b></summary>

- Aggiorna i driver NVIDIA all'ultima versione.
- Riavvia dopo l'installazione di Ollama.
- Verifica che la GPU sia visibile: nei log Ollama deve comparire il rilevamento CUDA.
</details>

---

## 🔐 Note di compliance

- ✅ **Tutto locale:** nessun byte esce dalla macchina — coerente con EU AI Act e dati proprietari.
- ⛔ **OpenViking escluso:** unico componente che manderebbe dati a cloud esterno (chiavi Volcengine). Non incluso di proposito.
- 📜 **Licenze:** Ollama, AnythingLLM, OpenWorker, book-to-skill sono **MIT**. Tolaria/OpenViking sono AGPL; Obsidian è proprietario. Verifica formale prima di deploy produttivo.
- 🔒 **Agente in `interactive`:** difesa contro azioni indesiderate; il transcript per ogni run supporta l'audit.

---

## 🗂 Struttura del repository

```
.
├── README.md                              # Questo file (hub principale)
├── LICENSE                                # MIT
├── CONTRIBUTING.md                        # Linee guida per contributi
├── CHANGELOG.md                           # Storico versioni
├── .gitignore                             # Esclusioni (modelli, chiavi, DB…)
├── .env.example                           # Template variabili (copia in .env)
└── docs/
    ├── Manuale_LLM_Locale.docx                  # Libro di testo completo (da neofita)
    ├── Appendice_Tecniche_Frontiera.docx        # RMM, GraphRAG, Harness, LoRA/QLoRA (avanzato)
    ├── Pattern_LLM_Wiki_Operativo.docx          # Pattern LLM Wiki Ingest/Query/Lint (local-first)
    ├── Guida_Agent_Loop_Multi_Agente.docx       # Loop dell'agente (3 livelli) + sistemi multi-agente
    ├── harness_diagram.svg                       # Diagramma originale del flusso di affidabilità
    ├── harness_diagram.png                       # (versione raster)
    ├── llmwiki_diagram.svg                       # Diagramma originale del ciclo LLM Wiki
    ├── llmwiki_diagram.png                       # (versione raster)
    ├── loop_levels_diagram.svg                   # Diagramma originale dei 3 livelli del loop
    ├── loop_levels_diagram.png                   # (versione raster)
    ├── mas_diagram.svg                           # Diagramma originale dei sistemi multi-agente
    ├── mas_diagram.png                           # (versione raster)
    ├── Percorso_Studio_Stack_LLM_Locale.docx    # Complemento teorico sintetico
    └── Guida_Installazione_Stack_LLM.docx       # Guida operativa
```

> ⚠️ **Prima del primo commit:** copia `.env.example` in `.env` e compila i valori. Il `.env` reale è già escluso dal versionamento.

### Documenti di approfondimento

| Documento | Contenuto |
|---|---|
| **Manuale «LLM in Locale»** *(docx)* | Libro di testo completo (8 capitoli): fondamenti, embeddings, RAG, agenti, context engineering, con definizioni formali, esempi ed esercizi. Da neofita. |
| **Appendice — Tecniche di Frontiera** *(docx)* | Documento avanzato unico (16 pag) con indice e diagramma originale: memoria a lungo termine (RMM), ragionamento a grafo (GraphRAG), imbracatura di affidabilità (Harness Engineering) e adattamento del modello (fine-tuning, LoRA, QLoRA). Con criteri di adozione. |
| **Pattern LLM Wiki operativo** *(docx)* | Scheda pratica (7 pag): il pattern LLM Wiki di Karpathy (Ingest/Query/Lint) adattato local-first, con schema `AGENTS.md` pronto all'uso, struttura file e comandi. Con diagramma originale. |
| **Il loop dell'agente e i sistemi multi-agente** *(docx)* | Guida estesa (17 pag) in due parti: i tre livelli di maturità del loop di un singolo agente, e l'architettura dei sistemi multi-agente (strati, coordinamento, affidabilità, governance). Con due diagrammi originali. |
| **harness_diagram** *(svg/png)* | Diagramma originale del flusso di affidabilità attorno all'LLM (asset riutilizzabile). |
| **Percorso di Studio** *(docx)* | Complemento teorico sintetico della guida: il «perché» dietro ogni fase. |
| **Guida di Installazione** *(docx)* | Guida operativa passo-passo tarata su RTX 4060 8GB: il «come fare». |

---

## 📚 Riferimenti

### Componenti core

- **Ollama** — <https://github.com/ollama/ollama> · MIT · runtime
- **AnythingLLM** — <https://github.com/Mintplex-Labs/anything-llm> · MIT · RAG
- **book-to-skill** — <https://github.com/virgiliojr94/book-to-skill> · MIT · doc→skill
- **OpenWorker** — <https://github.com/andrewyng/openworker> · MIT · agente

### Estensioni

- **Tolaria** — <https://github.com/refactoringhq/tolaria> · AGPL · PKM AI-native
- **Obsidian** — <https://obsidian.md> · proprietario · PKM
- **OpenViking** — <https://github.com/volcengine/OpenViking> · AGPL · context DB *(cloud)*
- **Hermes Agent** — <https://github.com/NousResearch/hermes-agent> · agente self-improving

### Modelli

- Ollama library — <https://ollama.com/library>
- Qwen3: `qwen3:8b` · `qwen3-coder:8b` · `qwen3:30b-a3b`
- Embedder: `nomic-embed-text`

---

<sub>Documento di studio personale · Stack local-first · Agosto 2026</sub>
