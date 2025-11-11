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

Before you deploy: Database setup (CRITICAL for Render)
**⚠️ IMPORTANT: Render's free tier uses an ephemeral filesystem. SQLite files on disk will NOT persist between restarts. Choose one:**

**Option A: In-Memory SQLite (data lost on restart — testing only)**
- Default `render.yaml` sets `DATABASE_URL=sqlite+aiosqlite:///:memory:`
- Use this only for testing/demo purposes. Data is wiped every restart.

**Option B: PostgreSQL on Render (recommended for production)**
1. In Render dashboard, create a **PostgreSQL instance**:
   - Click "New +" → PostgreSQL
   - Name: `beaverhabits-postgres` (or similar)
   - Set any password and region
   - Free tier or paid, as you prefer
2. Copy the **Internal Database URL** (e.g., `postgresql://user:pass@host:5432/dbname`)
3. In your web service environment variables, set `DATABASE_URL` to the copied URL. Render will auto-wire it if you use `fromDatabase`.
4. Or manually paste the connection string into the `DATABASE_URL` env var.

Before you deploy (other settings)
1. Confirm Python version in Render matches the project (pyproject requires Python >=3.12). Use Render's environment settings to set Python 3.12.
2. Ensure any secrets are added to Render as environment variables (for example, `SENTRY_DSN`, `PADDLE_*`, or other keys used by the app).

Deploy using Render dashboard (recommended)
1. Push your repo to GitHub (your fork or your own repo).
2. **If using PostgreSQL option:** Create a PostgreSQL instance first (see "Database setup" above), then copy the **Internal Database URL**.
3. In Render, create a new Web Service, connect your GitHub repo and choose the `main` branch.
4. Use the `render.yaml` in repo for auto-configuration, or manually set:
   - Environment: Python
   - Build command: `pip install . --no-cache-dir`
   - Start command: `gunicorn beaverhabits.main:app --bind 0.0.0.0:$PORT -w 1 -k uvicorn_worker.UvicornWorker --max-requests 5000 --log-level info`
5. Set environment variables under the service settings:
   - `ENV=production`
   - `NICEGUI_STORAGE_PATH=.user/.nicegui`
   - **`DATABASE_URL`**: Use `:memory:` for testing, or the PostgreSQL connection string for production. If using a managed DB, Render can link it automatically.

Deploy using the `render.yaml`
1. If you push a `render.yaml` to your repo, Render will pick it up and auto-create the service when you connect the repo (or you can import it from the dashboard).

Troubleshooting & logs
- Use Render's dashboard Logs tab to view startup errors.
- **Common issues:**
  - Wrong Python version → adjust Render service settings to Python 3.12.
  - Missing build dependencies → ensure `pyproject.toml` lists required packages.
  - App binds to a fixed port (8080) → use the `$PORT` variable in the start command.
  - **`sqlite3.OperationalError: unable to open database file`** → Render's filesystem is ephemeral. Set `DATABASE_URL` to `:memory:` (for testing) or a PostgreSQL connection string (for production).

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
3. **Decide on a database:**
   - For testing: Use in-memory SQLite (default, `DATABASE_URL=sqlite+aiosqlite:///:memory:`). Data is lost on restart.
   - For production: Create a managed PostgreSQL on Render and set `DATABASE_URL` to the connection string.
4. Add required env vars (ENV=production, NICEGUI_STORAGE_PATH, DATABASE_URL, any DB credentials, SENTRY_DSN if used).
5. Deploy and share logs if anything fails — I can help fix runtime issues.

If you'd like, I can also prepare a sample `render-postgres.yaml` that auto-creates a PostgreSQL instance alongside the web service. Tell me if you want that.

