# LunaLog

A daily mood journaling app that uses AI to surface insights and personalized recommendations from your entries.

## Architecture

```
LunaLogApp  ──►  LunaLogBackend  ──►  LunaLogAI  ──►  SQLite + Gemini
(Expo / RN)      (Express :8000)      (Flask :5100)
```

**LunaLogApp** — the mobile frontend. Presents 10 mood questions with a 1–10 moon rating scale, calculates a weighted mood score, and shows an AI overview screen after each submission.

**LunaLogBackend** — the Express API gateway. Receives journal submissions from the app, computes the weighted mood score, and forwards data to the AI service. Also proxies journal history back to the app.

**LunaLogAI** — the Python/Flask data and AI layer. Persists entries in SQLite and calls Google Gemini 2.5 Flash to generate 5 insights + 3 actionable recommendations from the last 10 entries.

---

## Getting Started

### Prerequisites

| Tool | Version |
|------|---------|
| Node.js | 18+ |
| Python | 3.10+ |
| Expo Go | Latest (iOS / Android) |

---

### 1. LunaLogAI (Flask — port 5100)

```bash
cd LunaLogAI
pip install -r requirements.txt
```

Create a `.env` file:

```
GEMINI_API_KEY=your_key_here
```

Initialize the database and start the server:

```bash
python Ingests.py
```

To seed the database with sample entries for testing:

```bash
python MockIngest.py
```

---

### 2. LunaLogBackend (Express — port 8000)

```bash
cd LunaLogBackend
npm install
npm run devStart
```

---

### 3. LunaLogApp (Expo)

```bash
cd LunaLogApp
npm install
```

Update the API base URL in [LunaLogApp/scripts/journalService.ts](LunaLogApp/scripts/journalService.ts) to point at your machine's local IP:

```ts
const API_BASE_URL = "http://<your-local-ip>:8000";
```

Start the dev server:

```bash
npm start
```

Scan the QR code with Expo Go on your phone, or press `i` / `a` for iOS/Android simulators.

---

## API Reference

### LunaLogBackend (`:8000`)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/submit` | Submit a journal entry. Body: `{ questionValues: number[] }` |
| `GET` | `/journalEntries` | Fetch all journal entries with AI insights |

### LunaLogAI (`:5100`)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/journal` | Persist a new journal entry to SQLite |
| `POST` | `/recommend` | Generate AI insights and recommendations from the last 10 entries |

---

## Mood Score

Each submission is scored from the 10 question responses using a weighted formula:

| Question | Weight | Note |
|----------|--------|------|
| Q1 | 4 | |
| Q2 | 1 | |
| Q3 | 4 | Inverted — higher answer = lower score |
| Q4 | 4 | |
| Q5 | 2 | |
| Q6 | 2 | |
| Q7 | 2 | |
| Q8 | 1 | |
| Q9 | 1 | |
| Q10 | 4 | |

`moodScore = Σ(value × weight) / 25`

---

## Project Structure

```
Lunalog-Hackathon-2025/
├── LunaLogApp/              # Expo / React Native mobile app
│   ├── app/(tabs)/          # Home (journal flow) + History screens
│   ├── components/          # MoonRatingInput, JournalEntryCard, AIOverview, etc.
│   ├── scripts/             # journalService.ts — API calls
│   └── assets/              # Moon rating images (1–10), questions.json
├── LunaLogBackend/          # Express API gateway
│   └── server.js
└── LunaLogAI/               # Flask AI + data service
    ├── Ingests.py            # Flask app, DB init, Gemini integration
    ├── CreateTables.py       # Standalone DB schema setup
    ├── MockIngest.py         # Seed script for test data
    └── requirements.txt
```
