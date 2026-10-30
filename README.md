# EduCards 🎓

EduCards is a collaborative, AI-powered study assistant designed to help students and educators practice multiple-choice questions (MCQs), ingest syllabus materials for Retrieval-Augmented Generation (RAG), and receive adaptive AI explanations for incorrect answers.

---

## 🌟 Key Features

1. **MCQ Study Deck Generator:** Generate four-option questions on any topic using Google Gemini or choose from seeded subject decks.
2. **Syllabus RAG Ingestion:** Upload lecture notes, syllabi, or textbooks (PDF or TXT) to ground AI deck generation directly in your coursework.
3. **Interactive Practice Mode:** Practice MCQs with randomized options, server-recorded attempts, points, and streaks.
4. **Adaptive AI Explainer:** When you answer a question incorrectly, trigger the AI tutor to explain the concept tailored to your exact understanding level (ELI5, Beginner, Intermediate, or Advanced Expert).
5. **Learner Profiles and Leaderboard:** Track accuracy, points, streaks, completed decks, and topics to review.
6. **Learning Analytics:** Explore personal and aggregate trends, response times, activity, and question difficulty with generated charts.
7. **Session Overview:** Review strengths, weak areas, missed answers, hints used, and next steps after finishing a deck.
8. **Soft pastel UI:** A blush-pink, pale-blue, and white palette with rounded, lightweight surfaces, plus a persistent dark-theme option, keyboard shortcuts, and a circular progress indicator.

---

## 🛠️ Tech Stack

- **Backend:** Python 3.10+, FastAPI, Pydantic v2, Uvicorn, Google GenAI SDK (`google-genai`), PyPDF
- **Frontend:** HTML5, CSS3, Vanilla JavaScript (ES6+)
- **Analytics:** pandas, NumPy, Matplotlib (Agg/non-interactive rendering)
- **Storage:** PostgreSQL when `DATABASE_URL` is configured; otherwise SQLite (`data/educards.sqlite3` by default), with imports from legacy JSON and optional one-time SQLite-to-PostgreSQL migration
- **Testing:** Pytest, HTTPX

---

## 📁 Project Structure

```text
ps_2_gfg/
├── backend/
│   ├── analytics.py      # Aggregated analytics and chart generation
│   ├── ai_service.py      # Gemini AI client & prompt engineering
│   ├── main.py            # FastAPI application & API endpoints
│   ├── models.py          # Pydantic data models (Deck, Card, Material)
│   ├── rag.py             # PDF/Text extraction & RAG chunking
│   ├── seed.py            # Idempotent subject deck seeding
│   ├── storage.py         # SQLite storage, legacy import, scoring
│   ├── requirements.txt   # Backend Python dependencies
│   └── test_*.py          # Unit and integration test suite
├── data/
│   └── decks.json         # Legacy deck import source
├── frontend/
│   ├── index.html         # Single-page application shell & modals
│   ├── style.css          # Solid color design system & UI styles
│   └── app.js             # Frontend router, state, and API client
└── README.md
```

---

## 🚀 Getting Started (Local)

### Requirements

- Python 3.10 or newer
- Internet access for the first dependency installation
- API keys are **optional**. Without them, the app uses built-in mock responses so you can demonstrate the project locally.

### Option A — easiest on Linux/macOS

From the project root directory:

```bash
chmod +x run.sh
./run.sh
```

The script creates a `.venv` virtual environment, installs dependencies, and starts the server. The first run may take a few minutes. Keep this terminal open while using the app.

### Option B — Windows

Double-click `run.bat`, or run it from Command Prompt in the project folder:

```bat
run.bat
```

### Option C — manual setup

Linux/macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r backend/requirements.txt
python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000 --reload
```

Windows Command Prompt:

```bat
py -3 -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
python -m pip install -r backend\requirements.txt
python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000 --reload
```

Open **http://127.0.0.1:8000**. API documentation is available at **http://127.0.0.1:8000/docs** and the health check is **http://127.0.0.1:8000/api/health**.

### Environment variables (optional)

| Variable | Required? | Used for |
|---|---|---|
| `GEMINI_API_KEY` | No | Real AI deck generation, explanations, and feedback. Without it, mock/fallback responses are used. |
| `GROQ_API_KEY` | No | Real AI mindmap generation. Without it, a mock mindmap is returned. |
| `DATABASE_URL` | No | PostgreSQL connection URL. When set, PostgreSQL is used instead of SQLite. |
| `POSTGRES_URL` | No | Alias used when `DATABASE_URL` is not set. |
| `EDUCART_DB_PATH` | No | SQLite file path; defaults to `data/educards.sqlite3` locally and `/tmp/educards.sqlite3` on Vercel/Lambda. Also used as the default SQLite source for PostgreSQL migration. |
| `EDUCART_SQLITE_MIGRATION_PATH` | No | Explicit SQLite database file to copy into PostgreSQL once. |

To configure keys, copy `.env.example` to `.env` and fill in your keys. `run.sh` and `run.bat` load `.env` when present. Never commit `.env` or share your API keys. You can run and present the project without either key.

Topic-specific AI deck generation requires a valid Gemini API key created in Google AI Studio. OAuth access tokens, service-account credentials, and placeholder strings are not accepted as `GEMINI_API_KEY`; restart the server after changing `.env`. Without a valid key, only supported built-in offline topics can generate decks.

### Data storage

Application data uses PostgreSQL when `DATABASE_URL` (or `POSTGRES_URL`) is set; otherwise it is stored in SQLite. Set `EDUCART_DB_PATH` to choose another SQLite file. When PostgreSQL is selected, startup copies any existing SQLite database at `EDUCART_SQLITE_MIGRATION_PATH` (or `EDUCART_DB_PATH`, or the default local database path) once; existing PostgreSQL rows are preserved on conflicts. Set a PostgreSQL connection URL before launching the app to use the hosted database.

On first startup, EduCards imports valid MCQs from the legacy `data/decks.json` file and materials from `data/materials.json`; legacy flashcards and invalid MCQs are discarded. The JSON files remain as import sources and are not used for ongoing writes. Run `python -m backend.seed` to safely seed the ten built-in subject decks.

Vercel's `/tmp` directory is ephemeral and is wiped on cold starts. SQLite data stored at `/tmp/educards.sqlite3` will not persist across cold starts; configure a hosted PostgreSQL database for persistent users, scores, decks, or materials.

During practice, press **1–4** to select an option and **Enter** to move on after answering. Use the header control to switch between the default pink-and-blue theme and dark mode; the preference is saved in localStorage.

Learners choose a username on first visit. The browser stores only the generated user ID in localStorage; there are no passwords. Usernames are unique without regard to capitalization.

The analytics module uses pandas for attempt-data preparation and grouped accuracy, hint-use, activity, and ranking calculations; NumPy for response-time percentiles, weak-card flags, point histograms, and learning-trend fitting; and Matplotlib's Agg backend for in-memory PNG chart rendering. Personal and global analytics responses are cached in memory for approximately 60 seconds. Global analytics exposes grouped metrics only, not learner identities, selected answers, or attempt-level records. See the [analytics implementation guide](backend/analytics.py) for the calculation and chart-generation details.

Correct answers earn 10 points plus a speed bonus of up to 5 points (full at 5 seconds or faster, declining linearly to 0 at 20 seconds), less 2 points per hint used. Five consecutive correct answers in one session activate a 1.5× multiplier. A repeat correct answer for the same card on the same UTC day is capped at 1 point; wrong answers earn 0.

### Running tests

Activate the virtual environment, then run:

```bash
python -m pytest backend/ -v
```

Tests that mock the Gemini or Groq SDK require the dependencies from `backend/requirements.txt` to be installed first.

---

## 🔌 API reference

The API is served by FastAPI. Interactive request/response schemas are available at
[`/docs`](http://127.0.0.1:8000/docs) while the app is running; the alternative
ReDoc interface is at [`/redoc`](http://127.0.0.1:8000/redoc). JSON endpoints
accept and return JSON unless an endpoint below is specifically marked as a file
upload. IDs in paths and request bodies are the IDs returned by the API.

### Health and decks

| Method and path | Purpose | Request / response notes |
|---|---|---|
| `GET /api/health` | Check that the API process is responding. | Returns `{"status":"healthy"}`. |
| `GET /api/decks` | List available seeded and saved decks. | Returns an array of decks and their MCQ cards. |
| `POST /api/decks` | Create an empty deck. | JSON: `title`, `description`, `category`, `creator`; optional `collaborators`. Returns the saved deck. |
| `GET /api/decks/{deck_id}` | Get one deck and its cards. | Returns `404` if the deck does not exist. |
| `DELETE /api/decks/{deck_id}` | Delete a deck. | Returns a success message or `404`. |
| `POST /api/decks/{deck_id}/cards` | Add a card to a deck. | JSON: `question`, `answer`, `options` (exactly four unique strings, including the exact answer), optional `explanation`, and `difficulty` (`easy`, `medium`, or `hard`). |
| `POST /api/decks/generate` | Generate and save an MCQ deck. | JSON: `topic`, optional `count` (1–50, default 5), optional `material_id`; `card_type` is accepted for older clients, but output is MCQ-only. Uses uploaded material as context where available. |
| `POST /api/decks/{deck_id}/mindmap` | Generate a mindmap for a saved deck. | Returns the AI-generated or offline fallback mindmap. |

Example deck-generation request:

```json
{
  "topic": "Python list comprehensions",
  "count": 10,
  "material_id": null
}
```

### Materials and learner identity

| Method and path | Purpose | Request / response notes |
|---|---|---|
| `GET /api/materials` | List uploaded study materials. | Returns material metadata and extracted text. |
| `POST /api/materials/upload` | Extract and store a PDF or TXT document for RAG. | `multipart/form-data` with a `file` field; returns its ID, filename, extracted content, and chunk count. |
| `GET /api/users` | List learner identities. | Optional query parameter `username` filters the list. |
| `POST /api/users` | Register a username-only learner. | JSON: `{"username":"Ada"}`. Returns the new user; a duplicate username returns `409`. |
| `GET /api/users/{user_id}` | Get one learner identity. | Returns `404` if not found. |
| `GET /api/users/{user_id}/profile` | Get a learner profile. | Includes total points, rank, streaks, accuracy by deck, weak topics, and recent sessions. |

Example material upload:

```bash
curl -F "file=@lecture-notes.pdf" http://127.0.0.1:8000/api/materials/upload
```

### Practice, scoring, and session review

| Method and path | Purpose | Request / response notes |
|---|---|---|
| `POST /api/deck-sessions` | Start a practice session. | JSON: `{"user_id":"...","deck_id":"..."}`. Returns the session ID and start details. |
| `POST /api/attempts` | Record an answer and calculate its points on the server. | JSON fields: `user_id`, `session_id`, `deck_id`, `card_id`, `selected`, `time_taken_ms`, and optional `hints_used` (0–4). Returns correctness and awarded points. |
| `POST /api/deck-sessions/{session_id}/finish?user_id={user_id}` | Finish a session. | `user_id` is a required query parameter; returns the finished session summary. |
| `POST /api/decks/{deck_id}/overview` | Get the validated end-of-session learning overview. | JSON: `{"session_id":"..."}`. The session must be finished. The result includes a summary, strengths, weak areas, mistake review, and next steps; it is saved and reused when requested again. |
| `GET /api/leaderboard?period=all&limit=10` | Get the ranked leaderboard. | `period` is `all` or `week`; `limit` is 1–100. Response includes rank, username, points, accuracy, and completed decks. |

Example answer submission:

```json
{
  "user_id": "USER_ID",
  "session_id": "SESSION_ID",
  "deck_id": "DECK_ID",
  "card_id": "CARD_ID",
  "selected": "list comprehension",
  "time_taken_ms": 4200,
  "hints_used": 0
}
```

The backend determines correctness from the stored card answer and computes
points; clients must not send a claimed correctness or points value. The scoring
rules are described in [Data storage](#data-storage).

### AI tutor

| Method and path | Purpose | Request / response notes |
|---|---|---|
| `POST /api/ai/explain` | Request an adaptive explanation, a staged hint, or a conversational tutor response. | See the request fields and hint levels below. |
| `POST /api/ai/feedback` | Get feedback on a learner's explanation of a concept. | JSON: `concept`, `correct_answer`, and `learner_restatement`; returns feedback, what was right, and what to refine. |

`POST /api/ai/explain` accepts `question`, `correct_answer`,
`understanding_level` (`ELI5`, `Beginner`, `Intermediate`, or `Advanced`,
default `Beginner`), `hint_level` (1–4, default 4), and optional `user_answer`,
`learner_attempt`, and conversation `history` (an array of objects with `role`
`user` or `assistant` and a `content` string).

| `hint_level` | Tutor response |
|---|---|
| 1 | A guiding question, without revealing the answer |
| 2 | A conceptual hint |
| 3 | A partial worked step |
| 4 | The full explanation |

Example request:

```json
{
  "question": "What is the powerhouse of the cell?",
  "user_answer": "Nucleus",
  "correct_answer": "Mitochondria",
  "understanding_level": "Beginner",
  "hint_level": 1,
  "learner_attempt": "Nucleus"
}
```

### Analytics

| Method and path | Purpose | Query parameters / response notes |
|---|---|---|
| `GET /api/analytics/me/{user_id}` | Return one learner's analytics. | Optional `theme=light` or `theme=dark`. Includes accuracy by deck/subject/difficulty, hint-use rate, daily activity, response-time percentiles, rank, learning-curve slope, weak cards, and chart data URIs. Unknown users return `404`; new users receive zero/empty metrics and valid empty charts. |
| `GET /api/analytics/global` | Return class-wide aggregated analytics. | Optional `theme=light` or `theme=dark`. Includes pooled accuracy by subject, hardest questions (minimum three attempts), most active days, points distribution, and charts. It does not expose learner IDs, selected answers, or individual attempt records. |

The charts are PNGs embedded in the JSON as Base64 `data:image/png` URIs.
Analytics responses are cached in memory for approximately 60 seconds. See
[`backend/analytics.py`](backend/analytics.py) for full explanations of the
pandas, NumPy, and Matplotlib processing and calculations.

## 🧭 Application workflow

1. **Start the service and initialize storage.** Run `./run.sh` (or the manual
   Uvicorn command above). Startup initializes the configured SQLite/PostgreSQL
   storage, imports eligible legacy JSON data once, and ensures the built-in
   subject decks exist.
2. **Create or restore a learner identity.** On the first visit, the frontend
   sends the chosen username to `POST /api/users` and stores the returned user ID
   in browser localStorage. There are no passwords. Later API requests use that
   ID to associate practice and profile data with the learner.
3. **Choose or prepare study content.** The app lists available decks with
   `GET /api/decks`. A learner can use a seeded deck, create one with
   `POST /api/decks`, or optionally upload notes using
   `POST /api/materials/upload`. Uploading extracts text and makes it available
   as context for topic-specific `POST /api/decks/generate`; valid generated
   MCQs are saved as a deck.
4. **Open a practice session.** The frontend sends the learner and deck IDs to
   `POST /api/deck-sessions`. It displays questions and shuffles choices for
   presentation; stored answers are checked by the backend, not inferred from
   the displayed option position.
5. **Record each response.** For every submitted answer, the frontend calls
   `POST /api/attempts` with IDs, selected text, elapsed milliseconds, and hints
   used. The backend checks the answer against the stored card, applies the
   first-correct-per-card-per-day rule and streak/speed/hint scoring, then
   persists the attempt and returns its correctness and points. The frontend
   updates its live score and streak from that response.
6. **Finish and review.** At the end, the session is finished through
   `POST /api/deck-sessions/{session_id}/finish`. The frontend requests
   `POST /api/decks/{deck_id}/overview`; the backend summarizes that session's
   attempts, validates the AI or deterministic fallback response, stores the
   overview, and reuses it on later requests. Wrong cards can then be replayed
   for review.
7. **Inspect progress.** The frontend loads the profile, leaderboard, and
   analytics endpoints. Personal analytics are filtered to the current learner;
   the global analytics view contains aggregates only. The generated charts are
   returned with the corresponding JSON metrics and can be displayed directly
   by the browser.

---

## 📄 License

MIT License

## Presentation demo (offline mode)

The repository seeds ten subject decks (Python, pandas/NumPy, Linux, DBMS/SQL, Operating Systems, Networking, Data Structures and Algorithms, Git, Web/HTTP/REST, and Cybersecurity basics) with 15 validated MCQs each. If no `GEMINI_API_KEY` is configured, generation uses offline practice content; live AI responses require a valid API key. Mindmaps have an offline fallback and do not require a Groq key.

For a smooth demo: (1) start with `./run.sh`, (2) choose a username, (3) practice a seeded deck, (4) view the leaderboard and profile, (5) generate a deck grounded in an uploaded `.txt` or PDF study file, and (6) open a deck mindmap. Use `/docs` to demonstrate the API.

The generator now waits for the saved deck list to refresh before entering practice mode, preventing the new-deck practice race condition.


## Troubleshooting AI generation

- Start the app with `./run.sh`; this script loads `GEMINI_API_KEY` from the project `.env` file.
- After changing frontend code, hard-refresh the browser with `Ctrl+Shift+R`. Static assets use a versioned URL to avoid stale JavaScript.
- If Gemini returns invalid/empty MCQs or the API key is unavailable, deck generation falls back to built-in offline practice questions. This fallback is labelled as offline practice content; it is not live AI output.
- Keep `.env` private and never paste your API key into screenshots or chat.
