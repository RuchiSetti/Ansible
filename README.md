# Task Management System

A full-stack web application for managing tasks — built with **React + Vite** on the frontend and **Spring Boot** on the backend, with **MySQL** as the database. The entire stack is containerized with Docker and automated through a GitHub Actions CI/CD pipeline.

---

## What This App Does

It's a simple but complete task tracker where you can:

- Create, edit, and delete tasks
- Assign priority levels (Low, Medium, High)
- Track task progress through a Kanban-style board (Assigned → In Progress → Completed / Rejected)
- Search tasks by any field — title, description, priority, status, ID
- View all tasks in a clean table on the dashboard

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite 7, React Router, Axios |
| Backend | Spring Boot 3.5, Spring Data JPA |
| Database | MySQL |
| Containerization | Docker (multi-stage builds) |
| CI/CD | GitHub Actions |
| API Docs | Swagger / SpringDoc OpenAPI |

---

## Project Structure

```
├── reactfrontend/              # React + Vite frontend
│   ├── src/
│   │   ├── components/         # UI components (Navbar, Dashboard, Board)
│   │   ├── api/                # Axios service layer (TaskService)
│   │   └── App.jsx             # Routes setup
│   ├── nginx.conf              # Nginx config for serving the build
│   └── frontend.Dockerfile     # Multi-stage Docker build for frontend
│
├── springbootbackend/          # Spring Boot backend
│   ├── src/main/java/com/klef/dev/
│   │   ├── controller/         # REST API endpoints
│   │   ├── entity/             # Task JPA entity
│   │   ├── repository/         # Spring Data JPA repository
│   │   ├── service/            # Business logic
│   │   └── config/             # Swagger config
│   └── backend.Dockerfile      # Multi-stage Docker build for backend
│
└── .github/workflows/
    └── docker-image.yml        # CI pipeline to build and push Docker images
```

---

## Running Locally (Without Docker)

### Prerequisites

- Node.js 20+
- Java 21
- Maven
- MySQL running locally

### Backend

1. Create a MySQL database named `projectdb`
2. Update `application.properties` if your MySQL credentials differ:
   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/projectdb
   spring.datasource.username=root
   spring.datasource.password=password
   ```
3. Run the backend:
   ```bash
   cd springbootbackend
   ./mvnw spring-boot:run
   ```
   The server starts on **port 2000**.

### Frontend

1. Set the API URL in `.env`:
   ```env
   VITE_API_URL=http://localhost:2000
   ```
2. Install dependencies and start:
   ```bash
   cd reactfrontend
   npm install
   npm run dev
   ```
   The app opens at **http://localhost:5173**.

---

## Running With Docker

Both services have multi-stage Dockerfiles that keep image sizes small.

### Build the backend image

```bash
cd springbootbackend
docker build -f backend.Dockerfile -t task-backend .
```

### Build the frontend image

```bash
cd reactfrontend
docker build -f frontend.Dockerfile -t task-frontend .
```

The frontend Dockerfile builds the Vite app and serves it using **Nginx** on port 80. The nginx config handles client-side routing by falling back to `index.html` for all routes.

---

## CI/CD Pipeline

Every push to `main` triggers the GitHub Actions workflow (`.github/workflows/docker-image.yml`), which:

1. Checks out the code
2. Sets up Docker Buildx
3. Logs into DockerHub using repository secrets
4. Builds and pushes the **backend** image as `<your-username>/ansible-backend:ruchita`
5. Builds and pushes the **frontend** image as `<your-username>/ansible-frontend:ruchita`

### Setting up secrets

Go to your GitHub repo → **Settings → Secrets → Actions** and add:

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_SECRET` | Your DockerHub access token |

---

## API Endpoints

The backend exposes the following REST endpoints under `/api/tasks`:

| Method | Endpoint | Description |
|---|---|---|
| GET | `/all` | Get all tasks |
| GET | `/get/{id}` | Get a task by ID |
| POST | `/add` | Create a new task (status auto-set to `ASSIGNED`) |
| PUT | `/update/{id}` | Update full task details |
| PUT | `/updatestatus/{id}` | Update only the task status |
| DELETE | `/delete/{id}` | Delete a task |

Full interactive API documentation is available at:
```
http://localhost:2000/swagger-ui.html
```

---

## Task Status Flow

Tasks follow this lifecycle on the Task Board:

```
ASSIGNED  →  PROGRESS  →  COMPLETED
                      ↘  REJECTED  →  ASSIGNED
```

Each status column on the board shows action buttons to move tasks forward or backward in the workflow.

---

## Environment Notes

- The backend expects MySQL to be reachable at the hostname `mysql-service` (Kubernetes/Docker network default). Change the datasource URL in `application.properties` when running locally.
- Task IDs are randomly generated in the backend — the frontend doesn't send an ID when creating a task.
- The `endDate` field is validated on the frontend to prevent it from being set before the `startDate`.
