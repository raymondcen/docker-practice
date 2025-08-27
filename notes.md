# Basic Docker Images and Containers
## Dockerfile file
- FROM python:3.13-alpine
  - Specifies the base image to start from. In this case, Alpine Linux with Python 3.13 preinstalled.
  - Sets the foundation for your container environment.
- COPY . /app
  - Copies files from your local directory (the `.`) into the `/app` directory inside the container.
  - Used to add your application code or other files into the container.
- WORKDIR /app
  - Sets the working directory inside the container to `/app`.
  - Any subsequent commands (like RUN or CMD) will be executed in this directory.
- RUN pip install -r requirements.txt
  - Executes a command inside the container at build time.
  - Installs Python dependencies listed in `requirements.txt`.
- CMD ["python", "app.py"]
  - Defines the default command that runs when the container starts.
  - Here, it runs your Python application `app.py`.

## Building the Image
- `docker build -t welcome-app .`
  - Builds a Docker image from the Dockerfile in the current directory (`.`).
  - The `-t welcome-app` tags the image with the name `welcome-app`.
  - You can then see your image by running `docker images`.
  - After building, you can run the container using:
    - `docker run -p 5000:5000 welcome-app`
      - Maps port 5000 on your host to port 5000 in the container (adjust if your app uses a different port).
- Checking docker images
  - `docker images`
### If you have more than 1 Dockerfile
- You **can have multiple Dockerfiles** in the same directory.
- Each Dockerfile must have a **unique name** (the default is `Dockerfile`).
- To build an image using a specific Dockerfile, use the `-f` flag:
- Build using the default Dockerfile
  - `docker build -t my-app .`
- Build using a custom Dockerfile named Dockerfile.dev
  - `docker build -f Dockerfile.dev -t my-app-dev .`

## Running the Image as a Container
- `docker run -p 5000:5000 welcome-app`
  - Runs a container from the `welcome-app` image you built.
  - The `-p 5000:5000` flag maps port 5000 on your host machine to port 5000 inside the container.
    - This allows you to access your application via `http://localhost:5000`.
  - Docker creates an isolated environment for your app with its own filesystem, network, and processes.
  - You can add optional flags:
    - `-d` to run the container in detached mode (in the background).
    - `--name my-container` to assign a custom name to the container.
    - `--rm` to automatically remove the container when it stops.
  - To see running containers, use `docker ps`.
  - To stop the container, use `docker stop <container_id_or_name>`.

## Docker image naming convention
- username/image-name
- Changing docker image name
  - `docker tag current-name new-name`

## Push Docker Image to Repository
- `docker push docker-image-name:latest`
  - Uploads your local Docker image to a remote container registry (e.g., Docker Hub, AWS ECR, Azure Container Registry).
  - `docker-image-name` should include the repository name and optionally the registry URL, e.g., `username/welcome-app`.
  - `:latest` is the tag specifying which version of the image to push. You can also use custom tags like `v1.0`.
  - Steps before pushing:
    1. Log in to the registry: `docker login` (Docker Hub) or the appropriate login command for other registries.
    2. Tag the image if needed: `docker tag welcome-app username/welcome-app:latest`.
  - After pushing, others (or your deployment environment) can pull the image using `docker pull docker-image-name:latest`.

## Pulling Docker Image
- `docker pull raymondcen/welcome-app1:latest`
  - Downloads the specified Docker image from a remote container registry (e.g., Docker Hub) to your local machine.
  - `raymondcen/welcome-app1` is the repository name in the registry.
  - `:latest` is the tag specifying which version of the image to pull. If no tag is specified, Docker defaults to `latest`.
  - After pulling, you can run the image locally using `docker run`.
  - Useful for:
    - Getting images built by others or from CI/CD pipelines.
    - Ensuring your local environment uses the same image version as production.
  - To see all images you have locally, use `docker images`.

# Dockere Compose
- Docker Compose is a tool used to **define and manage multi-container Docker applications**. Instead of manually running multiple `docker run` commands for different services, Compose allows you to describe all your containers, networks, and volumes in a single YAML file (`docker-compose.yml`) and manage them together.

### Key Uses
- **Define multiple services**: Each service (e.g., web app, database, cache) runs in its own container.
- **Simplify commands**: Start all services at once with `docker-compose up`.
- **Manage networking**: Containers can communicate by service name on a shared network.
- **Persistent storage**: Define volumes in the YAML file for data persistence.
- **Environment configuration**: Specify environment variables, build context, and ports for each service.
- **Scaling**: Easily scale services, e.g., `docker-compose up --scale web=3`.

## Dockerfile
- `FROM python:3.13-alpine`
  - Uses the Python 3.13 Alpine base image.
  - Alpine is a minimal Linux distribution, making the image lightweight.
- `WORKDIR /anyname`
  - Sets `/anyname` as the working directory inside the container.
  - All subsequent commands run in this directory.
- `ENV FLASK_APP=app_compose.py`
  - Sets an environment variable telling Flask which file contains your application.
- `ENV FLASK_RUN_HOST=0.0.0.0`
  - Configures Flask to listen on all network interfaces, allowing access from outside the container.
- `COPY . .`
  - Copies all files from the current host directory into the container’s working directory.
- `RUN pip install -r requirements.txt`
  - Installs Python dependencies listed in `requirements.txt`.
- `EXPOSE 5000`
  - Declares that the container will listen on port 5000.
  - Does not publish the port; you still need `docker run -p 5000:5000` to access it from the host.
- `CMD ["flask", "run"]`
  - Sets the default command to start the Flask application when the container runs.
  - Using JSON array form ensures proper signal handling.

## docker-compose.yml file
- `version: "3.0"`
  - Specifies the Compose file format version.
  - Version 3 is compatible with newer Docker features and swarm mode.
- `web:`
  - Defines the web application service.
  - `image: web-app`
    - Names the Docker image for this service.
  - `build:`
    - `context: .`  
      - Sets the build context to the current directory.
    - `dockerfile: Dockerfile.dev`  
      - Specifies a custom Dockerfile to use for building the image.
    - you can usually just do `build: .` when you only have 1 Dockerfile
  - `ports:`
    - `"5000:5000"`  
      - Maps port 5000 on the host to port 5000 in the container, allowing access to the Flask app.

- `redis:`
  - Defines a Redis service.
  - `image: redis`  
    - Uses the official Redis image from Docker Hub.
  - Can be accessed by other services in the Compose network (e.g., the web service) using the hostname `redis`.

- `mysql:` (commented out)
  - Example placeholder for adding a MySQL service.
  - `image: mysql`  
    - Would use the official MySQL Docker image.
- Docker Compose allows defining multiple services in a single YAML file, making it easy to manage multi-container applications.
- Services can communicate with each other over a default network created by Compose.
- To start all services:  
  ```bash
  docker-compose up
