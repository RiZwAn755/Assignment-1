# Deployment & Infrastructure Notes

I have updated the Docker setup to improve the project's security and performance. Here’s a quick overview of the changes.

### 1. Dockerfile Optimization
*   **Backend**: Switched to the `alpine` variant of Node.js to keep the image small and fast. I also added a dedicated **non-root user** (`appuser`) so the app doesn't run with administrative privileges—this is a standard security best practice.
*   **Frontend (Multi-Stage)**: Instead of running a full Node environment to serve the UI, I've used a multi-stage build. It compiles the React app and then uses **Nginx** to serve the static files. This makes the frontend container much lighter and better suited for production traffic.

### 2. Database & Orchestration
*   **Health Checks**: I added a health check to the PostgreSQL service in `docker-compose.yml`. This ensures the database is fully ready before the backend tries to connect, which fixes the common "connection refused" errors on startup.
*   **Port Mapping**: The frontend is now served on port 80 inside the container, but I've kept it mapped to port 3000 on the host to keep things simple for local testing.

### How to Run
Everything still runs via the standard command:
```bash
docker compose up --build
```
The app will be available at `localhost:3000`, and the database will persist data across restarts using a named volume.
