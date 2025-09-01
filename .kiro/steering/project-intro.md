---
inclusion: always
---

# Project Introduction

This is a **Docker-based reverse proxy infrastructure project** designed for managing multiple containerized applications on an Ubuntu EC2 server. Here's the comprehensive breakdown:

## Project Purpose
The project creates a centralized nginx reverse proxy system that automatically routes traffic to different dockerized applications based on domain names. It's designed for hosting multiple apps on a single server with automatic SSL certificate management.

## Core Components

**1. Main Infrastructure (`docker-compose.yml`)**
- **nginx-proxy**: Uses `jwilder/nginx-proxy` for automatic reverse proxying
- **letsencrypt**: Uses `nginxproxy/acme-companion` for automatic SSL certificate generation
- **Network**: Creates an external `proxy-net` network for inter-container communication
- **Volumes**: Manages certificates, virtual host configs, and static files

**2. Management Scripts**
- **`restart`**: Bash script for deploying individual apps
  - Takes app name as parameter
  - Validates required docker-compose files exist
  - Stops existing containers and starts new ones
  - Uses production configuration overlay
- **`code`**: Simple utility script to launch Kiro IDE

## Architecture Pattern

**Multi-App Structure:**
```
~/apps/
├── nginx-proxied-dockerized-apps/  (this repo)
│   ├── docker-compose.yml
│   └── restart script
├── app1/
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   └── docker-compose.dev.yml
└── app2/
    ├── docker-compose.yml
    ├── docker-compose.prod.yml
    └── docker-compose.dev.yml
```

## App Requirements & Conventions

Each child application must follow these strict conventions:

**Required Files:**
- `docker-compose.yml` (base configuration)
- `docker-compose.prod.yml` (production overrides with proxy config)
- `docker-compose.dev.yml` (development overrides without proxy)

**Container Requirements:**
- Container name must match repository name
- Must connect to `proxy-net` network
- Must define `VIRTUAL_HOST` environment variable (domain)
- Must define `VIRTUAL_PORT` environment variable (internal port)
- Should include Let's Encrypt configuration for SSL

## Setup Process

**Server Preparation:**
1. Install Docker and Docker Compose on Ubuntu
2. Configure user permissions for Docker
3. Create the proxy network
4. Clone this repository to `~/apps/`

**App Deployment:**
1. Clone app repository into `~/apps/`
2. Ensure app follows the required conventions
3. Run `./restart <app-name>` to deploy

## Key Features

- **Automatic SSL**: Let's Encrypt integration for automatic certificate generation
- **Zero-downtime deployments**: Apps can be restarted independently
- **Domain-based routing**: Automatic proxy configuration based on `VIRTUAL_HOST`
- **Minimal server footprint**: Only Docker and Docker Compose required
- **Scalable**: Easy to add new applications

## Configuration Strengths

- Clean separation between infrastructure and applications
- Standardized deployment process
- Automatic service discovery through Docker labels
- Production-ready with SSL termination
- Simple but effective management scripts

## Current Project Structure

- `docker-compose.yml` - Main nginx-proxy and Let's Encrypt configuration
- `restart` - Bash script for deploying individual applications
- `code` - Utility script to launch Kiro IDE
- `README.md` - Setup and usage documentation
- `.gitignore` - Currently empty

This is a well-architected solution for managing multiple web applications on a single server with professional-grade reverse proxy and SSL management.