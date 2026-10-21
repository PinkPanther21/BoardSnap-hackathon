# BoardSnap 📸✨
> **Snap a whiteboard. Get clean notes, a redrawn diagram, and a quiz.**

[![Banana Hacks 2026](https://img.shields.io/badge/Event-Banana%20Hacks%202026-yellow.svg)](https://devpost.com)
[![Theme](https://img.shields.io/badge/Theme-Image%20AI-blue.svg)](https://devpost.com)
[![Frontend](https://img.shields.io/badge/Frontend-React%20%7C%20Tailwind-emerald.svg)](./frontend)
[![Backend](https://img.shields.io/badge/Backend-Node.js%20%7C%20Express-teal.svg)](./backend)
[![Vision](https://img.shields.io/badge/Vision-OpenCV%20%7C%20PyTorch-orange.svg)](./vision)
[![LLM](https://img.shields.io/badge/Language-LangChain%20%7C%20RAG-purple.svg)](./llm)

**Team:** Ammar, Muavia, Atiqua, Awais  
 

---

## 🌟 The Problem
After a university lecture or brainstorm, the whiteboard is the only record. Students photograph it, then the photo sits in a phone gallery forever:
- Handwriting is illegible with skewed angles and harsh glare.
- Hand-drawn diagrams are locked in pixels and cannot be edited.
- Nobody converts the board into structured study materials or quizzes.

## 💡 The Solution
BoardSnap turns raw whiteboard photos into:
1. **Perspective-rectified, glare-attenuated clean imagery**.
2. **AI-detected regions** separating handwritten text blocks from flowchart shapes.
3. **Editable Mermaid vector diagrams** with SVG export.
4. **Strictly grounded study notes** (Key Points, Definitions, Action Items) with zero hallucinations.
5. **Interactive active recall quizzes, flashcards**, and a conversational RAG store.

---

## 🏗️ Repository Architecture
```text
boardsnap/
├── frontend/           # React + Tailwind CSS + Mermaid + Lucide UI (Moyeon)
│   ├── src/
│   │   ├── components/ # Comparison slider, diagram viewer, quiz, RAG chat
│   │   ├── data/       # Sample benchmark boards (Architecture, ML, Glare test)
│   │   └── utils/      # API client & simulation fallback
│   └── package.json
│
├── backend/            # Express.js REST API & Agent Orchestrator (Ammar)
│   ├── server.js       # API startup and graceful shutdown
│   ├── src/            # Routes, pipeline, MongoDB/in-memory repository, service clients
│   ├── test/           # API and end-to-end pipeline contract tests
│   └── package.json
│
├── vision/             # Computer Vision & ML services (Muavia)
│   ├── main.py         # OpenCV cleanup, Roboflow detection, TrOCR & spatial graph builder
│   └── requirements.txt
│
├── llm/                # Grounded language & RAG services (Awais)
│   ├── main.py         # Strict note synthesis, quiz generation & vector chat
│   └── requirements.txt
│
├── dataset/            # Roboflow dataset downloader and download manifest
│   └── roboflow/       # Downloaded dataset exports (not committed)
│
└── docs/               # Hackathon submission documentation
    ├── EVALUATION_REPORT.md  # Quantitative metrics table (Detector, OCR, Graph)
    ├── DEMO_VIDEO_SCRIPT.md  # 2:40 second-by-second video script
    └── DEVPOST_SUBMISSION.md # Complete Devpost submission write-up
```

---

## 🚀 Quick Start & How to Run

### 1. Run the Frontend (React + Vite)
```bash
cd frontend
npm install
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser.

*Note: The frontend includes full client-side pipeline simulation and pre-packaged benchmark boards, allowing judges to test camera capture, diagram rebuilding, and quiz generation instantly even without spinning up local Python microservices!*

### 2. Run the Express Backend
```bash
cd backend
npm install
# Copy .env.example to .env and configure service URLs / MongoDB if needed
npm run dev
```
Runs the orchestration API on [http://localhost:5000](http://localhost:5000). Without
`MONGODB_URI`, boards are kept in memory for local development and are lost when
the server restarts. Set `MONGODB_URI` to enable MongoDB persistence. Uploaded
images are stored in `backend/uploads` by default; production deployments need
a persistent disk or a configured `UPLOAD_DIR`.

### 3. Run the Vision & ML Service
```powershell
cd vision
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
python -m pip install -r requirements.txt
if (-not (Test-Path .env)) { New-Item -ItemType File -Path .env }
# Edit .env locally; never copy a private key into .env.example
python -m uvicorn main:app --port 8000 --reload
```

The Python virtual environment keeps ML packages isolated from other project
work. TrOCR downloads its pretrained weights from Hugging Face on first
inference. In `vision/.env`, set `ROBOFLOW_API_KEY` and paste the complete
inference URL for the region-detection workflow into `ROBOFLOW_WORKFLOW_URL`.
The Qwen-VL OCR workflow is configured by default at
`https://serverless.roboflow.com/moyeon-abrar/workflows/qwen-vl`; override it
with `ROBOFLOW_OCR_WORKFLOW_URL` if needed. Both workflows use the same
`ROBOFLOW_API_KEY`. `ROBOFLOW_MODEL_ID` is only a detector log label. The
service does not contain trained project YOLO weights. When the Roboflow key
is not configured, OCR continues to use the existing local OCR path. TrOCR
processes detected text blocks one line at a time. When configured, Qwen-VL
receives the full cleaned board once (not detector-limited crops), and its
transcription is passed to notes generation and converted into a top-to-bottom
flowchart using each non-empty transcript line as a sequential step. To select the smaller
official checkpoint on memory-limited systems, set
`TROCR_MODEL_ID=microsoft/trocr-small-handwritten` in the local `.env`; its
accuracy has not been benchmarked for this project. Never commit `vision/.env`
or paste private API keys into chat.

The backend calls the vision service at `http://localhost:8000` and the language
service at `http://localhost:8001` by default. Configure `VISION_SERVICE_URL`
and `LLM_SERVICE_URL` in `backend/.env` when these services are hosted elsewhere.
When detector configuration is missing, `/vision/detect` returns a clear
service error instead of demo regions. Set `ALLOW_DEMO_DETECTOR_FALLBACK=true`
only when intentionally demonstrating fallback data. An image
upload returns a queued board, then the frontend polls `GET /api/boards/:id`
while the backend runs cleanup, detection, OCR, graph construction, grounded
notes, and quiz generation. A quality-gate rejection completes with
`isQualityPass: false` so the UI can ask for a retake. Service or pipeline
failures are reported with `status: "failed"` and an error; they are not
replaced by fabricated sample-board results.

The public API also includes `POST /api/boards/:id/quiz`, `POST /api/chat`,
`GET /api/boards/:id/export?format=md`, and `GET /api/health`. Run backend
contract tests with `cd backend && npm test`.

Check vision setup and run local tests from the active `vision` virtual
environment with `python -m unittest test_vision.py`. The previous hard-coded
OCR benchmark values have been removed. To measure TrOCR on actual held-out,
manually transcribed image crops, create a JSON manifest with `image`,
`ground_truth`, and optional normalized `box: [x, y, width, height]` fields and
run `python benchmark_ocr.py --manifest path\to\test-manifest.json`. These
Roboflow datasets contain region annotations, not OCR transcripts; they are
not evidence of measured handwriting accuracy.

### 4. Run the Language Service
```bash
cd llm
pip install -r requirements.txt
# Copy .env.example to .env and set the API key for your selected provider
uvicorn main:app --port 8001 --reload
```

The language service supports Gemini, xAI Grok, and GroqCloud. For GroqCloud's
free plan (subject to rate limits), set `LLM_PROVIDER=groq` and `GROQ_API_KEY`;
`GROQ_MODEL` defaults to `openai/gpt-oss-120b`. Alternatively, use
`LLM_PROVIDER=gemini` with `GEMINI_API_KEY`, or `LLM_PROVIDER=grok` with
`XAI_API_KEY` and `GROK_MODEL`. The selected provider's key is required;
provider or generation errors are reported and the board job is marked
`failed` instead of returning unrelated sample content.

### 5. Download an annotated whiteboard dataset

The downloader fetches two separate public Roboflow Universe datasets in YOLOv8
format, including their annotations and native train/valid/test splits:

- [Whiteboard](https://universe.roboflow.com/taerua/whiteboard-ctvgi), version
  1: 8,784 images, instance-segmentation classes `pen` and `whiteboard`.
- [Handwriting detection](https://universe.roboflow.com/handwriting-detection-zfowd/handwriting-detection-eoxpj),
  version 8: 361 images, object-detection class `handwriting`.

Both project pages list the dataset license as CC BY 4.0. Keep the source
attribution when using or redistributing the exports. These datasets detect
whiteboard regions and handwriting regions; the handwriting labels do not
provide transcribed text for OCR training.

In PowerShell, install the downloader dependency, set your Roboflow API key,
and run the downloader:

```powershell
pip install -r dataset\requirements.txt
$env:ROBOFLOW_API_KEY = "your-roboflow-api-key"
python dataset\download_samples.py
```

The exports are saved separately under `dataset\roboflow\whiteboard-v1` and
`dataset\roboflow\handwriting-v8`. `dataset\manifest.json` records the source,
license, and downloaded image counts. The API key is read from the environment
and is not written to the manifest. Use `--format` to select another export
format, `--output-dir` to choose a different destination root, or
`--overwrite` to replace existing exports.

---

## 📊 Evaluation & Benchmark Results
No quantitative results are claimed for the Roboflow exports yet. Evaluate each
task against its matching held-out split and report those measurements before
using them as benchmark results.

---

## 👥 Team Split & Credits
- **Atiqua**: Frontend Lead, UI/UX, Camera integration, Test Data curation & Demo Video.
- **Ammar**: Backend Lead, Pipeline Orchestrator, REST API, Deck & Submission lead.
- **Muavia**: Vision & ML Lead, OpenCV cleanup, PyTorch detector fine-tuning, Graph builder.
- **Awais**: Language & RAG Lead, Grounded prompt engineering, Quiz generation, Quality review.

---
*Created for Banana Hacks 2026. All code and datasets created during the hackathon period (Oct 9–12, 2026).*
