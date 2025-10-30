# Introduction to Docker

## Introduction and Core Concepts

### What is Docker?
**Docker** is a platform for developing, shipping, and running applications inside isolated environments called **containers**. It packages an application along with all of its dependencies—such as libraries, configuration files, and system tools—into a single, standardized unit.

The best analogy is the **standardized shipping container**. Before these containers existed, transporting goods was chaotic. Goods of different shapes and sizes were loaded onto ships individually, making the process slow, inefficient, and prone to damage. The invention of the standardized shipping container revolutionized global trade because it allowed any type of cargo to be packed into a uniform box that could be easily handled, stacked, and transported by ships, trains, and trucks anywhere in the world.

Docker is the shipping container for software. It takes your application and all its "cargo" (dependencies) and packs it into a software "container." This container can then be run on any computer that has Docker installed, regardless of the underlying operating system or local configuration.

### Why was it created?

Docker was created to solve several persistent and costly problems in software development.

* **The "It Works on My Machine" Problem:** This is the classic scenario where a developer writes code that runs perfectly on their laptop but fails when it's moved to a testing or production environment. This happens because of subtle differences between environments, such as different operating system versions, library updates, or configuration settings. Docker eliminates this by ensuring the application runs in the exact same environment every single time.
* **Dependency Hell:** Modern applications rely on dozens of external libraries and frameworks. Sometimes, two different applications (or even two parts of the same application) might require conflicting versions of the same dependency. Managing these dependencies manually is complex and error-prone. Docker isolates each application and its dependencies, so they never conflict with each other.
* **Slow and Inconsistent Deployments:** Setting up new environments for testing, staging, and production used to be a manual, time-consuming, and inconsistent process. Docker automates this entirely. Spinning up a new, fully configured application environment is as simple as running a single command and takes seconds, not hours or days. This dramatically speeds up the development lifecycle.


### Core Architecture \& Philosophy

Docker's design is guided by a simple yet powerful philosophy: **"Build, Ship, and Run Any App, Anywhere."** The goal is to separate applications from infrastructure, enabling rapid and reliable software delivery.

At a high level, Docker operates on a **client-server architecture**.

* The **Docker Client** is the command-line tool (like `docker run ...`) that you interact with.
* The **Docker Daemon** (`dockerd`) is a background service running on the host machine that listens for API requests from the client and manages all the heavy lifting: building images, running containers, networking, and storage.

You, the developer, use the client to issue commands. The client sends these commands to the daemon. The daemon then builds, runs, and distributes your containers as requested. This separation allows you to manage a remote Docker daemon from your local machine, a key feature for modern cloud-based development.

***
## Module 2: The Core Curriculum (Beginner)

### 1. Containerization vs. Virtualization

**Virtualization** involves creating a **Virtual Machine (VM)**, which is an entire computer system emulated in software. A technology called a **hypervisor** (like VMware or VirtualBox) sits on top of the host operating system (or directly on the hardware) and allows you to run multiple, isolated **guest operating systems** on a single physical machine. Each VM has its own full-blown OS, its own kernel, and its own set of virtualized hardware.

**Containerization**, on the other hand, operates at the operating system level. Instead of virtualizing the hardware to run multiple operating systems, containerization virtualizes the operating system itself to run multiple, isolated application environments. All containers on a single host share the **same host OS kernel**. They only package the application code along with its specific libraries and dependencies (the "userspace").

#### Analogy: Houses vs. Apartments

* **Virtual Machines are like single-family houses.** Each house is a fully self-sufficient unit. It has its own foundation, plumbing, electrical system, and roof. You can build many houses in a neighborhood (the physical server), but each one requires its own complete set of infrastructure. This is resource-intensive and takes time to build.
* **Containers are like apartments in a large building.** The entire building (the host server) shares critical infrastructure: the foundation, the main water line, and the electrical grid (the host OS kernel). Each apartment is an isolated, private space (the container) with its own rooms, furniture, and decorations (the application and its dependencies). Because they share the core infrastructure, apartments are much faster to build and use resources far more efficiently than individual houses.

Here's a breakdown of the key differences:


| Feature | Virtual Machines (VMs) | Containers |
| :-- | :-- | :-- |
| **Isolation** | Full hardware-level isolation. Each VM has its own OS and kernel. | Process-level isolation. Containers share the host OS kernel. |
| **Overhead** | High. Each VM includes a full copy of an operating system. | Low. Containers only include the application and its dependencies. |
| **Startup Time** | Slow (minutes). It has to boot an entire operating system. | Fast (seconds or milliseconds). It's just starting a process. |
| **Size** | Large (several gigabytes). | Small (megabytes to hundreds of megabytes). |
| **Resource Usage** | High. Significant memory and CPU are reserved for each VM. | Low. Resources are shared more efficiently. |

### 2. Docker Architecture

**The Workflow:** You (the user) use the **Docker Client** to give a command, like `docker run nginx`. The client communicates this instruction to the **Docker Daemon**. The daemon checks if it has the `nginx` **Image** locally. If not, it pulls it from a **Docker Registry** (like Docker Hub). Once the image is local, the daemon uses it as a blueprint to create and run a **Container**.

Let's break down each component:

* **Docker Daemon (`dockerd`):** This is the brain and engine of Docker. It's a persistent background process that manages everything. Its primary responsibilities are:
    * Listening for API requests from the Docker Client.
    * Managing Docker Objects: images, containers, networks, and volumes.
    * Building images from Dockerfiles.
    * Running and supervising containers.
* **Docker Client (`docker`):** This is your primary interface to Docker. It’s a command-line tool that translates your commands into API requests and sends them to the daemon. While the `docker` CLI is the most common client, you can also use other tools that speak the Docker API. A single client can communicate with daemons on different machines.
* **Docker Registry:** This is a stateless, scalable storage system for Docker images. Think of it as a GitHub for Docker images.
    * **Docker Hub** is the default public registry where thousands of official and community-contributed images are stored.
    * You can also host your own **private registry** to store proprietary or custom images for your organization. This is a common practice for security and performance.
    * Commands like `docker pull` and `docker push` interact directly with a registry.
* **Docker Objects:** These are the various entities you create and manage with Docker. The main ones are:
    * **Images:** A read-only template containing a set of instructions for creating a container. An image includes the application code, a runtime, libraries, environment variables, and configuration files. Images are built from a special instruction file called a `Dockerfile`.
    * **Containers:** A runnable instance of an image. It is the living, breathing application. You can create, start, stop, move, and delete containers. Each container is isolated from other containers and the host machine.

***

## Module 3: The Core Curriculum (Intermediate)

### 1. The `Dockerfile`: Your Application's Blueprint

If an image is a template, the `Dockerfile` is the architectural blueprint used to create that template. It's a simple text file containing a series of sequential instructions that Docker follows to assemble your image automatically. This is the heart of automation and reproducibility in Docker.

Every `Dockerfile` starts with a base image (using the `FROM` instruction). From there, you add layers of instructions to copy your application code, install dependencies, set environment variables, and define what command should run when a container starts from the image. Each instruction creates a new "layer" in the image. Docker cleverly caches these layers, so if you only change the last few lines of your `Dockerfile`, it will reuse the unchanged layers from the cache, making subsequent builds incredibly fast.

**Analogy: A Recipe for a Cake**
>* A `Dockerfile` is like a detailed recipe for baking a cake.
>* `FROM ubuntu:22.04`: You start with a pre-made cake base (the Ubuntu OS).
>* `WORKDIR /app`: You decide to do your work on a clean countertop (setting a working directory).
>* `COPY . .`: You add your special ingredients (your application code) to the countertop.
>* `RUN pip install -r requirements.txt`: You mix the ingredients (install dependencies).
>* `CMD ["python", "app.py"]`: You write down the final instruction: "Bake at 350°F and serve" (the command to run the application).


#### Key Instructions and Code Example

Let's write a `Dockerfile` for a simple Python Flask web application.

**`app.py`:**

```python
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    # A simple greeting
    return "Hello from inside a Docker Container!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

**`requirements.txt`:**

```
Flask==2.2.2
```

**`Dockerfile`:**

```dockerfile
# Use an official Python runtime as a parent image. This provides a clean OS with Python pre-installed.
FROM python:3.9-slim

# Set the working directory inside the container to /app. All subsequent commands will run from this path.
WORKDIR /app

# Copy the file with our dependencies first. This is a best practice for layer caching.
# If requirements.txt doesn't change, Docker won't re-run the next step.
COPY requirements.txt .

# Install any needed packages specified in requirements.txt.
# The --no-cache-dir flag is used to keep the image size smaller.
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code from the current directory on the host into the /app directory in the container.
COPY . .

# Make port 5000 available to services outside this container. This is documentation for the user.
EXPOSE 5000

# Define the command to run the application when the container starts.
# This is the command that gets executed.
CMD ["python", "app.py"]
```


### 2. Building Images and Running Containers

Once you have your `Dockerfile` blueprint, you can use the `docker` CLI to execute it and manage your containers.

#### Key Commands

1. **Build the Image:** Navigate to the directory containing your `Dockerfile` and run:

```bash
# 'docker build' is the command to create an image.
# '-t my-flask-app:1.0' "tags" the image with a memorable name (my-flask-app) and version (1.0).
# '.' tells Docker to use the current directory as the "build context" (where it finds the Dockerfile and code).
docker build -t my-flask-app:1.0 .
```

2. **Run the Container:** Now, use the image you just built to start a live container.

```bash
# 'docker run' creates and starts a container from an image.
# '-d' runs the container in "detached" mode (in the background).
# '-p 5001:5000' "publishes" or maps a port. It connects port 5001 on your host machine to port 5000 inside the container.
# '--name my-web-app' gives your running container a custom, easy-to-remember name.
# 'my-flask-app:1.0' is the image to use.
docker run -d -p 5001:5000 --name my-web-app my-flask-app:1.0
```

Now, open your web browser and go to `http://localhost:5001`. You should see "Hello from inside a Docker Container!"
3. **Manage the Container:**
    * **List running containers:** `docker ps`
    * **View logs:** `docker logs my-web-app`
    * **Stop the container:** `docker stop my-web-app`
    * **Remove the container:** `docker rm my-web-app` (You must stop it first).
    * **List all local images:** `docker images`

### 3. Managing Data with Volumes

Containers are ephemeral, meaning any data written inside a container's filesystem is lost when the container is removed. This is a feature for stateless applications but a problem for stateful ones like databases. **Volumes** are Docker's mechanism for persisting data.

A volume is a special directory that lives on the host machine, managed by Docker, and is mounted into a container. The key idea is that the volume's lifecycle is completely separate from the container's lifecycle. You can stop, remove, and replace a container, and as long as you attach the same volume, the data will still be there.

**Analogy: A USB Drive for Your Container**

>Think of a **volume** as an external USB drive. You can plug it into your laptop (the container), save your work to it, and then safely eject it. Your laptop can be shut down, replaced, or upgraded, but the data on your USB drive remains untouched. You can then plug it into a new laptop and continue right where you left off.


#### How to Use Volumes

Let's run an official **PostgreSQL database** container and persist its data.

```bash
# 1. Create a named volume managed by Docker. This is the recommended approach.
docker volume create postgres-db-data

# 2. Run the PostgreSQL container, attaching the volume.
# The '-v' flag mounts the volume.
# 'postgres-db-data' is the name of the volume on the host.
# '/var/lib/postgresql/data' is the path inside the container where Postgres stores its data files.
# '-e' sets an environment variable, which is how we configure the database password.
docker run -d -p 5432:5432 --name my-database \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -v postgres-db-data:/var/lib/postgresql/data \
  postgres:14

# Now you can connect to this database, create tables, and add data.

# 3. Let's simulate a disaster. We'll stop and remove the container.
docker stop my-database
docker rm my-database

# At this point, the container is gone, but the volume 'postgres-db-data' still exists on your host.

# 4. Start a new container and re-attach the *same* volume.
docker run -d -p 5432:5432 --name my-new-database \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -v postgres-db-data:/var/lib/postgresql/data \
  postgres:14

# If you connect to this new database container, you will find all the data you created earlier is still there!
```

## Module 4: The Core Curriculum (Advanced)

### 1. Docker Networking: Connecting Containers

By default, containers are isolated. This is great for security but not very useful if your web application can't talk to its database. Docker Networking provides the solution.

When Docker is installed, it creates a few networks automatically. The most common one is the `bridge` network. Any container you run without specifying a network gets attached to this default bridge. Containers on the default bridge can communicate with each other, but it has limitations and is not recommended for production.

The best practice is to create your own **custom bridge networks**. These provide better isolation and, crucially, an automatic **DNS-based service discovery**. This means containers on the same custom network can find each other simply by using their container names as hostnames.

**Analogy: A Private Phone Network for Your Office**

>* The **default bridge network** is like a public party line. Anyone can join, and you have to know someone's specific extension number (their IP address) to call them, and these numbers can change.
>* A **custom bridge network** is like a private, internal phone system for your company. Only authorized employees (containers) can join. To call a colleague (another container), you just dial their name (the container name) into the phone, and the system automatically connects you. You don't need to know their specific extension number.


#### How to Use Custom Networks

Let's see this in action.

```bash
# 1. Create a custom bridge network
docker network create my-app-network

# 2. Run a PostgreSQL container on this network
# Note the --network flag and we are NOT publishing a port (-p)
docker run -d --name my-database --network my-app-network \
  -e POSTGRES_PASSWORD=mysecretpassword \
  postgres:14

# 3. Now, run our Flask app on the SAME network
# The app's code would need to be modified to connect to the hostname "my-database"
docker run -d -p 5001:5000 --name my-web-app --network my-app-network \
  -e DATABASE_HOST=my-database \
  my-flask-app:1.0
```

Inside the `my-web-app` container, it can now connect to the database simply by using the hostname `my-database`. Docker's internal DNS server resolves this name to the correct container's IP address.

### 2. Docker Compose: Managing Multi-Container Applications

Manually running and networking multiple containers with long `docker run` commands is tedious and error-prone. **Docker Compose** is a tool that lets you define and manage a complete application stack using a single YAML file called `docker-compose.yml`.

With Docker Compose, you declare all your services (web server, database, etc.), networks, and volumes in one file. Then, with a single command (`docker-compose up`), Compose will create the networks, build the images, create the volumes, and start all the containers in the correct order.

This is **declarative infrastructure**. You declare the desired *state* of your application, and Docker Compose figures out how to make it happen.

#### Code Example: `docker-compose.yml`

Let's define the Flask app and PostgreSQL database from our previous examples in a single file.

**`docker-compose.yml`:**

```yaml
# Specify the Compose file version. Version 3.8 is a good modern choice.
version: "3.8"

# Define all the services (containers) that make up your application.
services:
  # The web server service
  web:
    # Build the image from the Dockerfile in the current directory.
    build: .
    # Name the container for easy reference.
    container_name: flask-web-app
    # Map port 5000 on the host to port 5000 in the container.
    ports:
      - "5000:5000"
    # Mount the named volume 'app-code' to the '/app' directory for development (live code reload).
    volumes:
      - app-code:/app
    # Define environment variables. Here we tell Flask where to find the database.
    environment:
      - DATABASE_HOST=db
    # This service depends on the 'db' service. Compose will start 'db' before starting 'web'.
    depends_on:
      - db

  # The database service
  db:
    # Use the official PostgreSQL 14 image from Docker Hub.
    image: "postgres:14"
    container_name: postgres-db
    # Define environment variables required by the postgres image.
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
    # Mount a volume to persist database data.
    volumes:
      - postgres-data:/var/lib/postgresql/data

# Define the named volumes used by the services.
volumes:
  postgres-data:
  app-code:
```

With this file in your project directory, you can simply run:

* `docker-compose up -d`: To start the entire application stack in detached mode.
* `docker-compose down`: To stop and remove all containers, networks, and volumes defined in the file.


### Module 5: Expert - Interview Mastery

### Common Interview Questions (Theory)

1. **What's the difference between an image and a container?**
   > An **image** is a read-only, inert template containing the application and its dependencies. It's a blueprint. A **container** is a live, running instance of an image. It's the actual execution environment. You can have many containers running from a single image.
2. **Explain the Docker build cache. How can you optimize it?**
   > Docker builds images in layers, and it caches each layer. When you rebuild an image, Docker reuses unchanged layers from the cache, making builds faster. To optimize it, order your `Dockerfile` instructions from least to most frequently changing. For example, copy the `package.json` or `requirements.txt` and install dependencies *before* copying the application source code, as dependencies change less often.
3. **What is the difference between `CMD` and `ENTRYPOINT` in a `Dockerfile`?**
   > Both specify the command to be executed when a container starts. `ENTRYPOINT` configures a container that will run as an executable. Arguments passed to `docker run` are appended to the `ENTRYPOINT`. `CMD` provides default arguments for an `ENTRYPOINT` or the default command to execute if there is no `ENTRYPOINT`. `CMD` can be easily overridden by providing a command to `docker run`. A common pattern is to use `ENTRYPOINT` to set the executable and `CMD` to set the default flag.
4. **Why should you use custom bridge networks over the default bridge?**
   > Custom networks provide better isolation as only containers explicitly attached can communicate. Most importantly, they offer automatic DNS service discovery, allowing containers to resolve each other by name (`http://web-server:80`), which is more robust than relying on changing IP addresses.
5. **How do you manage sensitive data like passwords or API keys in Docker?**
   > The worst way is to hardcode them in the `Dockerfile`. A better way is to pass them as environment variables (`-e` flag), but these are still visible with `docker inspect`. The best practice is to use **Docker Secrets**, which are encrypted and only mounted into the container's memory at runtime, making them far more secure.
6. **What's a multi-stage build and why would you use it?**
   > A multi-stage build uses multiple `FROM` statements in a single `Dockerfile`. It allows you to use one image with all the build tools and compilers to create your application artifact (e.g., a compiled binary) and then copy *only that artifact* into a final, clean, and much smaller production image without any of the build dependencies. This drastically reduces the image size and attack surface.
7. **Explain the difference between a Docker volume and a bind mount.**
   > Both persist data. A **volume** is fully managed by Docker, stored in a dedicated area on the host filesystem, and is the recommended approach for production. A **bind mount** maps a specific file or directory from the host machine into the container. Bind mounts are useful for development when you want to see code changes reflected live in a container, but they are less portable as they depend on the host's directory structure.
8. **What problem does Docker solve?**
   > It solves the "it works on my machine" problem by bundling an application and all its dependencies into a standardized, portable container. This ensures consistency across development, testing, and production environments, eliminates dependency conflicts, and accelerates the development lifecycle.
9. **Can a container run without an operating system?**
   > No. This is a common misconception. A container does not bundle its own OS, but it *shares the kernel of the host operating system*. It requires a host OS (like Linux or Windows) to be running underneath it.
10. **What does the `-p 8080:80` flag do in `docker run`?**
   > It maps port `8080` on the host machine to port `80` inside the container. This allows external traffic reaching the host on port `8080` to be forwarded to the application running on port `80` within the isolated container environment.

### Common Interview Questions (Practical/Coding)

1. **Task:** Write a `Dockerfile` for a simple Node.js application that uses a multi-stage build for optimization.

    **Solution:**
    
    ```dockerfile
    # --- Build Stage ---
    # Use a full Node image that includes all build tools
    FROM node:16 as builder
    
    WORKDIR /usr/src/app
    
    # Copy package files and install dependencies
    COPY package*.json ./
    RUN npm install
    
    # Copy application source code
    COPY . .
    
    # Build the application (e.g., for a TypeScript or React app)
    RUN npm run build
    
    # --- Production Stage ---
    # Use a slim, lightweight base image for the final product
    FROM node:16-alpine
    
    WORKDIR /usr/src/app
    
    # Only copy the necessary production dependencies and build artifacts
    COPY package*.json ./
    RUN npm install --only=production
    
    # Copy the built application from the 'builder' stage
    COPY --from=builder /usr/src/app/dist ./dist
    
    EXPOSE 3000
    CMD [ "node", "dist/index.js" ]
    ```
    
    **Thought Process:** The goal is a small, secure production image. The `builder` stage has the full `npm` toolkit to install `devDependencies` and build the app. The final stage starts from a minimal `alpine` image and only copies the final build output (`dist` folder) and production dependencies. This leaves all the heavy build tools behind. 
<br><br>
2. **Task:** You have a container that is starting and then immediately exiting. What steps would you take to debug it?

    **Solution:**
    
    ```bash
    # 1. First, check the logs. This is the most common source of errors.
    # The 'docker logs' command shows the stdout/stderr from the container's main process.
    docker logs <container_name_or_id>
    
    # 2. If the logs aren't helpful, maybe the command is wrong. Run the container
    # but override its default command with an interactive shell.
    # This keeps the container alive so you can poke around inside.
    docker run -it --entrypoint /bin/sh <image_name>
    
    # 3. If it's a networking issue, inspect the container's configuration to
    # see its IP address, the networks it's connected to, and its environment variables.
    docker inspect <container_name_or_id>
    
    # 4. For a container that is already running but misbehaving, you can get a shell
    # inside it without stopping it.
    docker exec -it <container_name_or_id> /bin/sh
    ```
    
    **Thought Process:** Debugging follows a logical flow. Start with the most likely problem (application error logs). If that fails, bypass the application startup to inspect the container's internal state. Finally, use `inspect` for configuration issues and `exec` for live debugging.

3. **Task:** Write a `docker-compose.yml` to run a Redis cache service and a Python worker that connects to it.

    **Solution:**
    
    ```yaml
    version: "3.8"
    services:
      # The Redis caching service
      cache:
        image: "redis:6-alpine"
        container_name: my-redis-cache
        # Expose port for local connection if needed, but not necessary for inter-container communication
        ports:
          - "6379:6379"
    
      # The Python worker application
      worker:
        build: ./worker_app
        container_name: my-python-worker
        # The worker needs to know the hostname of the cache service.
        # Docker Compose automatically creates a network and makes 'cache' a valid hostname.
        environment:
          - REDIS_HOST=cache
        # Ensure the cache service is running before the worker starts.
        depends_on:
          - cache
    ```
    
    **Thought Process:** The key here is service discovery. I define two services, `cache` and `worker`. The `worker`'s Python code will be configured (via the `REDIS_HOST` environment variable) to connect to the hostname `cache`. Docker Compose automatically handles the networking so that the name `cache` resolves to the Redis container's IP address. `depends_on` ensures a graceful startup order.

---