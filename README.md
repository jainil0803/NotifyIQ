# 🌌 NOTIFYIQ: Intelligent User Notification System

> **Three-stage AI pipeline** that orchestrates behavioral segmentation → message generation → personalized delivery

[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue?style=flat-square&logo=python)](https://www.python.org/)
[![LLM: llama3.2](https://img.shields.io/badge/LLM-llama3.2-purple?style=flat-square)](https://ollama.com/)
[![Ollama](https://img.shields.io/badge/Powered%20by-Ollama-orange?style=flat-square)](https://ollama.com/)
[![LangChain](https://img.shields.io/badge/LangChain-Enabled-green?style=flat-square)](https://langchain.com/)

---

## 📁 Project Structure

```
aurora_final/
├── README.md
├── experiment_results.csv              # A/B test results used for iteration learning
├── codebase/
│   ├── .env                            # Ollama connection config (web vs local)
│   ├── instructions.txt
│   ├── task1.ipynb                     # Stage 1: Segmentation & KB extraction
│   ├── task2_timewondow.ipynb          # Stage 2: Templates & timing optimization
│   ├── task3_users_notifications_generation.ipynb  # Stage 3: Scheduling & learning
│   ├── input/
│   │   ├── users.csv                   # Raw user data
│   │   ├── KB.md                       # Company knowledge base
│   │   ├── KB_v2.md                    # Updated knowledge base
│   │   └── KB.pdf
│   ├── prompts/
│   │   └── prompts.json                # LLM prompt templates
│   └── output/                         # All generated artifacts
│       ├── users_imputed.csv
│       ├── users_propensity_generated.csv
│       ├── users_segmented.csv
│       ├── users_frequency_assigned.csv
│       ├── company_north_star.json
│       ├── allowed_tone_hook_matrix.json
│       ├── feature_goal_map.json
│       ├── segment_lifecycle_goals.csv
│       ├── segment_themes.json
│       ├── communication_themes.csv
│       ├── message_templates.csv
│       ├── message_templates_scored.csv
│       ├── message_templates_weighted_pool.csv
│       ├── timing_recommendations.csv
│       ├── user_schedules_final.csv            # Iteration 0 schedule
│       ├── user_schedules_final_iter2.csv      # Iteration 1 schedule
│       └── learning_delta_report.csv
├── iteration_0_before_learning/        # Snapshot before experiment feedback
│   ├── user_segments.csv
│   ├── segment_goals.csv
│   ├── company_north_star.json
│   ├── allowed_tone_hook_matrix.json
│   ├── feature_goal_map.json
│   ├── communication_themes.csv
│   ├── message_templates.csv
│   ├── timing_recommendations.csv
│   └── user_notification_schedule.csv
└── iteration_1_after_learning/         # Updated outputs after experiment feedback
    ├── message_templates.csv
    ├── timing_recommendations.csv
    ├── user_notification_schedule.csv
    └── learning_delta_report.csv
```

---

## 🏗️ Architecture Overview

```
users.csv + KB.md
    │
    ▼
┌─────────────────────────────────────────────┐
│  STAGE 1 — Segmentation & Profiling         │
│  Imputation → Propensity Scores → PCA       │
│  → Rule-based Segments (S1–S9)              │
│  → RAG + LLM KB Extraction                 │
└───────────────────┬─────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│  STAGE 2 — Message & Timing Optimization    │
│  Tone/Hook Assignment → Template Generation │
│  → GMM Timing Windows → Frequency Limits   │
└───────────────────┬─────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│  STAGE 3 — Scheduling & Learning            │
│  Iteration 0 Schedule → A/B Experiment      │
│  → Epsilon-Greedy Reweighting               │
│  → Iteration 1 Schedule + Delta Report      │
└─────────────────────────────────────────────┘
```

---

## 🔬 Stage Details

### Stage 1 — User Segmentation & Profiling (`task1.ipynb`)

| Input | Output |
|-------|--------|
| `input/users.csv` | `users_imputed.csv` |
| `input/KB.md` | `users_propensity_generated.csv` |
| | `users_segmented.csv` |
| | `company_north_star.json` |
| | `allowed_tone_hook_matrix.json` |
| | `feature_goal_map.json` |
| | `segment_lifecycle_goals.csv` |
| | `segment_themes.json` |

**Processing steps:**
- 🔧 Missing values imputed (numeric column means)
- 📈 Sigmoid-normalized propensity scores: `gamification`, `ai_tutor`, `leaderboard`
- 🧮 PCA-derived `activation_score` (sessions, exercises, streak, notif\_open\_rate, motivation)
- 🏷️ Rule-based segmentation: `dominant_propensity × activation_level` → **9 named segments (S1–S9)**
- 🔍 RAG pipeline: chunks KB → Chroma vector store → retrieves context per extraction
- 🤖 LLM (llama3.2) extracts: north star metric, tone/hook matrix, feature-goal mappings
- 🎮 LLM generates Octalysis theme profiles and segment-level goals for all lifecycle stages

---

### Stage 2 — Message & Timing Optimization (`task2_timewondow.ipynb`)

#### 2a. Tone/Hook Assignment
- LLM selects **2–3 tones + 1–2 hooks** per segment based on Octalysis theme
- Output: `communication_themes.csv`

#### 2b. Template Generation
- Generates **5 push notification templates** per `(segment × lifecycle stage × Octalysis drive)`
- Structured output: English + Hindi content + psychological hook + feature reference
- Output: `message_templates.csv`

#### 2c. Timing Window Optimization
- GMM fits user `preferred_hour` distribution per segment (max 3 clusters, BIC-selected)
- Maps cluster means to **6 time windows:**

| Window | Time Range |
|--------|-----------|
| `early_morning` | Dawn hours |
| `mid_morning` | Mid-morning |
| `afternoon` | Afternoon |
| `late_afternoon` | Late afternoon |
| `evening` | Evening |
| `night` | Night |

- Calculates expected CTR + engagement\_score per window
- Output: `timing_recommendations.csv`

#### 2d. User Frequency Assignment

| Activation Score | Daily Notification Limit |
|-----------------|--------------------------|
| > 0.7 | 8 notifications/day |
| 0.4 – 0.7 | 5 notifications/day |
| < 0.4 | 3 notifications/day |

- Output: `users_frequency_assigned.csv`

---

### Stage 3 — Notification Scheduling & Learning (`task3_users_notifications_generation.ipynb`)

#### 3a. Iteration 0 — Schedule Generation
- Merges users, timing windows, and templates into a **wide-format daily schedule**
- Assigns up to **9 notification slots** per user (time, template\_id, channel)
- Output: `user_schedules_final.csv`

#### 3b. Iteration 1 — Experiment-Informed Update

| Template Rating | Weight |
|----------------|--------|
| ✅ GOOD | 100 |
| ➖ NEUTRAL | 1 |
| ❌ BAD | 0 (suppressed) |

- Reads `experiment_results.csv` to score templates with **epsilon-greedy weighting**
- Updates timing windows via composite score:
  ```
  score = 0.5×CTR + 0.3×engagement - 0.2×uninstall
  ```
  with softmax redistribution (`T=0.05`) and iterative pruning of low-share windows
- Caps daily notifications to **2/day** for segments with uninstall rate > 2%
- Output: `message_templates_scored.csv`, `message_templates_weighted_pool.csv`, `user_schedules_final_iter2.csv`

#### 3c. Delta Reporter
- Compares Iteration 0 vs Iteration 1 across templates, timing windows, and frequency limits
- Logs entity-level changes with metric triggers and before/after values
- Output: `learning_delta_report.csv`

---

## 🤖 Key Models

| Component | Model / Library | Usage |
|-----------|----------------|-------|
| **LLM** | ChatOllama (llama3.2) | KB extraction, Octalysis inference, tone/hook selection, template generation |
| **Clustering** | Scikit-learn `GaussianMixture` (BIC) | Timing window detection per segment |
| **Dimensionality Reduction** | Scikit-learn `PCA` | Activation score derivation |
| **Vector Store** | ChromaDB | RAG pipeline for KB retrieval |

**LLM Temperature settings:**
- `0.0` — Deterministic extraction (KB parsing, segmentation)
- `0.3` — Creative diversity (template generation)

---

## ⚙️ Configuration

### `.env` File (`codebase/.env`)

```env
OLLAMA_MODE=web          # "web" uses ngrok URL, "local" uses 127.0.0.1:11434
OLLAMA_LOCAL_URL=http://127.0.0.1:11434
OLLAMA_WEB_URL=https://your-ngrok-url.ngrok-free.dev
```

### Key Hyperparameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `SOFTMAX_TEMPERATURE` | `0.05` | Controls exploitation vs. exploration in timing window selection |
| `UNINSTALL_RATE_THRESHOLD` | `0.02` | Triggers notification cap for high-churn segments |
| `max_windows` | `3` | GMM max clusters for timing granularity |
| LLM `temperature` (extraction) | `0` | Deterministic output |
| LLM `temperature` (generation) | `0.3` | Template diversity |

---

## 🚀 Getting Started

### Prerequisites

```bash
pip install pandas numpy scikit-learn langchain langchain-ollama \
            langchain-community langchain-text-splitters langgraph \
            pydantic chromadb python-dotenv
```

> Also requires [Ollama](https://ollama.com/) accessible locally or via ngrok tunnel.

### Pull the LLM model

```bash
ollama pull llama3.2:latest
```

### Required Input Files

```
codebase/input/users.csv
codebase/input/KB.md
codebase/prompts/prompts.json
```

### Execution Order

```bash
# Step 1: Configure your Ollama connection
# Edit codebase/.env → set OLLAMA_MODE to "web" or "local"

# Step 2: Run Stage 1 — Segmentation & KB extraction
#         → jupyter notebook codebase/task1.ipynb

# Step 3: Run Stage 2 — Templates & timing optimization
#         → jupyter notebook codebase/task2_timewondow.ipynb

# Step 4: Run Stage 3 — Scheduling & learning loop
#         → jupyter notebook codebase/task3_users_notifications_generation.ipynb
```

All outputs are written to `codebase/output/`.  
Iteration snapshots are saved to `iteration_0_before_learning/` and `iteration_1_after_learning/`.

---

## 📊 Iteration 1 Note

> **`experiment_results.csv` is required for Iteration 1.**

Since A/B test feedback data is not included in this submission, the `iteration_1_after_learning/` folder is intentionally left empty. The code is **fully functional** — to run Iteration 1:

1. Place your `experiment_results.csv` in the project root directory
2. Re-run `task3_users_notifications_generation.ipynb`

The pipeline will automatically:
- Apply epsilon-greedy template weighting
- Update timing windows with composite scoring
- Populate `iteration_1_after_learning/` with the updated schedule and delta report

---

## 📦 GitHub Repository

🔗 [https://github.com/jainil0803/Aurora-project](https://github.com/jainil0803/Aurora-project)
