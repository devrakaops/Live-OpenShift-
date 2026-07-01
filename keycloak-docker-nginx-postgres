# Keycloak Production-Ready Setup with Docker, PostgreSQL, Nginx Reverse Proxy, and SSL

## Overview

This document provides a complete step-by-step guide for deploying a production-ready Keycloak environment on an Ubuntu EC2 instance using:

* Docker Compose
* PostgreSQL Database
* Nginx Reverse Proxy
* Let's Encrypt SSL Certificate

The setup enables secure HTTPS access to Keycloak through a custom domain.

---

# Prerequisites

Before starting, ensure the following requirements are met:

## Infrastructure Requirements

* Ubuntu Linux Server (EC2 Instance)
* Domain Name (Example: `keycloak.projectwala.site`)
* Domain A Record pointing to the EC2 Public IP

## AWS Security Group Requirements

The following ports must be open to `0.0.0.0/0` (Anywhere):

| Port | Protocol | Purpose       |
| ---- | -------- | ------------- |
| 22   | TCP      | SSH Access    |
| 80   | TCP      | HTTP Traffic  |
| 443  | TCP      | HTTPS Traffic |

---

# Step 1: System Update and Dependency Installation

Update the operating system packages and install all required software packages.

## Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

## Install Docker and Docker Compose

```bash
sudo apt install docker.io docker-compose-v2 -y
```

## Install Nginx and Certbot

```bash
sudo apt install nginx certbot python3-certbot-nginx -y
```

---

# Step 2: Docker Compose Setup

Create a dedicated directory for the Keycloak deployment.

```bash
mkdir ~/keycloak-setup && cd ~/keycloak-setup
nano docker-compose.yml
```

---

## Docker Compose Configuration

Paste the following content into the `docker-compose.yml` file.

```yaml
version: '3.8'

services:
  keycloak_db:
    image: postgres:16
    container_name: keycloak_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: YourSecurePasswordHere
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U keycloak"]
      interval: 10s
      timeout: 5s
      retries: 5

  keycloak_app:
    image: quay.io/keycloak/keycloak:26.1
    container_name: keycloak_app
    command: start --optimized
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://keycloak_db:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: YourSecurePasswordHere
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: YourAdminPasswordHere

      # Domain Settings
      KC_HOSTNAME: "keycloak.projectwala.site"
      KC_HOSTNAME_STRICT: "false"

      # Proxy Settings for Nginx
      KC_PROXY: edge
      KC_HTTP_ENABLED: "true"

    ports:
      - "8080:8080"

    depends_on:
      keycloak_db:
        condition: service_healthy

volumes:
  postgres_data:
```

---

## Configuration Explanation

### PostgreSQL Container

The PostgreSQL container provides persistent database storage for Keycloak.

#### Database Settings

```yaml
POSTGRES_DB: keycloak
POSTGRES_USER: keycloak
POSTGRES_PASSWORD: YourSecurePasswordHere
```

#### Persistent Storage

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

The database files are stored inside the Docker volume named `postgres_data`.

---

### Keycloak Container

The Keycloak container runs the Identity and Access Management (IAM) service.

#### Database Connectivity

```yaml
KC_DB: postgres
KC_DB_URL: jdbc:postgresql://keycloak_db:5432/keycloak
KC_DB_USERNAME: keycloak
KC_DB_PASSWORD: YourSecurePasswordHere
```

#### Initial Administrator Credentials

```yaml
KEYCLOAK_ADMIN: admin
KEYCLOAK_ADMIN_PASSWORD: YourAdminPasswordHere
```

#### Hostname Configuration

```yaml
KC_HOSTNAME: "keycloak.projectwala.site"
KC_HOSTNAME_STRICT: "false"
```

#### Reverse Proxy Support

```yaml
KC_PROXY: edge
KC_HTTP_ENABLED: "true"
```

#### Port Mapping

```yaml
ports:
  - "8080:8080"
```

This exposes the Keycloak service on port 8080 of the server.

---

# Step 3: Start the Containers

Launch the complete application stack.

```bash
sudo docker compose up -d
```

---

## Verify Container Status

Check whether both containers are running successfully.

```bash
sudo docker ps
```

Expected result:

* `keycloak_db` status should be **Up**
* `keycloak_app` status should be **Up**
* Port mapping should display:

```text
0.0.0.0:8080->8080/tcp
```

---

# Step 4: Configure Nginx Reverse Proxy

Nginx will receive incoming HTTP/HTTPS requests and forward them to the Keycloak container running on port 8080.

Create a new Nginx virtual host file.

```bash
sudo nano /etc/nginx/sites-available/keycloak
```

---

## Nginx Configuration

Paste the following content:

```nginx
server {
    listen 80;
    server_name keycloak.projectwala.site;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Enable the Configuration

Create a symbolic link.

```bash
sudo ln -s /etc/nginx/sites-available/keycloak /etc/nginx/sites-enabled/
```

Remove the default Nginx website if enabled.

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Validate the Nginx configuration.

```bash
sudo nginx -t
```

Restart Nginx.

```bash
sudo systemctl restart nginx
```

---

# Step 5: Configure SSL Using Let's Encrypt

Generate and install a free SSL certificate using Certbot.

```bash
sudo certbot --nginx -d keycloak.projectwala.site
```

During the installation process:

1. Enter a valid email address.
2. Accept the Terms of Service.
3. Select the option to redirect HTTP traffic to HTTPS.

Certbot will automatically:

* Generate SSL certificates.
* Update the Nginx configuration.
* Configure HTTPS on port 443.
* Redirect HTTP traffic to HTTPS.

---

## Restart Nginx After SSL Installation

```bash
sudo systemctl restart nginx
```

---

# Step 6: Access Keycloak

Once the deployment is complete, access Keycloak using the secure URL:

```text
https://keycloak.projectwala.site
```

Login using the administrator credentials configured in the Docker Compose file.

```yaml
KEYCLOAK_ADMIN: admin
KEYCLOAK_ADMIN_PASSWORD: YourAdminPasswordHere
```

---

# Architecture Flow

```text
User Browser
      │
      ▼
HTTPS (443)
      │
      ▼
Nginx Reverse Proxy
      │
      ▼
HTTP (8080)
      │
      ▼
Keycloak Container
      │
      ▼
PostgreSQL Database Container
```

---

# Useful Troubleshooting Commands

## View Keycloak Logs

```bash
sudo docker compose logs -f keycloak_app
```

---

## View PostgreSQL Logs

```bash
sudo docker compose logs -f keycloak_db
```

---

## View Nginx Error Logs

```bash
sudo tail -n 20 /var/log/nginx/error.log
```

---

## Check Whether Port 8080 Is Listening

```bash
sudo ss -tulnp | grep 8080
```

---

## Verify Running Containers

```bash
sudo docker ps
```

---

## Verify Docker Compose Services

```bash
sudo docker compose ps
```

---

## Restart the Complete Stack

```bash
sudo docker compose restart
```

---

## Stop the Complete Stack

```bash
sudo docker compose down
```

---

## Start the Complete Stack Again

```bash
sudo docker compose up -d
```

---

# Important Notes

1. Replace the following placeholders before using in production:

```text
YourSecurePasswordHere
YourAdminPasswordHere
```

2. Ensure the DNS A Record correctly points to the EC2 Public IP.

3. Verify that ports 80 and 443 are allowed in AWS Security Groups.

4. Keep backups of:

   * PostgreSQL database
   * Docker Compose file
   * Nginx configuration
   * SSL certificates

5. Never expose PostgreSQL directly to the internet.

---

# Deployment Summary

This deployment provides:

* Keycloak 26.1
* PostgreSQL 16 Database
* Docker Compose-based Management
* Nginx Reverse Proxy
* Let's Encrypt SSL Certificates
* HTTPS Secure Access
* Persistent Database Storage
* Health Check-Based Container Startup Dependency

This setup is suitable for small to medium production environments and serves as a strong foundation for integrating Keycloak with applications, Kubernetes clusters, OpenShift environments, Jenkins, Grafana, ArgoCD, and other enterprise platforms.
