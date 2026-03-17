CerebraUI – Community-Driven AI Interface

CerebraUI is an open-source, modular AI platform integrating authentication (BetterAuth), Deep Research, LangFlow workflows, and parallel Web Search.
This README explains how to set up and run all components together via Docker.

System Overview

CerebraUI runs multiple interconnected services:

BetterAuth – Authentication microservice (runs separately first)

CerebraUI (Open WebUI) – Main user interface and API layer

Deep Research – Async research assistant connected through Docker network

LangFlow – Visual AI-agent workflow builder

Redis – Cache for chat performance

All services communicate through a shared Docker network called cerebra_net.

Prerequisites

Docker Desktop 4.x (includes Docker Compose)

Node.js 18+

Cloned repositories:

cerebra_ui

Final-BetterAuth

open_deep_research

Internet access for email verification (Resend API)

Cloudflare account for CAPTCHA setup

1. Run BetterAuth (Authentication Service)

BetterAuth must start before CerebraUI.

Clone and enter the BetterAuth repo, then follow its internal README instructions.

Key Steps

Create .env file (same content as .env.docker):

DATABASE_URL="postgresql+psycopg2://postgres:postgres@db:5432/cerebraui"
JWT_SECRET="super-secret-key"
EMAIL_PROVIDER_API_KEY="your-resend-api-key"
MAIL_FROM="CerebraUI <onboarding@resend.dev>"
FRONTEND_URL="http://localhost:3000"
SERVICE_PUBLIC_URL="http://localhost:4000"


Create Docker network (shared with CerebraUI):

docker network create cerebra_net


Run migrations:

npm install
npx @better-auth/cli@latest generate --config ./auth.config.mjs
npx @better-auth/cli@latest migrate --config ./auth.config.mjs


Start BetterAuth:

docker compose up -d --build


BetterAuth will now expose API on port 4000 and join cerebra_net.

2. Cloudflare Turnstile Setup (Human Verification)

CerebraUI uses Cloudflare Turnstile for CAPTCHA verification on login.

Steps

Log in to Cloudflare Dashboard
 → Turnstile → Add Widget

Add hostnames:

http://localhost:3000

http://localhost:5173

Copy the Site Key and Secret Key

Add them in .env:

VITE_TURNSTILE_SITE_KEY=<your-site-key>
TURNSTILE_SECRET_KEY=<your-secret-key>


Do not push keys to GitHub. For Docker deployments, add them in docker-compose.override.yaml.

3. Run CerebraUI + Connected Services

Ensure the folder structure:

/cerebra_ui
/open_deep_research
/final-betterauth


Deep Research is automatically built from open_deep_research when you run CerebraUI.

Run:

npm ci
NODE_OPTIONS=--max-old-space-size=4096 npm run build
OPEN_WEBUI_PORT=3000 docker compose -f docker-compose.yaml -f docker-compose.override.yaml up -d --build --force-recreate


This will:

Start Open WebUI, Deep Research, Redis, and LangFlow

Connect all services to the shared cerebra_net

Enable Turnstile verification and BetterAuth authentication

4. Docker Compose (Override)

Excerpt for clarity:

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

5. Workflow Setup (LangFlow)

If no workflows or credentials appear initially:

Open CerebraUI → Workflows

Create new LangFlow credentials

Add or import example workflows

✅ Verification Checklist
Feature	Status
BetterAuth login/signup	✅ Working
Email verification	✅ Working (Resend API)
Turnstile CAPTCHA	✅ Displayed on Sign-in
Deep Research agent	✅ Integrated
LangFlow workflows	✅ Connected
Redis caching	✅ Active
Docker network	✅ Shared (cerebra_net)
🧩 Troubleshooting

Email not sent? → Recheck EMAIL_PROVIDER_API_KEY in .env.

CAPTCHA not showing? → Verify VITE_TURNSTILE_SITE_KEY.

Auth errors? → Ensure both services share the same cerebra_net.

Slow responses? → Restart Redis or clear cache:

docker exec -it redis redis-cli flushall

🏁 Summary

To run the full system:

1️⃣ Start BetterAuth
docker compose -f ./betterauth/docker-compose.yaml up -d --build

2️⃣ Start CerebraUI (auto-runs Deep Research + LangFlow)
npm ci && NODE_OPTIONS=--max-old-space-size=4096 npm run build && \
OPEN_WEBUI_PORT=3000 docker compose -f docker-compose.yaml -f docker-compose.override.yaml up -d --build --force-recreate


CerebraUI will now be live at http://localhost:3000
, fully authenticated, research-enabled, and ready for workflows.