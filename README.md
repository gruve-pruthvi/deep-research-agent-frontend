# Deep Research Agent — Frontend

A React + Vite single-page application that provides a streaming chat interface and a full deep research workflow powered by a FastAPI backend.

---

## Prerequisites

Before you begin, make sure you have the following installed:

| Tool | Minimum Version | Check |
|------|----------------|-------|
| Node.js | 18+ | `node -v` |
| npm | 9+ | `npm -v` |
| Docker | 24+ | `docker -v` *(only needed for Docker setup)* |

The frontend talks to the backend at `http://127.0.0.1:8000`. **The backend must be running before you open the UI.** See the backend README for backend setup instructions.

---

## Quick Start (Local)

```bash
# 1. Install dependencies
npm install

# 2. Start the development server
npm run dev
```

Open your browser at **http://localhost:5173**.

That's it. The app hot-reloads on every file save.

---

## All Available Commands

| Command | What it does |
|---------|-------------|
| `npm install` | Install all dependencies |
| `npm run dev` | Start dev server at http://localhost:5173 (with hot reload) |
| `npm run build` | Type-check with TypeScript, then build for production into `dist/` |
| `npm run preview` | Serve the production build locally for verification |
| `npm run lint` | Run ESLint across all source files |

---

## Running with Docker

A two-stage Docker build compiles the app with Node and serves the result with Nginx.

### Build the image

```bash
docker build -t deep-research-frontend .
```

### Run the container

```bash
docker run -p 3000:80 deep-research-frontend
```

Open your browser at **http://localhost:3000**.

> **Important:** The app expects the backend at `http://127.0.0.1:8000`. If your backend runs on a different host or port (e.g., inside another Docker container), update `API_URL` at the top of `src/App.tsx` before running `docker build`.

### Run backend and frontend together (docker compose)

If you want to run the frontend alongside a containerized backend, add this service to the backend's `docker-compose.yml`:

```yaml
frontend:
  build:
    context: ../deep_research_frontend
  ports:
    - "3000:80"
  depends_on:
    - backend
```

Then update `API_URL` in `src/App.tsx` to point to the backend service name (e.g., `http://backend:8000`).

---

## Changing the Backend URL

The backend URL is defined in one place:

```
src/App.tsx  →  line 38  →  const API_URL = 'http://127.0.0.1:8000'
```

Edit that value to point to wherever your backend is running, then restart the dev server (or rebuild the Docker image).

---

## What the App Does

### Chat Mode

Type a message and get a streaming response, token by token. Any tools the assistant runs are displayed in an expandable **Tool activity** panel below the response.

### Research Mode

Submit a research topic and the backend runs a full multi-agent pipeline. The UI shows:

- **Clarification dialog** — If the query is ambiguous, a modal appears with 1–2 clarifying questions. You can answer or skip.
- **Depth selector** — Choose `Shallow` (~400–700 words), `Standard` (~700–1200 words), or `Deep` (~1200–1800 words).
- **Advanced options** — Click *Advanced* to reveal an iterations slider (1–5).
- **Research plan** — The planned search queries are shown before searching begins.
- **Live progress timeline** — 16 pipeline stages tracked in a side panel, with the active stage highlighted.
- **Source cards** — Each source gets a credibility badge (green ≥ 80%, amber ≥ 60%, red < 60%).
- **Confidence gauge** — A visual bar showing evidence confidence after the verify stage.
- **Verifier notes** — Collapsible section with the verifier agent's assessment.
- **Evaluation scores** — Coverage / Evidence / Clarity scored out of 5.
- **Transparency panel** — Full list of queries run, sources used, and verifier notes.
- **Export** — After the report finishes, **Download .md** and **Copy to clipboard** buttons appear.
- **History** — Click *History* in the header to browse past research runs for the current session and restore any report.

---

## Project Structure

```
deep_research_frontend/
├── src/
│   ├── App.tsx       # All component logic, SSE parsing, UX flows
│   ├── App.css       # Theme, layout, and component styles
│   ├── index.css     # Global resets and font imports
│   └── main.tsx      # React entry point
├── index.html        # HTML shell
├── vite.config.ts    # Vite configuration
├── tsconfig.json     # TypeScript configuration
├── Dockerfile        # Multi-stage Docker build
└── package.json      # Dependencies and scripts
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Blank page | Open the browser console — there is likely a runtime error in `App.tsx` |
| 404 on API calls | Make sure the backend is running on port 8000 |
| CORS errors | Set `ALLOWED_ORIGINS=http://localhost:5173` in the backend `.env` |
| Clarification dialog never shows | Check that `OPENAI_API_KEY` is set in the backend environment |
| History panel is empty | The backend needs Postgres running and at least one completed research run |
| Export buttons never appear | Check the browser Network tab — verify that a `[DONE]` SSE event is being received |
