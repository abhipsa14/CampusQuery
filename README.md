# 🎓 CampusQuery - Rag-based Chatbot

An AI-powered **voice assistant** build for UIT (United Institute of Technology) that works like Google Assistant — always listening for a wake word, answers questions using a custom knowledge base, and leverages **Ollama (llama3.2)** for versatile local AI responses.

## ✨ Features

| Feature | Description |
|---|---|
| **Wake Word Activation** | Say **"Hey Assistant"**, **"Hey Computer"**, or **"OK Assistant"** to activate |
| **Custom Knowledge Base** | Answers college-specific questions from `UIT Data Set.docx` and web sources |
| **Local LLM (Ollama)** | Uses Ollama with llama3.2 for private, offline AI responses |
| **Hybrid RAG Pipeline** | Automatically routes queries — college data via RAG, everything else via Ollama |
| **System Tray App** | Runs silently in the background with a tray icon (like Cortana/Google Assistant) |
| **Offline TTS** | Speaks responses aloud using `pyttsx3` (no internet needed for speech output) |
| **Console Mode** | Text-based mode for testing without a microphone |
| **Raspberry Pi 4** | Runs as an always-on systemd service — auto-starts on boot |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────┐
│                   System Tray App                    │
│              (tray_app.py / main.py)                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────┐    ┌──────────────┐    ┌──────────┐   │
│  │ Wake Word│───▶│ Speech-to-   │───▶│ Hybrid   │   │
│  │ Detector │    │ Text (STT)   │    │ Pipeline │   │
│  └──────────┘    └──────────────┘    └────┬─────┘   │
│                                           │         │
│                    ┌──────────────────────┤         │
│                    ▼                      ▼         │
│             ┌──────────┐          ┌──────────┐     │
│             │ RAG +    │          │  Ollama  │     │
│             │ ChromaDB │          │ (General)│     │
│             └──────────┘          └──────────┘     │
│                    │                      │         │
│                    └──────────┬───────────┘         │
│                               ▼                     │
│                      ┌──────────────┐               │
│                      │ Text-to-     │               │
│                      │ Speech (TTS) │               │
│                      └──────────────┘               │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 📂 Project Structure

```
cllg_chatbot/
├── main.py                    # Main entry point
├── assistant.py               # Core voice assistant logic
├── tray_app.py                # System tray application (Windows)
├── config.py                  # Central configuration (auto-detects Pi)
├── UIT Data Set.docx          # Primary knowledge base 
│
├── pi/                        # Raspberry Pi 4 files
│   ├── install_pi.sh          # One-command full Pi setup
│   ├── uit-assistant.service  # systemd service (auto-start on boot)
│   ├── ollama.service         # Ollama systemd service
│   ├── asoundrc               # ALSA audio config for USB mic
│   └── test_audio.sh          # Test speaker & mic on Pi
│
├── voice/                     # Voice I/O modules
│   ├── speech_to_text.py      # Microphone → text
│   ├── text_to_speech.py      # Text → speech
│   └── wake_word.py           # Wake word detection
│
├── rag/                       # AI pipeline
│   └── hybrid_pipeline.py     # RAG + Ollama router
│
├── llm/                       # LLM clients
│   └── ollama_client.py       # Local Ollama (llama3.2)
│
├── ingestion/                 # Data processing
│   ├── run_ingestion.py       # Full pipeline
│   ├── pdf_ingestor.py        # PDF/DOCX/TXT reader
│   ├── web_ingestor.py        # Web scraper
│   ├── normalizer.py          # Text cleanup
│   ├── chunker.py             # Text chunking
│   └── embedder.py            # Sentence embeddings
│
├── vector_db/                 # Vector store
│   ├── chroma_client.py       # ChromaDB client
│   ├── indexer.py             # Chunk indexer
│   └── retriever.py           # Similarity search
│
└── data/                      # Processed data
    ├── raw_pdfs/
    ├── extracted_text/
    ├── normalized_text/
    └── processed_chunks/
```

---

## 🚀 Quick Start

### 1. Install Dependencies

```bash
pip install -r requirements.txt
```

> **Note:** On Windows, `PyAudio` may need a pre-built wheel:
> ```bash
> pip install pipwin
> pipwin install pyaudio
> ```

### 2. Install & Start Ollama

Download from [ollama.com](https://ollama.com/) and pull the model:

```bash
ollama pull llama3.2
ollama serve    # keep running in background
```

### 3. Ingest Knowledge Base

Process `UIT Data Set.docx` and web sources into the vector database:

```bash
python main.py --ingest
```

### 4. Start the Assistant

```bash
# System tray mode (recommended — runs in background)
python main.py

# Voice-only mode (no tray icon)
python main.py --voice

# Console/text mode (for testing without microphone)
python main.py --console
```

---

## How to use it?

1. **Start the app** — it runs in the system tray
2. **Say "Hey Assistant"** — the assistant activates
3. **Ask your question** — speak naturally
4. **Listen to the response** — the assistant speaks the answer
5. **Follow-up** — keep asking or stay silent to deactivate

### Example Queries

- *"What programs does UIT offer?"* → Answered from knowledge base
- *"What is the code of conduct?"* → Answered from knowledge base
- *"Tell me about the library"* → Answered from knowledge base
- *"What is quantum computing?"* → Answered by Ollama
- *"Write me a poem about college"* → Answered by Ollama
- *"What time is it?"* → Handled by system

---

## ⚙️ Configuration

Edit `config.py` to customize:

| Setting | Default | Description |
|---|---|---|
| `WAKE_WORDS` | `["hey assistant", "hey computer", "ok assistant"]` | Trigger phrases |
| `OLLAMA_MODEL` | `llama3.2` | Ollama model to use |
| `TOP_K` | `5` | Number of RAG chunks to retrieve |
| `TTS_RATE` | `175` | Speech speed (words/min) |
| `CHUNK_SIZE` | `500` | Words per chunk |

---

## 📋 Requirements

- Python 3.10+
- Microphone (for voice mode)
- [Ollama](https://ollama.com/) installed and running
- Internet connection (for Google STT — optional if using offline STT)

---

## Raspberry Pi 4/5 Deployment

The assistant is fully compatible with **Raspberry Pi 4/5 ** and runs as an always-on system service just like a smart speaker.

### One-Command Install

```bash
# 1. Copy project to your Pi (via SCP, USB, or git clone)
cd /home/pi
git clone <your-repo-url> CampusQuery  # or copy the folder

# 2. Run the installer (does everything automatically)
cd CampusQuery
chmod +x pi/install_pi.sh
sudo ./pi/install_pi.sh
```

This single script will:
1. Install all system packages (Python, PortAudio, espeak, ALSA)
2. Configure audio (USB mic + headphone jack)
3. Install Ollama and pull the llama3.2 model
4. Set up Python virtual environment and install dependencies
5. Ingest the UIT knowledge base
6. Install and start **systemd services** for both Ollama and the assistant
7. Enable auto-start on every boot

### After Installation

The assistant **starts automatically** on boot. Just say **"Hey Assistant"**!

```bash
# Check status
sudo systemctl status uit-assistant

# View live logs
journalctl -u uit-assistant -f

# Restart
sudo systemctl restart uit-assistant

# Stop
sudo systemctl stop uit-assistant

# Disable auto-start
sudo systemctl disable uit-assistant
```

### You can Test it on Pi also.

Before running, verify your mic and speaker work:

```bash
chmod +x pi/test_audio.sh
./pi/test_audio.sh
```

### Pi-Specific Notes
- **Audio**: Default config uses USB mic for input and 3.5mm jack for output. Edit `pi/asoundrc` if your setup differs. Run `arecord -l` to find your mic's card number.
- **Model**: llama3.2 (3B) runs on Pi 4 with 4GB RAM. For faster responses, consider `tinyllama` or `phi`.
- **Memory**: Close other apps. The LLM + embeddings use most of the RAM.
- **First boot**: The first query takes longer as models load into memory. Subsequent queries are faster.
