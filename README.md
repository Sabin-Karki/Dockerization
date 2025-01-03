# Dockerized Fullstack Application Setup

This project demonstrates how to Dockerize a fullstack application with a React frontend, Spring Boot backend, and PostgreSQL database. It explains the need for containerization, how to achieve it, and the advantages it provides for development and deployment.

---

## Why Dockerization?

Dockerization refers to packaging your application along with its dependencies into containers. These containers provide a consistent and isolated environment, ensuring that your application runs seamlessly regardless of the host machine. 

### Benefits:
1. **Portability**: Containers can run on any system with Docker installed, eliminating environment-specific issues.
2. **Isolation**: Each service (frontend, backend, and database) runs in its container, preventing conflicts.
3. **Scalability**: Easily scale individual services based on demand.
4. **Simplified Deployment**: Using `docker-compose`, you can start your entire application stack with a single command.
5. **Consistency**: Ensures that the application behaves the same in development, testing, and production.

---

## Project Structure

```
project-root/
├── Backend/       # Spring Boot backend
│   ├── src/       # Source code for the backend
│   ├── pom.xml    # Maven configuration file
│   └── Dockerfile # Dockerfile for backend
│
├── Frontend/      # React frontend
│   ├── src/       # React source code
│   ├── public/    # Public assets
│   ├── package.json # npm configuration file
│   └── Dockerfile # Dockerfile for frontend
│
├── docker-compose.yml # Docker Compose configuration
```

---

## Prerequisites

Ensure you have the following installed:
- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)

---

## Docker Compose Configuration

The `docker-compose.yml` file orchestrates the backend, frontend, and database services. Below is an overview of each service:

### Backend Service
- **Context**: `./Backend`
- **Exposed Port**: `8081` (mapped to container's port `8080`)
- **Environment Variables**:
  - `SPRING_DATASOURCE_URL`: Connection string for the database.
  - `SPRING_DATASOURCE_USERNAME`: Database username.
  - `SPRING_DATASOURCE_PASSWORD`: Database password.

### Frontend Service
- **Context**: `./Frontend`
- **Exposed Port**: `3000`
- **Volumes**:
  - `./Frontend/src:/app/src`: For hot-reloading React code.
  - `./Frontend/public:/app/public`: To serve static assets.
  - `/app/node_modules`: Prevent node_modules conflicts.
- **Environment Variables**:
  - `CHOKIDAR_USEPOLLING=true`: Enables file polling for hot-reloading.
  - `WATCHPACK_POLLING=true`: Ensures compatibility with Docker-mounted volumes.

### Database Service
- **Image**: `postgres:15`
- **Exposed Port**: `5434` (mapped to container's port `5432`)
- **Environment Variables**:
  - `POSTGRES_DB`: Name of the database.
  - `POSTGRES_USER`: Username for the database.
  - `POSTGRES_PASSWORD`: Password for the database.
- **Volumes**:
  - `postgres_data:/var/lib/postgresql/data`: To persist database data.

---

## Dockerfile Details

### Backend Dockerfile
Located in `Backend/Dockerfile`:
```Dockerfile
# Stage 1: Build the application
FROM maven:3.8.4-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run the application
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar /app/app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

### Frontend Dockerfile
Located in `Frontend/Dockerfile`:
```Dockerfile
FROM node:21
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY ./src ./src
COPY ./public ./public
EXPOSE 3000
CMD ["npm", "start"]
```

---

## How to Run the Project

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd project-root
   ```

2. **Start the Application**:
   Build and start all services:
   ```bash
   docker-compose up --build
   ```

3. **Access the Services**:
   - **Frontend**: [http://localhost:3000](http://localhost:3000)
   - **Backend API**: [http://localhost:8081](http://localhost:8081)
   - **Database**: PostgreSQL running on `localhost:5434`

---

## Why This Setup Works

1. **Backend**:
   - Spring Boot runs inside a Docker container, ensuring consistent runtime and dependencies.
   - It connects to the database using environment variables defined in `docker-compose.yml`.

2. **Frontend**:
   - The React app runs in a containerized Node.js environment.
   - Hot-reloading is supported via mounted volumes and polling configurations.

3. **Database**:
   - PostgreSQL is isolated in its container, with data persistence enabled through a named volume.

4. **Networking**:
   - Docker Compose automatically sets up a network so all services can communicate.

5. **Port Mapping**:
   - Frontend: Accessible via `localhost:3000`
   - Backend: Accessible via `localhost:8081`
   - Database: Accessible via `localhost:5434`

---

## Troubleshooting

### Common Issues
1. **Missing Files in Container**:
   - Ensure volumes are mounted correctly in `docker-compose.yml`.
2. **Frontend Hot-Reload Not Working**:
   - Ensure `CHOKIDAR_USEPOLLING=true` and `WATCHPACK_POLLING=true` are set in the frontend service.
3. **Database Connection Errors**:
   - Verify `SPRING_DATASOURCE_URL`, username, and password match the `db` service configuration.

---

## Future Improvements
- Add production configurations for both frontend and backend.
- Use environment-specific `.env` files to manage variables.
- Integrate CI/CD pipelines for automated deployment.

---

## License
This project is licensed under the MIT License. See the LICENSE file for details.

---

Enjoy exploring Dockerized fullstack applications! 🚀

