# Attendance testing stack

This Compose project runs the API, workers, frontend, Nginx, SQL Server,
Redis, and ngrok in Docker. It uses its own SQL Server Developer container and
database volume for testing; it does not connect to the organization's SQL
Server or use the production Compose configuration under `deploy`.

The backend reads application settings from `backend/.env` and the SQL
credentials from `attendance_dev/.env`. Copy `.env.example` to `.env` and set
two unique, strong passwords before starting the stack. The app connects to
the `attendance_dev` database in this stack's SQL Server container. Redis and
the legacy MongoDB service are isolated in this stack; the app uses Redis but
not MongoDB. The deploy stack's MongoDB and Redis data are not mounted or connected.
Interakt credentials are read from `backend/.env` for webhook verification
and messaging; attendance events processed in this environment can send
real WhatsApp messages through the configured Interakt account.

The test stack's roster hierarchy assigns field supervisors, local supervisors,
and local supervisor scanning participants directly to project co-ordinators.
The admin attendance screens include serial numbers, totals, area filters, and
Excel exports. The participant screen has separate exports for filtered
attendance and roster details only. Attendance photo OCR in this stack uses English, Hindi, and
Marathi language data for reading dates, times, and assigned-site coordinates.
Super admins and manual-team accounts can change an attendance record's status
from the Attendance session or edit any participant/date cell in the monthly
grid. Status changes are retained in the override history; adding a reason is
optional.
If a saved status override blocks a manual entry, the dialog explains that the
override remains after deleting its submission and offers to update that status
instead.
The test stack flags faces smaller than 7% of the photo for review; users should
frame a clear face while retaining the location and timestamp overlay.

All services run in the `attendance-dev` Compose network. SQL Server and Redis
are not published to host ports. The web app is available on `localhost:8081`,
leaving port `8080` available to the existing deployment. The SQL Server
Developer edition and its database volume are for testing only; production
continues to use its separately configured database.

The ngrok service mounts the existing Windows ngrok config read-only from
`%LOCALAPPDATA%\ngrok\ngrok.yml`. The ngrok URL is public; share it only with the testing team. Ngrok's free
browser interstitial may appear on first visit; testers can select "Visit Site"
to continue. The dashboard remains protected by the application's login.

From the repository root in PowerShell:

```powershell
Copy-Item attendance_dev/.env.example attendance_dev/.env
# Edit attendance_dev/.env and replace both placeholder passwords.
docker compose --env-file backend/.env --env-file attendance_dev/.env -f attendance_dev/docker-compose.yml up --build -d
docker compose --env-file backend/.env --env-file attendance_dev/.env -f attendance_dev/docker-compose.yml ps
docker compose --env-file backend/.env --env-file attendance_dev/.env -f attendance_dev/docker-compose.yml logs -f ngrok
```

Open `http://localhost:8081` locally. The ngrok logs show the public HTTPS URL.
To stop the services without deleting test data:

```powershell
docker compose --env-file backend/.env --env-file attendance_dev/.env -f attendance_dev/docker-compose.yml down
```

To delete all testing database and Redis volumes, including the SQL Server
database:

```powershell
docker compose --env-file backend/.env --env-file attendance_dev/.env -f attendance_dev/docker-compose.yml down --volumes
```

The last command deletes all test data.
