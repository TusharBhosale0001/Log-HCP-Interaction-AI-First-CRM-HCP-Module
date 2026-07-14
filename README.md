# AI-First HCP CRM — Log Interaction Screen

Full submission for the "AI-First CRM HCP Module – Log Interaction Screen"
technical assignment: a React/Redux frontend and a FastAPI + LangGraph
backend, letting a pharma field rep log HCP interactions either through a
structured form or a conversational AI Assistant.

```
hcp-crm-backend/     FastAPI + LangGraph + Groq backend  (see its own README.md)
hcp-crm-frontend/    React + Redux Toolkit frontend       (see its own README.md)
```

## Quick start (both halves together)

**1. Backend**

```bash
cd hcp-crm-backend
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # add your GROQ_API_KEY and DATABASE_URL
uvicorn app.main:app --reload
# → http://localhost:8000
```

**2. Frontend** (separate terminal)

```bash
cd hcp-crm-frontend
npm install
npm run dev
# → http://localhost:5173
```

The frontend's Vite dev server proxies `/api/*` to `http://localhost:8000`
(configured in `vite.config.js`), so no CORS setup is needed locally — just
run both and open `http://localhost:5173`.

## What's implemented

- **Structured form** (left panel): all fields from the spec — HCP name,
  interaction type, date/time, attendees, topics, materials/samples (as
  chip-style tag inputs), sentiment (3-way pill selector), outcomes,
  follow-up actions — posts to `POST /interactions`.
- **AI Assistant chat** (right panel): free-text interaction logging via
  `POST /chat`, which drives the LangGraph agent and its 5 tools.
- **LangGraph agent** with the 2 mandatory tools (`log_interaction`,
  `edit_interaction`) plus 3 more (`get_hcp_history`, `search_materials`,
  `suggest_follow_up`) — see `hcp-crm-backend/README.md` for the full
  breakdown and the agent's role.
- Redux Toolkit slices for form state and chat state, with async thunks
  hitting the FastAPI endpoints.

## Verified during development

Both halves were installed, built, and run together in a sandboxed
environment before packaging:
- Backend: `pip install`, server boot, and all REST endpoints
  (`POST/PATCH /interactions`, `GET /interactions/hcp/{id}`) tested against a
  live SQLite DB. `POST /chat` confirmed to build the LangGraph agent and
  reach Groq's API (network egress to the real Groq host isn't available in
  this sandbox, but the full request pipeline up to that point is confirmed
  working).
- **Bug fixed**: `groq==0.11.0` is incompatible with `httpx>=0.28` (dropped
  the `proxies` kwarg) — pinned `httpx==0.27.2` in `requirements.txt` to
  avoid a broken install.
- Frontend: `npm install` and `npm run build` both clean. Dev server verified
  to serve the app and proxy `/api/*` through to the backend, including a
  live `POST /interactions` write test through the proxy.

## Still to do before submitting

- Add a real `GROQ_API_KEY` and confirm the actual model output quality from
  `log_interaction`'s extraction and `suggest_follow_up`'s reasoning — not
  verifiable from this sandbox.
- Point `DATABASE_URL` at a real Postgres/MySQL instance (SQLite was only
  used here for a quick local smoke test).
- Record the 10–15 min walkthrough video per the assignment brief.
- Push to a GitHub repo and submit via the Google Form link in the
  assignment doc.
