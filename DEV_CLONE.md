# Isolated development clone

This folder is a separate copy of the attendance app. Its Docker Compose project,
MongoDB/Redis data, and localhost ports are isolated from the shared testing
instance in `updated_automation-main`.

## Docker clone

From this folder in PowerShell:

```powershell
docker compose -p attendance-dev --env-file backend\.env -f deploy\docker-compose.production.yml -f deploy\docker-compose.local.yml -f deploy\docker-compose.clone.yml up --build -d
```

Open `http://localhost:8081`. The clone's API health endpoint is
`http://localhost:8081/api/health`.

The clone binds only to localhost and does not use the shared ngrok tunnel.
Its isolated MongoDB and Redis are available at `localhost:27019` and
`localhost:6380`; its backend is at `localhost:8001`.

To stop only the clone:

```powershell
docker compose -p attendance-dev --env-file backend\.env -f deploy\docker-compose.production.yml -f deploy\docker-compose.local.yml -f deploy\docker-compose.clone.yml down
```

The clone's database starts empty, including its admin-user collection. There
is no public registration page. Create the first clone admin from the backend
container with:

```powershell
docker compose -p attendance-dev --env-file backend\.env -f deploy\docker-compose.production.yml -f deploy\docker-compose.local.yml -f deploy\docker-compose.clone.yml exec backend python scripts/create_admin.py
```

## Local frontend development

The Vite dev server uses the clone API on port `8001` and serves on
`http://localhost:5174`. From this folder:

```powershell
cd frontend
npm ci
npm run dev
```

Run the Docker clone first so the API and its isolated databases are available.
