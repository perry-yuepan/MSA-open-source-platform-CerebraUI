# CerebraUI – Community-Driven AI Interface

CerebraUI is an open-source, modular AI platform built for self-hosted, extensible AI interaction. It integrates secure authentication, multimodal workflows, deep research capabilities, and parallel web search into a unified interface. This repository documents how to set up and run the full CerebraUI system using Docker.

## System Overview

CerebraUI is composed of several interconnected services:

- **BetterAuth** – authentication microservice
- **CerebraUI (Open WebUI-based)** – main user interface and API layer
- **Deep Research** – asynchronous research assistant connected through Docker networking
- **LangFlow** – visual workflow builder for AI agents
- **Redis** – cache layer for improved responsiveness

All services communicate through a shared Docker network named `cerebra_net`.

## Prerequisites

Before getting started, make sure the following are installed:

- Docker Desktop 4.x (with Docker Compose)
- Node.js 18+
- Internet access for email verification via Resend API
- A Cloudflare account for Turnstile CAPTCHA configuration

You should also have the following repositories cloned locally:

- `cerebra_ui`
- `Final-BetterAuth`
- `open_deep_research`

## Architecture

The full system runs across multiple services:

- **BetterAuth** must be started first, as it provides authentication for the main platform.
- **CerebraUI** serves as the central interface for users.
- **Deep Research** is built and connected through Docker during startup.
- **LangFlow** provides visual workflow orchestration for AI pipelines.
- **Redis** supports caching for better chat and interaction performance.

## Step 1: Start BetterAuth

BetterAuth must be running before CerebraUI starts.

First, clone and enter the BetterAuth repository, then follow its internal setup instructions.

### Environment configuration

Create a `.env` file using the same values as `.env.docker`:

```env
DATABASE_URL="postgresql+psycopg2://postgres:postgres@db:5432/cerebraui"
JWT_SECRET="super-secret-key"
EMAIL_PROVIDER_API_KEY="your-resend-api-key"
MAIL_FROM="CerebraUI onboarding@resend.dev"
FRONTEND_URL="http://localhost:3000"
SERVICE_PUBLIC_URL="http://localhost:4000"
