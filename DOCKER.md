# Docker Deployment Guide

This guide explains how to deploy Log Voyager using Docker and Docker Compose.

## Quick Start

### Using Docker Run

Pull and run the latest image:

```bash
docker pull ghcr.io/simpl-it-srl/log-voyager:latest
docker run -d -p 8080:80 --name log-voyager ghcr.io/simpl-it-srl/log-voyager:latest
```

Access the application at: http://localhost:8080

### Using Docker Compose

#### Local Development

```bash
# Build and run locally with source code mounted (hot-reload)
docker-compose -f docker-compose.dev.yml up

# Access at http://localhost:5173
```

#### Standard Deployment

```bash
# Build from local Dockerfile
docker-compose up -d

# Access at http://localhost:8080
```

#### Production Deployment

```bash
# Use pre-built image from GitHub Container Registry
docker-compose -f docker-compose.prod.yml up -d

# Access at http://localhost (port 80)
```

## Available Docker Compose Files

### `docker-compose.yml` (Default)
- **Use case**: Local testing and development
- **Image**: Builds from local Dockerfile
- **Port**: 8080
- **Features**:
  - Health checks
  - Restart policy: unless-stopped
  - Option to switch to GHCR image

### `docker-compose.dev.yml` (Development)
- **Use case**: Active development with hot-reload
- **Image**: Builds from local Dockerfile (builder stage)
- **Port**: 5173 (Vite dev server)
- **Features**:
  - Source code mounted as volumes
  - Hot-reload enabled
  - Development environment variables

### `docker-compose.prod.yml` (Production)
- **Use case**: Production deployment
- **Image**: Pre-built from GHCR (ghcr.io/simpl-it-srl/log-voyager:latest)
- **Port**: 80
- **Features**:
  - Resource limits (CPU: 1 core, Memory: 512MB)
  - Health checks
  - Restart policy: always
  - Production-optimized settings

## Building the Image

### Build locally

```bash
docker build -t log-voyager:local .
```

### Build with specific tag

```bash
docker build -t log-voyager:v1.0.0 .
```

### Multi-platform build

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t log-voyager:multiarch .
```

## Docker Compose Commands

### Start services

```bash
# Default (builds locally)
docker-compose up -d

# Development
docker-compose -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.prod.yml up -d
```

### Stop services

```bash
docker-compose down
```

### View logs

```bash
docker-compose logs -f log-voyager
```

### Rebuild and restart

```bash
docker-compose up -d --build
```

### Remove everything (including volumes)

```bash
docker-compose down -v
```

## Health Checks

All configurations include health checks that verify the application is running:

- **Interval**: 30 seconds
- **Timeout**: 10 seconds
- **Retries**: 3
- **Start period**: 40 seconds

Check health status:

```bash
docker ps
docker inspect --format='{{.State.Health.Status}}' log-voyager
```

## Environment Variables

You can customize the deployment using environment variables:

```yaml
environment:
  - NGINX_HOST=localhost
  - NGINX_PORT=80
```

## Resource Limits (Production)

The production configuration includes resource limits:

- **CPU Limit**: 1 core
- **Memory Limit**: 512MB
- **CPU Reservation**: 0.25 cores
- **Memory Reservation**: 128MB

Adjust these in `docker-compose.prod.yml` as needed.

## Networking

All configurations use a custom bridge network `log-voyager-network` for better isolation and security.

## Switching Between Local and GHCR Image

In `docker-compose.yml`, you can easily switch between building locally and using the GHCR image:

**Option 1: Build Locally (Default)**
```yaml
build:
  context: .
  dockerfile: Dockerfile
# image: ghcr.io/simpl-it-srl/log-voyager:latest  # Commented out
```

**Option 2: Use GHCR Image**
```yaml
# build:  # Commented out
#   context: .
#   dockerfile: Dockerfile
image: ghcr.io/simpl-it-srl/log-voyager:latest
```

## Troubleshooting

### Container won't start

Check logs:
```bash
docker-compose logs log-voyager
```

### Permission issues

Ensure Docker has permission to bind to the specified ports:
```bash
# Linux: May need sudo or add user to docker group
sudo usermod -aG docker $USER
```

### Port already in use

Change the port mapping in docker-compose.yml:
```yaml
ports:
  - "8081:80"  # Use port 8081 instead of 8080
```

### Image pull fails

Ensure you're authenticated to GHCR:
```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u YOUR_USERNAME --password-stdin
```

## Production Considerations

1. **Reverse Proxy**: Use Nginx or Traefik in front of the container
2. **SSL/TLS**: Configure HTTPS using Let's Encrypt
3. **Monitoring**: Add monitoring tools (Prometheus, Grafana)
4. **Backups**: Not needed (stateless application)
5. **Updates**: Use GitHub Actions to auto-deploy new releases

## Example: Production with Traefik

```yaml
version: '3.8'

services:
  log-voyager:
    image: ghcr.io/simpl-it-srl/log-voyager:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.logvoyager.rule=Host(`logvoyager.cc`)"
      - "traefik.http.routers.logvoyager.entrypoints=websecure"
      - "traefik.http.routers.logvoyager.tls.certresolver=letsencrypt"
    networks:
      - traefik-network

networks:
  traefik-network:
    external: true
```

## Support

For issues or questions:
- GitHub Issues: https://github.com/simpl-it-srl/log-voyager/issues
- Website: https://www.logvoyager.cc
