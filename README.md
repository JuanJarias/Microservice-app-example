# Microservice App - PRFT Devops Training

This is the application you are going to use through the whole traninig. This, hopefully, will teach you the fundamentals you need in a real project. You will find a basic TODO application designed with a [microservice architecture](https://microservices.io). Although is a TODO application, it is interesting because the microservices that compose it are written in different programming language or frameworks (Go, Python, Vue, Java, and NodeJS). With this design you will experiment with multiple build tools and environments. 

## Components
In each folder you can find a more in-depth explanation of each component:

1. [Users API](/users-api) is a Spring Boot application. Provides user profiles. At the moment, does not provide full CRUD, just getting a single user and all users.
2. [Auth API](/auth-api) is a Go application, and provides authorization functionality. Generates [JWT](https://jwt.io/) tokens to be used with other APIs.
3. [TODOs API](/todos-api) is a NodeJS application, provides CRUD functionality over user's TODO records. Also, it logs "create" and "delete" operations to [Redis](https://redis.io/) queue.
4. [Log Message Processor](/log-message-processor) is a queue processor written in Python. Its purpose is to read messages from a Redis queue and print them to standard output.
5. [Frontend](/frontend) Vue application, provides UI.

## Architecture

Take a look at the components diagram that describes them and their interactions.
![microservice-app-example](/arch-img/Microservices.png)


# Microservice App Example

A small microservices-based application that can be started locally with Docker Compose. The project groups independent services behind one command, so you do not need to install Python, Node.js, Redis, or each service's dependencies directly on your machine.

## What runs when you deploy it

Docker Compose reads the `docker-compose.yml` file and creates a shared Docker network for the application. It then builds the service images from the Dockerfiles in their respective folders and starts the containers as one application stack.

The stack contains the following components:

| Service | Purpose | Host port |
|---|---|---:|
| `frontend` | Browser-facing user interface | Check `docker-compose.yml` |
| `auth-api` | Authentication API and JWT-related operations | `8000` |
| `users-api` | User management API | `8083` |
| `todos-api` | To-do management API | `8082` |
| `log-message-processor` | Background service that consumes log messages | Internal service |
| `redis` | Message broker used by the application services | `6379` |

Each container can reach the others through the Compose network using the **service name** as the hostname. For example, `auth-api` reaches the users service through `http://users-api:8083`, and `todos-api` connects to Redis through the host `redis` and port `6379`.

> Inside a container, `localhost` means the current container only. Use service names such as `redis` or `users-api` for communication between services.

## Requirements

Install the following before starting the project:

- [Docker Engine](https://docs.docker.com/engine/install/) or Docker Desktop
- Docker Compose v2, available through the `docker compose` command
- An Internet connection the first time you build, because Docker must download base images and packages

Verify the installation:

```bash
docker --version
docker compose version
```

On Linux, if Docker requires elevated permissions, run the commands below with `sudo`, as shown in the examples.

## Start the application

Clone the repository and move into its root directory:

```bash
git clone https://github.com/JuanJarias/Microservice-app-example.git
cd Microservice-app-example
```

Build the images and start the entire stack:

```bash
sudo docker compose up --build
```

Docker Compose will:

1. Read the service definitions from `docker-compose.yml`.
2. Build local images for the APIs, frontend, and background processor.
3. Pull the Redis image if it is not already available locally.
4. Create a shared network for the containers.
5. Start Redis and the application services.
6. Stream all service logs to the terminal.

Use `Ctrl + C` to stop the running stack.

To run it in the background instead:

```bash
sudo docker compose up --build -d
```

Check which services are running:

```bash
sudo docker compose ps
```

Follow the logs of every service:

```bash
sudo docker compose logs -f
```

Or inspect one service, for example the authentication API:

```bash
sudo docker compose logs -f auth-api
```

## Service configuration

The `docker-compose.yml` file provides the runtime configuration for the containers.

- `users-api` is exposed on port `8083` and receives `SERVER_PORT=8083` plus the shared JWT secret.
- `auth-api` is exposed on port `8000`. It uses `USERS_API_ADDRESS=http://users-api:8083` to call the users service after Compose networking is available.
- `todos-api` is exposed on port `8082`. It connects to Redis with `REDIS_HOST=redis`, `REDIS_PORT=6379`, and publishes to the `log_channel` channel.
- `log-message-processor` is built from `./log-message-processor` and works with the Redis-backed logging flow.
- `redis` publishes port `6379` for local development and is reachable by the other containers at `redis:6379`.

For a real deployment, do not keep a JWT secret directly in the Compose file. Put sensitive values in environment variables, a `.env` file excluded from Git, Docker secrets, or your platform's secret manager.

## Rebuild after code changes

When you change application code or a Dockerfile, rebuild the affected service:

```bash
sudo docker compose build auth-api
sudo docker compose up -d
```

To rebuild every service without using Docker's layer cache:

```bash
sudo docker compose build --no-cache
sudo docker compose up -d
```

To rebuild only the log processor after changing its Dockerfile or Python dependencies:

```bash
sudo docker compose build --no-cache log-message-processor
sudo docker compose up -d log-message-processor
```

## Stop and clean up

Stop and remove the containers and Compose network:

```bash
sudo docker compose down --remove-orphans
```

If you also need to delete Docker volumes created by the stack, use:

```bash
sudo docker compose down -v --remove-orphans
```

Be careful with `-v`: it removes persisted Docker volume data.

## Troubleshooting

### `apt-get` exits with code 100

If an image build fails while installing Linux packages, make sure the Dockerfile uses a supported base image. For the log processor, use a current Python slim image such as:

```dockerfile
FROM python:3.11-slim
```

Then rebuild it without cache:

```bash
sudo docker compose build --no-cache log-message-processor
```

### A port is already in use

Find the process or container already using the port:

```bash
sudo docker ps
```

Then stop the conflicting container, or change the host-side port mapping in `docker-compose.yml`.

### A service cannot reach another service

Use the Docker Compose service name as the hostname. For example:

```text
http://users-api:8083
redis://redis:6379
```

Do not use `localhost` for communication between containers.

### Docker requires `sudo`

You can keep using `sudo docker compose ...`. Alternatively, add your current Linux user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Log out and sign in again before retrying Docker without `sudo`.

## Useful commands

```bash
# Build and start all services
sudo docker compose up --build

# Start in background
sudo docker compose up -d

# Show container status
sudo docker compose ps

# Read logs from all services
sudo docker compose logs -f

# Read logs from one service
sudo docker compose logs -f todos-api

# Stop and remove the stack
sudo docker compose down --remove-orphans

# Remove containers, network, and volumes
sudo docker compose down -v --remove-orphans
```

## Notes

This repository is intended as a local development and learning example of service-to-service communication with Docker Compose. For production, add environment-specific configuration, health checks, persistent storage policies, secrets management, observability, and an external reverse proxy or orchestration platform where appropriate.
