# Jenkins for Docker

This repository provides a simple Jenkins container setup that can control the Docker host through the host Docker socket.

It builds a custom Jenkins image with Docker installed, mounts `/var/run/docker.sock`, and adds the Jenkins container user to the host Docker socket group using `DOCKER_GID`.

## What This Includes

- Jenkins LTS with JDK 21
- Docker installed inside the Jenkins image
- Jenkins plugins installed from `jenkins/plugins.txt`
- Docker socket access through `/var/run/docker.sock`
- Persistent Jenkins data through the `jenkins_home` Docker volume

## Requirements

- Docker
- Docker Compose
- A host environment where `/var/run/docker.sock` is available

## Quick Start

Clone this repository:

```bash
git clone <repository-url>
cd jenkins-for-docker
```

Create your local `.env` file from the example:

```bash
cp .env.example .env
```

Find the group ID of the host Docker socket:

```bash
stat -c '%g' /var/run/docker.sock
```

Update `.env` with the returned group ID:

```env
DOCKER_GID=123
```

Start Jenkins:

```bash
docker compose up -d --build
```

Open Jenkins in your browser:

```text
http://localhost:8080
```

## Getting the Initial Admin Password

After the container starts, get the initial Jenkins admin password:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Use this password to complete the Jenkins setup wizard.

## How Docker Host Access Works

The Compose file mounts the host Docker socket into the Jenkins container:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

The Jenkins container is also added to the same group as the host Docker socket:

```yaml
group_add:
  - "${DOCKER_GID}"
```

This allows Jenkins jobs to run Docker commands against the host Docker daemon, for example:

```bash
docker ps
docker build -t my-app .
docker run --rm my-app
```

## Installed Jenkins Plugins

Plugins are listed in `jenkins/plugins.txt` and installed during the image build:

```text
git
workflow-aggregator
docker-workflow
credentials
blueocean
```

To add more plugins, update `jenkins/plugins.txt` and rebuild:

```bash
docker compose up -d --build
```

## Useful Commands

View logs:

```bash
docker compose logs -f jenkins
```

Stop Jenkins:

```bash
docker compose down
```

Stop Jenkins and remove the persistent Jenkins volume:

```bash
docker compose down -v
```

## Security Note

Mounting `/var/run/docker.sock` gives the Jenkins container control over the host Docker daemon. Any Jenkins job with Docker access can effectively control containers and images on the host. Use this setup only in trusted environments.
