# LibrAIry  
**Retrieval-Augmented Reading Assistant for Your Personal e-Library**

[![Build](https://img.shields.io/github/actions/workflow/status/your-org/librairy/ci.yml?branch=main)](../../actions)  
[![Docker](https://img.shields.io/badge/docker-ready-to-run-2496ED?logo=docker&logoColor=white)](https://hub.docker.com/u/your-org)  
[![License](https://img.shields.io/github/license/your-org/librairy)](LICENSE)

---

> **En deux mots** : LibrAIry indexe tous vos PDF/EPUB libres d’accès (hébergés sur votre espace Mega, un NAS ou simplement votre disque) et vous offre un
> **chatbot contextuel**, des **résumés**, des **flashcards** et même un mode
> **lecture audio** – le tout propulsé par un pipeline RAG de pointe.

---

## ✨ Fonctionnalités principales

| Fonction | Détails techniques | Valeur utilisateur |
|-----------|-------------------|--------------------|
| **Chat universel** | RAG • Llama 3/Mixtral • bge-reranker | Posez n’importe quelle question, recevez une réponse sourcée paragraphe + page. |
| **Recherche sémantique** | Embeddings Instructor-XL • Qdrant | Trouvez un concept même si les mots exacts diffèrent. |
| **Résumés & plans** | Summarisation (Llama 3) | Synthèse par chapitre ou par ouvrage complet. |
| **Flashcards automatiques** | LLM chain → format Anki `.apkg` | Révisez efficacement et exportez vers AnkiMobile. |
| **Lecture audio TTS** | Bark / XTTS | Convertit un extrait en podcast (MP3). |
| **Tableau de bord d’apprentissage** | TimescaleDB • Grafana | Suivi du temps de lecture, thèmes, progression. |
| **Recommandations croisées** | Classification + similarity | “Si ce chapitre vous a plu, lisez aussi…” |

---

## 🧐 Pourquoi LibrAIry ?

* **Déniche l’info enfouie** dans des milliers de pages PDF.
* Fonctionne **entièrement hors-ligne** (utile pour contenu sensible).
* **Project-portfolio** complet : ingestion → vecteur DB → RAG → UI → MLOps.
* Combine **Python pour le ML** et **Rust pour la performance**.

---

## 🏗️ Architecture

          +-------------+
```
User Chat ⇆   | FastAPI API |  ← Auth, Rate-limit
+——+——+
|
+———–+———–+
|  RAG Orchestrator     |  (LangChain / LlamaIndex)
+———–+———–+
|
┌────────Retrieve─────────┐
│  Vector DB (Qdrant)     │
└─────────▲───────────────┘
│ embeddings
PDF/EPUB   │
─────────────┘
Ingestion Pipeline (Python)
├─ PyMuPDF / Tika         # extraction texte
├─ OCR (Tesseract)        # pages scannées
└─ Embeddings (Instructor-XL)

After Retrieve
▼
Re-rank (bge-reranker) →  Context Compressor  →  LLM (Llama 3)
│
Summaries / Flashcards / TTS

*Toutes les briques sont containerisées ; orchestration : **Docker Compose** (dev) ou **Helm/k3s** (prod).*
```
---

## 🧰 Stack technologique

| Couche | Choix | Raisons |
|--------|-------|---------|
| **Ingestion** | Python 3.11 • PyMuPDF • pdfminer.six • unstructured • Tesseract-OCR | Extraction fiable, multi-langue. |
| **Embeddings** | `sentence-transformers/instructor-xl` (GGUF quantisé) | Excellente perf multilingue. |
| **Vector Store** | **Qdrant** (Docker) | API gRPC & REST, scale horizontale. |
| **RAG Framework** | **LangChain** ou **LlamaIndex** | Graph d’outils flexible. |
| **LLM** | Llama 3 8B Instruct (gguf Q4) via `llama.cpp-server` (Rust) | Latence <1 s sur GPU moyen ou MPS. |
| **Backend API** | **FastAPI** | Docs auto (Swagger), WebSocket facile. |
| **Service haute perf** | **Rust/axum** – streaming TTS & gros download | Apprentissage Rust + débit. |
| **Frontend** | **Next.js 14** (React Server) • Tailwind • shadcn/ui | SSR performant, dark/light. |
| **MLOps** | MLflow • DVC • Prefect • Prometheus • Grafana | Traçabilité, monitoring, pipelines planifiés. |
| **CI/CD** | GitHub Actions • Docker Buildx • Helm | Ligne verte jusqu’à k3s. |

---

## ⚡ Mise en route rapide

### 1. Prérequis

| Outil | Version minimale |
|-------|------------------|
| Docker / Compose | 24 + (compose v2) |
| Python | 3.11 |
| Node.js | 20 |
| Rust | 1.76 |
| Tesseract OCR | 5.3 |
| (Option) k3d/k3s | 1.29 |

### 2. Clone & quickstart

```bash
git clone https://github.com/your-org/librairy.git
cd librairy
make quickstart           # build & docker-compose up -d
open http://localhost:3000  # UI Next.js
```
Le script make quickstart :
	1.	télécharge/installe les modèles (gguf, bark).
	2.	crée les conteneurs Postgres, Qdrant, FastAPI, llama.cpp-server.
	3.	expose l’UI sur localhost:3000.

3. Indexer vos livres
	1.	Déposez vos PDF/EPUB/CBZ dans data/books/.
	2.	Exécutez : ``` python ingest/run_pipeline.py --path data/books ```
	3.	Surveillez l’avancement dans prefect Orion (http://localhost:4200).

Tips : gros corpus ? Ajoutez l’option --chunk 100 pour batcher les
embeddings et éviter l’OOM.

4. Tester l’API
```bash
curl -X POST http://localhost:8000/chat \
     -d '{"query": "Explique la différence entre monades et applicatives"}'
```

⸻

📂 Structure du repo
```
librairy/
├── ingest/                # pipelines Prefect + scripts d'extraction
├── rag/                   # orchestrateur LangChain
├── api/                   # FastAPI routes
├── rust-services/
│   └── tts-stream/        # service Rust axum + Bark
├── frontend/              # Next.js 14 app
├── deployments/
│   ├── docker-compose.yaml
│   └── helm/
├── mlops/
│   ├── mlruns/            # MLflow
│   └── dvc.yaml
└── .github/workflows/     # CI/CD
```

⸻

🔍 Observabilité
	•	Prometheus : métriques rag_latency_seconds, query_error_total.
	•	Grafana : dashboards prêts dans dashboards/.
	•	OpenTelemetry : traces end-to-end (optionnel).
	•	Sentry : erreurs front & back.

⸻

🗓️ Roadmap 8 semaines

Semaine	Livrable
1	Docker skeleton + Qdrant + FastAPI alive.
2	Pipeline d’extraction & embeddings (PDF + OCR).
3	Endpoint RAG POC (retrieve + LLM).
4	Re-ranker + citations précises.
5	UI chat + file manager + surlignage page.
6	Flashcards Anki + service TTS streaming.
7	Dashboard apprentissage (Grafana) + stats.
8	MLOps, CI/CD, doc complète, démo vidéo.


⸻

🔒 Licences & éthique
	•	Contenu : assurez-vous que vos ouvrages sont dans le domaine public ou
sous licence Creative Commons.
	•	Modèles : Llama 3 est CC-BY-SA ; MiDaS CC-BY-NC, etc. Vérifiez avant mise
en prod commerciale.
	•	Usage : LibrAIry ne stocke pas vos PDF hors de votre infra – pratique pour
protéger du contenu sensible.

⸻

🤝 Contribuer
	1.	Fork, git clone, make dev-setup (installe pre-commit).
	2.	Créez une branche feat/ma-feature.
	3.	Commits style Conventional Commits.
	4.	Ouvrez un PR, laissez les tests CI passer.

Voir CONTRIBUTING.md & CODE_OF_CONDUCT.md.

⸻

📜 Licence

Distribué sous licence MIT – voir LICENSE.

⸻

🙏 Remerciements
	•	Hugging Face community pour les modèles open-weight.
	•	Projets LangChain et LlamaIndex pour le tooling RAG.
	•	Qdrant, Prefect, Grafana pour leurs outils open-source formidables.

⸻

📫 Contact

Support	Lien
Email	dev@your-domain.com
Twitter	@YourHandle
Discord	discord.gg/abcdefg

Bon hacking et bonnes lectures ! 📚

