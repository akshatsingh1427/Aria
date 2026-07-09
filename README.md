<div align="center">

# 🎙️ ARIA

### Adaptive Real-time Interview Agent

<img src="https://img.shields.io/badge/Domain-Artificial%20Intelligence-2563EB?style=for-the-badge">
<img src="https://img.shields.io/badge/Category-HR%20Tech-EA580C?style=for-the-badge">
<img src="https://img.shields.io/badge/Type-Multimodal%20AI-16A34A?style=for-the-badge">
<img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge">

**AureetureAI × Technex 2026 | IIT-BHU**

> A fully browser-based, AI-native interview system that conducts intelligent, adaptive interviews in real time — evaluating technical skills, communication, and behavioral traits with explainable, bias-aware scoring.

</div>

---

## Table of Contents

- [What ARIA Does](#-what-aria-does)
- [Architecture](#️-architecture)
- [Models Used](#-models-used)
- [Inference Flow](#-inference-flow)
- [Scoring System](#-scoring-system)
- [Bias Mitigation](#-bias-mitigation)
- [Role-Specific Design](#-role-specific-design)
- [Scalability Design](#-scalability-design)
- [Tech Stack](#️-tech-stack)
- [How to Run](#️-how-to-run)
- [Project Structure](#-project-structure)
- [Key Technical Decisions](#-key-technical-decisions)
- [Team](#-team)

---

## 🚀 What ARIA Does

ARIA is a single-file, production-grade AI interview agent that:

- Conducts **role-specific adaptive interviews** across 6 domains — Software Engineer, Data Scientist, ML Engineer, Product Manager, Data Analyst, Data Engineer
- **Dynamically generates questions** using LLaMA 3.3 70B via Groq — adapting difficulty and topic based on each answer
- **Evaluates in real-time** across 5 dimensions with live score bars
- **Analyzes multimodal signals** — webcam engagement, eye contact, speech rate, filler word detection
- Produces a **fully explainable final scorecard** with per-dimension evidence, reasoning, and a bias audit

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│              MULTIMODAL INPUT LAYER             │
│   Webcam (WebRTC) · Microphone · Text Input     │
└──────────┬────────────────┬─────────────────────┘
           │                │
  ┌────────▼──────┐  ┌──────▼────────────────────┐
  │ Vision Module │  │     Speech + NLP Module   │
  │ MediaPipe     │  │  Web Speech API (STT)     │
  │ Eye contact   │  │  WPM · Filler rate        │
  │ Engagement    │  │  Answer text analysis     │
  └────────┬──────┘  └──────┬────────────────────┘
           └────────┬────────┘
                    │
     ┌──────────────▼──────────────────────┐
     │     ADAPTIVE INTERVIEW BRAIN        │
     │   Groq API · LLaMA 3.3 70B          │
     │                                     │
     │  • Role-aware system prompt         │
     │  • Per-role forbidden topic lists   │
     │  • Conversation history context     │
     │  • Quality signal: strong/adequate/ │
     │    weak/unclear per answer          │
     │  • Dynamic: drill deeper if weak,   │
     │    advance difficulty if strong     │
     └──────────────┬──────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
┌───────▼──────┐     ┌──────────▼──────────┐
│  Live Scores │     │  Final Report Engine│
│  5 dimensions│     │  Groq LLaMA scoring │
│  Rolling avg │     │  Evidence per dim   │
│  Hire signal │     │  Bias audit field   │
└──────────────┘     └─────────────────────┘
```

---

## 🤖 Models Used

| Component | Model | Provider | Cost |
|---|---|---|---|
| Interview Brain | LLaMA 3.3 70B Versatile | Groq | Free |
| Final Scorecard | LLaMA 3.3 70B Versatile | Groq | Free |
| Speech-to-Text | Web Speech API | Browser native | Free |
| Face Engagement | MediaPipe (simulated signals) | Browser native | Free |

**Why Groq + LLaMA 3.3 70B:**

- ~300 tokens/second inference — near real-time question generation
- OpenAI-compatible API, JSON mode supported
- Completely free tier — no credit card required
- 70B parameter model gives strong reasoning for answer quality assessment

---

## 🔄 Inference Flow

```
1. Candidate submits answer
        │
2. Compute audio signals
   └── WPM = word_count / elapsed_minutes
   └── filler_rate = filler_words / total_words
        │
3. Build role-aware prompt
   └── Inject: role title, skills, forbidden topics, focus area
   └── Include: full conversation history (last 8 turns)
   └── Include: face engagement %, eye contact %, WPM, filler rate
        │
4. Groq API call → LLaMA 3.3 70B
   └── response_format: json_object (guaranteed JSON)
   └── Returns: question, skill_tested, difficulty,
               question_type, quality_signal, analysis
   └── Candidate messages matching "no"/"idk"/"skip"/"pass" (whole answer)
       are flagged as quality_signal: skipped before the call — the model
       is told to acknowledge and move on, never press or lecture
        │
5. Parse & validate JSON response
   └── 3-layer fallback parser (direct → regex → field extract)
        │
6. Update live scores (rolling average)
   └── technical, communication, problem_solving,
       behavioral, confidence
        │
7. Display next question with analysis chip
   └── ✓ Strong / ✓ Good / ↺ Needs depth / ? Unclear / ⊘ Skipped
        │
8. Repeat for 7 questions
        │
9. Generate final report
   └── Second Groq call with full transcript
   └── Returns per-dimension scores with evidence quotes
   └── Bias audit field explicitly checked
```

---

## 📊 Scoring System

### 5 Evaluation Dimensions

| Dimension | What It Measures | Signals Used |
|---|---|---|
| Technical depth | Domain knowledge, accuracy, depth of answer | LLM quality signal + question difficulty |
| Communication | Clarity, structure, conciseness | Eye contact ratio, filler word rate, answer structure |
| Problem solving | Logical approach, handling ambiguity | Quality signal on hard questions |
| Behavioural | Self-awareness, growth mindset, teamwork | Quality signal on behavioral questions |
| Confidence | Consistency, directness | Face engagement + eye contact scores |

### Score Calculation

```
base_score = { strong: 80, adequate: 62, weak: 38, unclear: 44, skipped: 6 }
difficulty_bonus = { easy: 0, medium: +5, hard: +10 }

technical = base + (difficulty_bonus if technical question)
communication = base + (eye_contact × 12) - (filler_rate × 0.4)
problem_solving = base + 12 bonus if (strong + hard question)
behavioral = base + 14 bonus if (strong + behavioral question)
confidence = engagement × 70 + eye_contact × 25

final_score = rolling_average(all answers, per dimension)
overall = mean(5 dimensions)
```

**Handling declined answers:** A candidate typing "no", "idk", "skip", "pass", "not sure," etc. as their *entire* answer is detected client-side (`isSkipAnswer()`) and flagged as `quality_signal: "skipped"` — separately from a genuine but weak attempt. This does two things:

- The interview brain is instructed not to press the candidate on why they don't know something, and not to re-probe the same topic — it simply acknowledges briefly and moves to a new skill area.
- Scoring drops to a realistic floor (base 6, no randomness, no minimum-15 safety floor) instead of the 38–44 baseline a weak/unclear answer gets. Answering "no" to every question now trends the overall score toward 0, not ~39.

### Hire Signal Thresholds

```
overall ≥ 75  →  Strong Hire
overall ≥ 62  →  Hire
overall ≥ 48  →  Maybe
overall < 48  →  No Hire
```

---

## ⚖️ Bias Mitigation

ARIA is designed to score fairly regardless of accent, speech style, or cultural communication patterns:

1. **Content over delivery** — LLM quality signal assesses answer correctness, not how it sounds
2. **Filler word tolerance** — minor filler rates have minimal score impact; only extreme rates (>30%) affect communication score by at most a few points
3. **No accent penalty** — Web Speech API transcription errors affect all candidates equally; scoring is on transcribed content
4. **Explicit bias audit** — every final report includes a `bias_check` field where the LLM audits its own scoring for demographic proxies
5. **Role-specific forbidden topics** — prevents irrelevant technical questions that disadvantage candidates from non-CS backgrounds (e.g. Data Scientists never get DSA questions)

---

## 🎯 Role-Specific Design

Each of the 6 roles has a dedicated profile:

| Role | Key Skills Tested | Forbidden Topics |
|---|---|---|
| Data Scientist | Statistics, ML theory, model evaluation, A/B testing, feature engineering | DSA, algorithms, LeetCode, system design |
| Software Engineer | Data structures, algorithms, system design, code quality | None |
| ML Engineer | MLOps, deployment, pipelines, model optimization | Pure SWE, pure data science |
| Product Manager | Product thinking, metrics, prioritization, user empathy | DSA, coding, system design |
| Data Analyst | SQL, data visualization, BI tools, KPI definition | DSA, ML engineering, coding |
| Data Engineer | ETL pipelines, data warehousing, Spark, Kafka, Airflow | ML theory, pure DSA |

---

## 📈 Scalability Design

ARIA is architected for real-world scale:

**Current (single-file browser app):**

- Zero backend — runs entirely in the browser
- Each interview is a stateless WebSocket-equivalent (fetch-based) session
- No shared state between sessions
- Works on any device with a modern browser

**Production scale path:**

```
Load Balancer
    ├── Browser Client 1  ──→  Groq API (stateless)
    ├── Browser Client 2  ──→  Groq API (stateless)
    └── Browser Client N  ──→  Groq API (stateless)
```

- No server needed for core interview functionality
- Session state lives in browser memory
- 10,000 concurrent interviews = 10,000 independent browser tabs, each making direct Groq API calls
- Groq's rate limits are per API key — production would use a proxy layer to pool multiple keys

---

## 🛠️ Tech Stack

<div align="center">

<img src="https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white">
<img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white">
<img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black">
<img src="https://img.shields.io/badge/Groq-00D4AA?style=for-the-badge">
<img src="https://img.shields.io/badge/LLaMA%203.3%2070B-7C3AED?style=for-the-badge">

</div>

| Layer | Technology |
|---|---|
| LLM Inference | Groq Cloud API |
| Language Model | Meta LLaMA 3.3 70B Versatile |
| Speech-to-Text | Web Speech API (Chrome) |
| Face/Engagement | MediaPipe FaceMesh signals |
| Frontend | Vanilla HTML + CSS + JavaScript |
| Fonts | Syne, JetBrains Mono, Lora (Google Fonts) |
| Build Step | None — single `.html` file, open directly in browser |

---

## ▶️ How to Run

> **Requirements:** Chrome browser (for Web Speech API), free Groq API key

```
1. Get free Groq key → https://console.groq.com
2. Open index.html in Chrome
3. Enter your Groq key on the first screen
4. Enter your name, select role, click Start Interview
5. Answer questions in the text box or use the mic button
6. Press Enter or click Send after each answer
7. After 7 questions, view your full evaluation report
```

---

## 📁 Project Structure

```
ARIA/
├── index.html      ← Complete application (HTML + CSS + JS)
├── README.md        ← This file
└── .env.example     ← GROQ_API_KEY=gsk_your_key_here
```

---

## 💡 Key Technical Decisions

**Why a single HTML file?**
The challenge asks for a working demo. A single file means zero setup, zero dependencies, zero deployment — judges can open it instantly. The entire system, including UI, interview logic, scoring engine, and report generation, is self-contained.

**Why Groq over other APIs?**
Groq delivers ~300 tokens/second, which makes the interview feel natural — no waiting 10 seconds for the next question. Combined with the free tier, it's the only API that enables a real-time interview experience at zero cost.

**Why LLaMA 3.3 70B over smaller models?**
Interview quality assessment requires understanding nuance — a candidate who says they'd use precision and recall is stronger than one who says they'd check accuracy. Smaller models (7B, 13B) frequently miss these distinctions. 70B gets it right consistently.

**Why role-specific forbidden topic lists?**
In testing, without explicit forbidden lists, the model would ask Data Scientists about binary trees and Product Managers about time complexity. The forbidden list is injected into every single API call, making topic drift impossible.

---

## 👥 Team

Built for **AureetureAI × Technex 2026 | IIT-BHU**
Submission via Unstop
