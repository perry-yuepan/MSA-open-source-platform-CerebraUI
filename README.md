# CerebraUI

**Community-Driven AI Interface with Authentication, Research Workflows, and Modular Service Integration**

CerebraUI is an open-source, modular AI platform designed to bring together secure authentication, agent-based research workflows, web-enabled intelligence, and an extensible user interface in a unified Docker-based environment.

This project integrates the following core components:

- **BetterAuth** for authentication and account management
- **CerebraUI / Open WebUI** as the primary user interface and API layer
- **Deep Research** for asynchronous research and agent-assisted tasks
- **LangFlow** for visual workflow orchestration
- **Redis** for response caching and performance enhancement

All services are connected through a shared Docker network named `cerebra_net`, enabling reliable service-to-service communication across the stack.

---

## Architecture Overview

CerebraUI operates as a multi-service system in which each component serves a distinct role:

| Service | Role |
|---|---|
| **BetterAuth** | Authentication microservice for sign-up, sign-in, and email verification |
| **CerebraUI / Open WebUI** | Main application interface and central API layer |
| **Deep Research** | Asynchronous research assistant connected through Docker networking |
| **LangFlow** | Visual workflow builder for AI-agent pipelines |
| **Redis** | Short-term cache for improving chat responsiveness |

### Network Design

All containers communicate through a shared external Docker network:

```bash
docker network create cerebra_net
```

This network must exist before starting the full stack.

---

## Prerequisites

Before deployment, ensure the following dependencies and resources are available:

- Docker Desktop 4.x or later (with Docker Compose support)
- Node.js 18+
- Access to the following local repositories:
  - `cerebra_ui`
  - `Final-BetterAuth`
  - `open_deep_research`
- Internet access for email verification services (for example, Resend API)
- A Cloudflare account for Turnstile CAPTCHA configuration

---

## Recommended Repository Structure

Arrange the repositories at the same directory level for predictable Docker build paths:

```text
/project-root
├── cerebra_ui
├── Final-BetterAuth
└── open_deep_research
```

This layout ensures that Deep Research and related services can be built correctly when CerebraUI is started.

---

## Step 1 — Start BetterAuth

BetterAuth must be configured and running before CerebraUI is launched.

### 1.1 Prepare the environment file

Inside the BetterAuth project, create a `.env` file using the same values expected by `.env.docker`.

```env
DATABASE_URL="postgresql+psycopg2://postgres:postgres@db:5432/cerebraui"
JWT_SECRET="super-secret-key"
EMAIL_PROVIDER_API_KEY="your-resend-api-key"
MAIL_FROM="CerebraUI <onboarding@resend.dev>"
FRONTEND_URL="http://localhost:3000"
SERVICE_PUBLIC_URL="http://localhost:4000"
```

### 1.2 Create the shared Docker network

```bash
docker network create cerebra_net
```

### 1.3 Install dependencies and run migrations

```bash
npm install
npx @better-auth/cli@latest generate --config ./auth.config.mjs
npx @better-auth/cli@latest migrate --config ./auth.config.mjs
```

### 1.4 Start BetterAuth

```bash
docker compose up -d --build
```

Once started successfully, BetterAuth will expose its API on port `4000` and join `cerebra_net`.

---

## Step 2 — Configure Cloudflare Turnstile

CerebraUI uses **Cloudflare Turnstile** to provide CAPTCHA-based human verification during authentication.

### Setup process

1. Open the Cloudflare Dashboard.
2. Navigate to **Turnstile**.
3. Create a new widget.
4. Add the following hostnames:

```text
http://localhost:3000
http://localhost:5173
```

5. Copy the generated **Site Key** and **Secret Key**.
6. Add them to your environment configuration:

```env
VITE_TURNSTILE_SITE_KEY=<your-site-key>
TURNSTILE_SECRET_KEY=<your-secret-key>
```

### Security note

Do not commit these credentials to GitHub. For Docker-based local deployments, store them in a local override file such as `docker-compose.override.yaml` or another untracked environment file.

---

## Step 3 — Build and Start CerebraUI

After BetterAuth is running and Turnstile is configured, start the main CerebraUI stack.

### 3.1 Build the frontend

```bash
npm ci
NODE_OPTIONS=--max-old-space-size=4096 npm run build
```

### 3.2 Launch the service stack

```bash
OPEN_WEBUI_PORT=3000 docker compose -f docker-compose.yaml -f docker-compose.override.yaml up -d --build --force-recreate
```

### What this command starts

This process will:

- start **Open WebUI**
- build and start **Deep Research**
- start **Redis**
- start **LangFlow**
- connect all services to `cerebra_net`
- enable BetterAuth-backed authentication and Turnstile verification

---

## Docker Compose Override Example

The following excerpt illustrates the expected configuration pattern:

```yaml
services:
  open-webui:
    environment:
      - BETTERAUTH_BASE_URL=http://betterauth-service-betterauth-1:4000
      - ENABLE_CHAT_CACHE=true
      - CHAT_CACHE_MAX_RECENT=3
      - CHAT_CACHE_TTL_SECONDS=900
      - VITE_TURNSTILE_SITE_KEY=<site-key>
      - TURNSTILE_SECRET_KEY=<secret-key>
    networks:
      - cerebra_net

  deep-research:
    build:
      context: ../open_deep_research
    ports:
      - "2024:2024"
    networks:
      - cerebra_net

  langflow:
    image: langflowai/langflow:latest
    ports:
      - "7860:7860"
    networks:
      - cerebra_net

networks:
  cerebra_net:
    external: true
```

---

## LangFlow Workflow Setup

If workflows or credentials do not appear immediately after startup:

1. Open **CerebraUI**.
2. Navigate to **Workflows**.
3. Create new **LangFlow credentials**.
4. Add or import example workflows as required.

---

## Verification Checklist

Use the following checklist after deployment:

| Feature | Expected Status |
|---|---|
| BetterAuth login and signup | Working |
| Email verification | Working |
| Turnstile CAPTCHA | Displayed on sign-in |
| Deep Research integration | Connected |
| LangFlow workflows | Connected |
| Redis chat cache | Active |
| Shared Docker network | `cerebra_net` available |

---

## Troubleshooting

### Email verification is not working
Check the value of `EMAIL_PROVIDER_API_KEY` in the BetterAuth environment configuration.

### CAPTCHA is not displayed
Verify that `VITE_TURNSTILE_SITE_KEY` is set correctly and that the allowed hostnames match your local deployment URLs.

### Authentication requests fail
Ensure that both BetterAuth and CerebraUI are attached to the same Docker network: `cerebra_net`.

### Responses are slow or stale
Redis cache may require a reset:

```bash
docker exec -it redis redis-cli flushall
```

---

## Quick Start Summary

### Start BetterAuth

```bash
docker compose -f ./betterauth/docker-compose.yaml up -d --build
```

### Start CerebraUI and connected services

```bash
npm ci && NODE_OPTIONS=--max-old-space-size=4096 npm run build && \
OPEN_WEBUI_PORT=3000 docker compose -f docker-compose.yaml -f docker-compose.override.yaml up -d --build --force-recreate
```

---

## Access Points

After successful startup, the main services should be available at:

- **CerebraUI**: `http://localhost:3000`
- **BetterAuth API**: `http://localhost:4000`
- **Deep Research**: `http://localhost:2024`
- **LangFlow**: `http://localhost:7860`

---

## Conclusion

CerebraUI provides a modular and extensible AI interface that combines authentication, workflow orchestration, research assistance, and performance optimization into a unified local deployment architecture.

With BetterAuth initialized first, Cloudflare Turnstile configured correctly, and all services attached to `cerebra_net`, the platform can be launched reliably for development, testing, and workflow experimentation.
