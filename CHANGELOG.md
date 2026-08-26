# Changelog

Tutte le modifiche rilevanti a questo repository sono documentate qui.

Il formato segue [Keep a Changelog](https://keepachangelog.com/it/1.1.0/)
e il progetto adotta il [Versioning Semantico](https://semver.org/lang/it/).

## [Unreleased]

### Da valutare
- Sezione GraphRAG come estensione L4
- Docker Compose unificato per l'intero stack
- Script di setup automatico (`setup.sh` / `setup.ps1`)

## [1.0.0] — 2026-08-24

### Aggiunto
- Architettura a 4 livelli (L1 Infrastruttura, L2 Knowledge, L3 Applicazione, L4 Metodo/Context Engineering)
- Guida di installazione completa tarata su RTX 4060 8GB / 32GB RAM
- Strategia a due velocità (Profilo A veloce in VRAM, Profilo B potente occasionale)
- Fase 1 — Ollama + modelli Qwen3 + embedder
- Fase 2 — AnythingLLM (RAG) su Ollama
- Fase 3 — book-to-skill (documento → skill)
- Fase 4 — OpenWorker (agente, opzionale)
- Sezione metodi di context engineering (RAG, GraphRAG, Agent Memory, LLM Wiki, ACE)
- Troubleshooting con casi comuni (VRAM, CUDA, tool calling, RAG)
- Note di compliance (local-first, EU AI Act, licenze)
- File di corredo: LICENSE (MIT), .gitignore, .env.example, CONTRIBUTING.md
- Documenti di studio in Word: percorso di studio (complemento teorico) e guida di installazione

### Documento di studio — v2
- Riscritto come complemento teorico della guida (il «perché» accanto al «come»)
- Aggiunti fondamenti: inferenza vs addestramento, quantizzazione, VRAM/PCIe, densi vs MoE
- Aggiunta teoria embeddings, chunking e pipeline RAG passo-passo
- Aggiunta teoria loop agentico, tool calling e permessi
- Aggiunta sezione autovalutazione per ogni tappa

### Manuale didattico e appendice
- Manuale «LLM in Locale» (8 capitoli, stile universitario): libro di testo da neofita con definizioni formali, esempi ed esercizi
- Appendice «Tecniche di Frontiera» consolidata in documento unico (16 pagine, con indice e diagramma originale):
  - A: memoria a lungo termine (Reflective Memory Management)
  - B: ragionamento a grafo (GraphRAG)
  - C: imbracatura di affidabilità (Harness Engineering), con diagramma originale del flusso, osservabilità e ciclo di feedback
  - D: adattamento del modello (fine-tuning, LoRA, QLoRA; RAG vs fine-tuning)
- Diagramma originale `harness_diagram` (SVG + PNG) come asset riutilizzabile

### Note
- OpenViking escluso di proposito dallo stack locale (richiede chiavi cloud).
- Tutti i componenti core sono MIT; estensioni AGPL segnalate.
