# CerebraUI Testing README

## Overview

This document describes the testing approach used for **CerebraUI**, an open-source, community-driven AI interface built as a secure, modular, and demonstrably reliable extension of Open WebUI. The testing strategy was designed to support a reproducible staging demo, validate critical end-to-end workflows, and provide lightweight but credible evidence for handover, debugging, and future extension.

The overall goal of testing was not large-scale production certification, but **evidence-backed validation** of the platform’s key functional and non-functional requirements, including authentication, workflow execution, web search reliability, caching behaviour, image generation, usability, and integrated system operation.

## Testing Objectives

The testing work focused on the following objectives:

- verify that all key user-facing features operate correctly in an integrated environment
- confirm that the security flow is reproducible from registration to verification to login
- validate that workflow execution, web search, and image generation are stable and interpretable
- demonstrate measurable performance improvements where applicable, especially for Redis-backed caching
- capture logs, screenshots, and latency snapshots to support reproducibility and handover

## Testing Strategy

CerebraUI followed a **layered test strategy** combining:

1. **Feature-level functional testing** for each module
2. **Smoke testing** for critical system paths
3. **End-to-end testing** across frontend, backend, and supporting services
4. **Performance and reliability checks** using latency snapshots, cache hit behaviour, concurrency observations, and fallback verification
5. **Evidence collection** through screenshots, structured logs, and metric summaries

This approach aligns with the project’s development method of incremental staging integration, modular feature workflows, and rollback-ready deployment.

## Scope of Testing

### In Scope

- reproducible end-to-end platform demo
- BetterAuth security workflow
- AI Agent Workflow and Deep Research execution
- Web Search retrieval pipeline
- Redis chat caching
- ComfyUI image generation
- frontend integration and usability verification
- basic latency and reliability evidence
- logs with correlation or request identifiers where available

### Out of Scope

- large-scale load testing
- long-duration steady-state benchmarking
- full-link chaos engineering
- compliance-grade security auditing
- enterprise SIEM integration
- multi-region or high-availability fault simulation

## Test Environment

Testing was conducted in a Docker-based environment across the project’s integrated service stack, including:

- **Frontend:** SvelteKit-based CerebraUI interface
- **Backend:** FastAPI services
- **Authentication:** BetterAuth with PostgreSQL
- **Caching:** Redis
- **Workflow orchestration:** LangFlow and Deep Research
- **Image generation:** ComfyUI through Fal API integration
- **Search pipeline:** Crawl4AI-powered parallel web search

The environment was configured for local and staging-style reproducibility, with service communication handled through the shared Docker network `cerebra_net`.

## Module-by-Module Testing

### 1. Security and Authentication

Security testing validated the full user lifecycle:

- sign-up
- email verification
- sign-in
- protected route access
- password reset
- session refresh and persistence

The team tested BetterAuth integration with PostgreSQL, Cloudflare Turnstile CAPTCHA, and email verification through Resend. The minimum acceptance condition was a complete and reproducible flow from registration to verified login, supported by API checks and frontend validation.

**Evidence collected**
- successful registration and login screenshots
- verification email flow screenshots
- protected route access checks
- logs showing successful or failed auth operations

### 2. AI Agent Workflow

Testing for the AI Agent Workflow focused on whether a chat request could be converted into a traceable asynchronous execution and then returned to the user in a readable form.

Test cases covered:

- authenticated workflow invocation
- payload validation
- execution insertion and non-blocking response
- Deep Research execution
- LangFlow execution
- status polling from frontend
- graceful handling of missing prerequisites, timeout, or API errors
- preservation of session continuity for follow-up requests

**Evidence collected**
- status transitions in logs
- execution results shown in chat
- latency observations
- error handling cases and fallback behaviour

### 3. Web Search

Web Search testing focused on correctness, concurrency, fallback, and traceability.

The search pipeline was tested for:

- cross-engine and paged concurrency
- timeout and quota controls
- fallback from primary to secondary retrieval
- deduplication and ranking
- metadata-rich output with summary and sources
- reproducible logs including engine choice, timing, and fallback decisions

Representative queries were used to validate content extraction accuracy, retrieval usefulness, and citation quality.

**Evidence collected**
- screenshots of returned summaries and sources
- logs confirming concurrency
- fallback traces
- structured output validation

### 4. Redis Chat Caching

Redis testing focused on whether repeated chat retrievals were accelerated while preserving correctness and safe fallback.

Test cases included:

- first load from database
- repeated load from cache
- TTL behaviour
- LRU behaviour
- cache invalidation path
- Redis unavailable fallback to database

The cache was configured with **LRU(3)** and **TTL = 900 seconds**, and the database remained the source of truth.

**Evidence collected**
- latency comparisons between cold and warm retrievals
- Redis TTL checks
- cache hit and miss observations
- fallback logs when Redis was unavailable

### 5. ComfyUI Image Generation

ComfyUI testing verified the text-to-image, image-to-image, and prompt analysis paths.

Test coverage included:

- text-to-image generation
- image-to-image generation
- multi-round generation continuity
- prompt analysis path
- fallback when external API calls fail
- readable error messages
- session linkage between parent and child generations

The design also validated confidence-based prompt analysis, bilingual prompts, and fallback reliability when OpenAI or Fal-related calls became unavailable.

**Evidence collected**
- generated image outputs
- session chaining validation
- success and failure logs
- confidence-based test summaries

### 6. Frontend, Branding, and Usability

Frontend testing focused on consistency, readable system states, and usability.

Checks included:

- loading, empty, error, and success state rendering
- navigation consistency
- accessibility baseline on major pages
- visual hierarchy and identity consistency
- correct integration of backend-driven features into the UI

This part of testing supported the project’s goal of making CerebraUI more learnable, clear, and suitable for collaborative or research-facing use.

**Evidence collected**
- interface screenshots
- state rendering checks
- responsive validation
- before/after comparisons

## End-to-End Testing

A reproducible end-to-end scenario was used to verify that the integrated platform worked as a complete system rather than as isolated modules.

Representative end-to-end flows included:

1. User signs up and verifies email
2. User signs in successfully
3. User opens the CerebraUI interface
4. User executes a research or workflow task
5. Web search retrieves relevant sources
6. Results are returned to the chat
7. Redis accelerates repeated retrieval
8. ComfyUI generates an image from prompt input
9. Frontend displays outputs correctly and readably

This end-to-end validation was essential to proving that the project had moved from feature prototypes to a coherent, demonstrable platform.

## Metrics and Evidence

The project relied on lightweight but meaningful evidence rather than heavy enterprise monitoring.

Examples of evidence used included:

- pass or fail status of critical paths
- screenshots of feature flows
- p50 and p95 latency snapshots where available
- cache hit observations
- concurrency logs
- retry and fallback evidence
- structured logs with request or correlation identifiers

## Acceptance Criteria

Testing was considered successful when the following conditions were met:

- staging demo could be reproduced end to end
- registration → verification → login flow worked reliably
- workflow execution produced stable and readable results
- web search returned summaries with clickable sources
- cache behaviour showed measurable benefit and safe fallback
- image generation worked across core supported paths
- documentation, screenshots, and logs were sufficient for handover

## Known Limitations

The testing approach intentionally prioritised delivery and reproducibility over heavy infrastructure testing. As a result, the following areas remain future work:

- long-run performance validation
- large-scale concurrent load testing
- full security audit and penetration testing
- role-based permission testing across multiple user tiers
- production-grade observability and alerting

## Reproducing the Tests

To reproduce testing in a local or staging-style setup:

1. start BetterAuth and supporting infrastructure
2. start CerebraUI and connected services
3. verify that all services join `cerebra_net`
4. execute the core module checks:
   - auth flow
   - search flow
   - workflow flow
   - cache flow
   - ComfyUI generation flow
5. record screenshots, logs, and latency snapshots
6. compare results against the expected behaviour described above

## Summary

CerebraUI’s testing approach was designed to support a modular, research-oriented AI platform with credible functional validation and reproducible evidence. Rather than aiming for enterprise-scale certification, the project used a layered strategy that balanced feasibility, coverage, and demonstrability. The resulting test evidence supported the team’s conclusion that CerebraUI had progressed from an inherited prototype into a more secure, observable, configurable, and research-ready platform.

