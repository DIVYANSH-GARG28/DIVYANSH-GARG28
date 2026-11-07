# APEX CONNECT AI — Voice-Synthesized Autonomous Calendar Agent

> **Production-grade conversational scheduling agent integrating open-weights 70B LLMs with browser-native speech synthesis and deterministic calendar collision resolution.**

[![Status](https://img.shields.io/badge/Status-Operational%20Prototype-00D9FF?style=flat-square)](#)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=flat-square&logo=python)](#)
[![Model](https://img.shields.io/badge/LLM-Qwen2--72B--Instruct-purple?style=flat-square)](#)
[![Backend](https://img.shields.io/badge/Framework-Flask-black?style=flat-square&logo=flask)](#)

---

## 1. System Overview & Problem Statement

Modern scheduling systems require users to context-switch across complex web calendar interfaces, cross-reference timezones manually, and manually resolve overlapping commitments. While commercial voice assistants exist, they generally rely on closed proprietary APIs with rigid intent slot-filling that breaks during dynamic multi-turn negotiations.

**APEX CONNECT AI** demonstrates that state-of-the-art open-weights language models (`Qwen2-72B-Instruct`) can be orchestrated directly from a lightweight web application to conduct dynamic voice interactions, parse unstructured temporal intent into structured JSON tool calls, and execute deterministic scheduling actions.

---

## 2. Technical Architecture

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                           APEX CONNECT AI PIPELINE                          │
└─────────────────────────────────────────────────────────────────────────────┘

    [ Browser Client ]
           │
           ├─ (1) Microphone Audio Stream ────────► [ Web Speech API (Recognition) ]
           │                                                      │
           │                                          (Transcribed Utterance)
           │                                                      ▼
           ├─ (2) HTTP POST /chat { utterance } ──► [ Flask Gateway API ]
           │                                                      │
           │                                        (System Prompt + History)
           │                                                      ▼
           │                                        [ Featherless AI API ]
           │                                        (Qwen2-72B-Instruct)
           │                                                      │
           │                                        (Structured Tool Call JSON)
           │                                                      ▼
           ├─ (3) Parse Tool Intent ──────────────► [ Calendar Tool Engine ]
           │                                                      │
           │                                          [ Collision Detection ]
           │                                          ├── No Conflict: Commit
           │                                          └── Conflict: Propose Alt
           │                                                      │
           ├─ (4) HTTP Response { reply, action } ◄───────────────┘
           │
           └─ (5) Browser Audio Engine ───────────► [ Web Speech API (Synthesis) ]
```

---

## 3. Core Engineering Modules

### A. Dynamic Open-Weights Inference (`Qwen2-72B-Instruct`)
The backend interfaces with `Qwen2-72B-Instruct` hosted on high-throughput serverless infrastructure (Featherless AI). The system prompt enforces strict operational constraints:
- Extract event metadata: `title`, `start_time`, `end_time`, `participants`, `priority`.
- Emit tool-calling commands in a deterministic JSON envelope:
  ```json
  {
    "action": "schedule_event",
    "parameters": {
      "title": "Systems Architecture Review",
      "timestamp": "2026-11-08T15:00:00Z",
      "duration_minutes": 45
    },
    "voice_response": "Checking your schedule for the Systems Architecture Review at 3 PM tomorrow."
  }
  ```

### B. Autonomous Collision Resolution
When a requested time window overlaps with an existing commitment, the calendar engine calculates the minimum required shift:
1. Scan for nearest free window of equal or greater duration within working hours.
2. Formulate a counter-proposal returned to the LLM context.
3. Inform the user conversationally through synthesized speech without discarding context.

### C. Zero-Dependency Audio Pipeline
Rather than relying on paid third-party voice APIs (e.g. ElevenLabs), APEX CONNECT AI utilizes the browser's native `window.webkitSpeechRecognition` and `window.speechSynthesis` APIs, cutting speech round-trip latency to near zero and eliminating per-word API billing.

---

## 4. Repository Structure

```text
APEX_CONNECT_AI/
├── app.py                  # Core Flask server and endpoint routing
├── config.py               # Environment configuration and API keys
├── static/
│   ├── css/style.css       # Midnight Engineering interface styling
│   └── js/app.js           # Web Speech API bindings and DOM updates
├── templates/
│   └── index.html          # Interactive agent dashboard
├── requirements.txt        # Python dependency manifest
└── README.md               # Technical overview
```

---

## 5. Local Setup & Execution

### Prerequisites
- Python 3.9 or higher
- Modern Chromium-based browser (Chrome, Edge, Brave) with microphone permissions enabled
- Featherless AI API key (or compatible OpenAI-format inference endpoint)

### Installation
```bash
# Clone the repository
git clone https://github.com/DIVYANSH-GARG28/APEX_CONNECT_AI.git
cd APEX_CONNECT_AI

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment variables
export FEATHERLESS_API_KEY="your_api_key_here"
# On Windows PowerShell:
# $env:FEATHERLESS_API_KEY="your_api_key_here"

# Launch Flask development server
python app.py
```

Navigate to `http://localhost:5000` in your browser. Click the microphone activation button and begin speaking.

---

## 6. Planned Technical Enhancements
- [ ] Integration with Google Calendar API (OAuth 2.0 refresh tokens).
- [ ] Local streaming audio via Whisper.cpp running on local hardware.
- [ ] Multi-agent negotiation for coordinating schedules across multiple users.
