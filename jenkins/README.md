# Jenkins Docker Setup

This project sets up a Jenkins server in a Docker container, along with necessary tools such as .NET Core SDK, Docker CLI, Node.js, and Heroku CLI. The configuration is managed using Docker Compose.

## Prerequisites

Before you begin, ensure you have the following installed on your machine:

- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose**: [Install Docker Compose](https://docs.docker.com/compose/install/)

## Files in This Project

### 1. `docker-compose.yml`

This file defines the Jenkins service and its configurations, including the network settings, ports, volumes, and the build context.

```yaml
services:
  jenkins:
    build:
      context: .
      dockerfile: Dockerfile
    image: jenkins/jenkins:lts
    container_name: jenkins
    networks:
      eso-dev-network:
        ipv4_address: 10.10.10.2
    ports:
      - "81:8080"  # Bind port 8080 in container to 80 on host
    volumes:
      - jenkins_home:/var/jenkins_home # Persist Jenkins data
      - /var/run/docker.sock:/var/run/docker.sock # Optional: if you want Jenkins to use Docker on the host

networks:
  eso-dev-network:
    external: true

volumes:
  jenkins_home:
```

### 2. `Dockerfile`

The Dockerfile is used to customize the Jenkins Docker image by installing additional tools and dependencies, including the .NET Core SDK, Docker CLI, Node.js, and Heroku CLI.

```Dockerfile
# Use an official Jenkins image as a parent image
FROM jenkins/jenkins:lts

# Switch to root user for installing packages
USER root

# Install .NET Core SDK and other prerequisites
RUN apt-get update && \
    apt-get install -y apt-transport-https wget && \
    wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
    dpkg -i packages-microsoft-prod.deb && \
    apt-get update && \
    apt-get install -y dotnet-sdk-6.0

# Install Docker CLI
RUN apt-get update && \
    apt-get install -y docker.io

# Add NodeSource repository for Node.js 20 and install it
RUN apt-get update && \
    apt-get install -y ca-certificates curl gnupg && \
    mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install -y nodejs

# Confirm that Node.js and npm were installed
RUN node -v && \
    npm -v

RUN apt-get install -y zip jq

RUN npm install -g badlinks

# Install Heroku CLI
RUN curl https://cli-assets.heroku.com/install.sh | sh

# Switch back to Jenkins user
USER jenkins
```

## How to Build and Run Jenkins

Follow the steps below to build and run the Jenkins container.

### Step 1: Build the Docker Image

Navigate to the directory containing the `Dockerfile` and `docker-compose.yml`, and run the following command to build the Docker image:

```bash
sudo docker compose build
```

This command will build the Jenkins image using the specified `Dockerfile`.

### Step 2: Run the Jenkins Container

After the image is built, you can run the Jenkins container using the following command:

```bash
sudo docker compose up -d
```

This command starts the Jenkins container in detached mode. The Jenkins service will be accessible on port 81 of your host machine.

### Step 3: Access Jenkins

Once the container is running, you can access the Jenkins interface by navigating to the following URL in your web browser:

```plaintext
http://localhost:81
```

### Step 4: Stop and Remove the Jenkins Container

When you need to stop the Jenkins container, run the following command:

```bash
sudo docker compose down
```

This command stops and removes the Jenkins container, along with any networks and volumes created by Docker Compose.

## Persistent Data

The Jenkins home directory is persisted using a Docker volume named `jenkins_home`. This ensures that Jenkins data is retained even if the container is stopped or removed.

## Network Configuration

The Jenkins container is connected to a custom Docker network named `eso-dev-network`. It is assigned the static IP address `10.10.10.2` within this network.

## Troubleshooting

### No Output on `docker compose build`

If `sudo docker compose build` doesn't produce any output, ensure that:

1. The `Dockerfile` exists in the specified `context`.
2. The `docker-compose.yml` file is correctly configured.
3. Docker is running on your system.

You can increase verbosity by running:

```bash
sudo docker compose build --progress=plain
```

This command provides more detailed output during the build process.

### Access Issues

If you cannot access Jenkins on `http://localhost:81`, ensure:

- The container is running (`sudo docker ps`).
- The port mapping is correct in `docker-compose.yml`.
- There are no firewall or network issues on your host machine.

## Conclusion

This setup allows you to easily build and run a Jenkins server with additional tools and dependencies using Docker and Docker Compose. By following the steps outlined in this README, you can manage your Jenkins environment efficiently.

