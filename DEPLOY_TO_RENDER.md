# Deploying BeaverHabits to Render

This file explains how to deploy this repository to Render and how to configure the service.

Important legal/ethics note
- I cannot assist with removing or hiding the original author's attribution. The project includes a `LICENSE` and `pyproject.toml` that list the original author. If you want this to be "your" project, the correct paths are:
  1. Fork the repository on GitHub and make meaningful changes (branding, features, config). Keep and respect the original license and attribution unless the license explicitly permits re-licensing.
  2. Add your name to the project metadata where appropriate (e.g., AUTHORS or project metadata) while keeping the original author's credit in place per the license.

Quick summary (what I added)
- `render.yaml` — Render manifest that defines a web service and a start command that uses Gunicorn bound to `$PORT`.

How Render runs this app
- Build step installs the package from the repo: `pip install .`
- Start uses: `gunicorn beaverhabits.main:app --bind 0.0.0.0:$PORT -w 1 -k uvicorn_worker.UvicornWorker` which runs the FastAPI app via Gunicorn + Uvicorn worker.

Before you deploy
1. Confirm Python version in Render matches the project (pyproject requires Python >=3.12). Use Render's environment settings to set Python 3.12.
2. Decide on a data store:
   - By default the app can use sqlite (file under `.user`) or memory stores. For production you likely want Postgres.
   - If you use Render Postgres, add a managed database and set `DATABASE_URL` in Render (the app will pick it up via configs if configured).
3. Ensure any secrets are added to Render as environment variables (for example, `SENTRY_DSN`, `PADDLE_*`, or other keys used by the app).

Deploy using Render dashboard (recommended)
1. Push your repo to GitHub (your fork or your own repo).
2. In Render, create a new Web Service, connect your GitHub repo and choose the `main` branch.
3. Use the `render.yaml` in repo for auto-configuration, or manually set:
   - Environment: Python
   - Build command: `pip install . --no-cache-dir`
   - Start command: use the `startCommand` from `render.yaml` or `bash start.sh prd` (but note `start.sh` uses hard-coded port 8080; using the `gunicorn ... $PORT` command is preferred so it binds to Render's assigned `$PORT`).
4. Set environment variables under the service settings: `ENV=production` and `NICEGUI_STORAGE_PATH=.user/.nicegui` at minimum.

Deploy using the `render.yaml`
1. If you push a `render.yaml` to your repo, Render will pick it up and auto-create the service when you connect the repo (or you can import it from the dashboard).

Troubleshooting & logs
- Use Render's dashboard Logs tab to view startup errors.
- Common issues:
  - Wrong Python version → adjust Render service settings.
  - Missing build dependencies → ensure `pyproject.toml` lists required packages.
  - App binds to a fixed port (8080) → use the `$PORT` variable in the start command.

What I can do for you (without credentials)
- Prepare or update a `render.yaml` (done).
- Prepare additional env var examples or a sample `render-postgres` manifest to add a managed DB.
- Walk you through connecting the repo to Render UI and setting env vars.
- Review logs and help fix runtime errors if you share log snippets.

What I cannot do without your input/credentials
- I cannot push to your Render account or create services there (requires access to your GitHub/Render account).
- I cannot remove author attribution or metadata. Please respect the original LICENSE and author fields.

Next steps I recommend you take
1. Fork or move the repo into your GitHub account.
2. Push changes and connect the repo to Render; use the included `render.yaml`.
3. Add required env vars (ENV=production, NICEGUI_STORAGE_PATH, any DB credentials, SENTRY_DSN if used).
4. Deploy and share logs if anything fails — I can help fix runtime issues.

If you'd like, I can also prepare a small `render-postgres.yaml` example and a brief `branding.md` containing recommended places to add your logo and name in the UI (without removing the license). Tell me if you want that and whether you plan to use Postgres or sqlite.

