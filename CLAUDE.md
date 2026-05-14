# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AI Reliability Scanner** — A full-stack TypeScript application that evaluates the reliability, security, and hallucination risks of AI systems. It runs a comprehensive suite of tests (security, chaos, tool validation, memory, browser, load, and hallucination detection) against a target AI service and produces a production readiness score.

**Stack:** React 19 + TypeScript + Express + Vite + Tailwind CSS v4 + Google Gemini API

## Quick Start Commands

| Command | Purpose |
|---------|---------|
| `npm install` | Install dependencies |
| `npm run dev` | Start dev server with hot reload (http://localhost:3000) |
| `npm run build` | Build frontend (Vite) + bundle server (esbuild) to `dist/` |
| `npm start` | Run production server from `dist/server.cjs` |
| `npm run lint` | Type check with TypeScript (no emit) |
| `npm run preview` | Preview production build locally |
| `npm run clean` | Remove `dist/` and build artifacts |

## Configuration

**Environment Variables** (see `.env.example`):
- `GEMINI_API_KEY`: Required for AI hallucination detection via Google Gemini API
- `APP_URL`: The host URL for self-referential links and OAuth callbacks (auto-injected in AI Studio)
- `NODE_ENV`: Set to `"production"` for production builds; `"development"` enables Vite middleware

**TypeScript Path Alias:**
- `@/*` resolves to project root (e.g., `@/src/engine/runner`)

## Architecture

### Server Entry Point (`server.ts`)
- Express server listening on port 3000
- **Main endpoint:** `GET /api/scan?url=<targetUrl>&auth=<optional-header>`
  - Streams test results as Server-Sent Events (SSE)
  - Orchestrates 7 test phases in sequence
  - Returns scores (security, reliability, chaos_resilience, tool_integrity, hallucination_risk, production_readiness) and findings
- **Health endpoint:** `GET /api/health`
- Vite middleware integration for dev; static file serving for production

### Frontend (`src/`)
- `main.tsx` — React entry point
- `App.tsx` — Main UI component for scan interface
- Tailwind CSS v4 (via `@tailwindcss/vite` plugin)
- Lucide React icons + Motion animations

### Test Engine (`src/engine/runner.ts`)
Exports test functions called by server.ts orchestrator:
- `runSecurityTest()` — Checks for security vulnerabilities
- `runLoadTest()` — Stress tests reliability under load
- `runChaosTest()` — Tests chaos resilience
- `runHallucinationTest()` — Semantic validation using Gemini API
- `runToolValidationTest()` — Validates tool/function integrity
- `runMemoryTest()` — Tests state management and memory leaks

Each test function:
- Receives `(targetUrl, headers, emit)` parameters
- Uses the `emit(type, data)` callback to stream SSE messages (`"log"`, `"status"`, `"error"`)
- Returns `{ score: number, findings: any[], risk?: string, tool_integrity?: number }`

### Build Process
- **Frontend:** Vite bundles React + Tailwind into `dist/` with HMR in dev
- **Server:** esbuild bundles `server.ts` to CommonJS (`dist/server.cjs`) with external dependencies
- **Dev:** `tsx` runs TypeScript directly without compilation
- **Production:** Node runs prebuilt `dist/server.cjs`; Express serves frontend from `dist/`

## Key Design Patterns

- **SSE Streaming:** Test phases emit real-time status and log updates to clients
- **Modular Test Phases:** Each test is isolated; runner orchestrates sequencing and score aggregation
- **Score Blending:** Final scores combine individual test results with penalty adjustments (e.g., high hallucination risk reduces production_readiness by 20 points)
- **Vite + Express:** Vite handles development HMR; production builds static assets for Express to serve

## Testing & Validation

- Type checking: `npm run lint` (no emit, just validation)
- Manual testing: Start with `npm run dev` and navigate to http://localhost:3000
- To test the scan API: Call `/api/scan?url=<your-target-url>` in a GET request or use the UI
- For local testing without Gemini API: Mock the `runHallucinationTest()` function or set a valid `GEMINI_API_KEY`

## Common Development Tasks

**Adding a new test phase:**
1. Export a new function from `src/engine/runner.ts` following the pattern `(targetUrl, headers, emit) => Promise<{ score, findings, ... }>`
2. Import and call it in `server.ts` after the appropriate phase
3. Update status phase names and emit appropriate status/log messages
4. Blend scores into `finalScores` and aggregate into `production_readiness`

**Updating the frontend:**
- Vite hot reload is enabled in dev mode (see `vite.config.ts`)
- Tailwind CSS is compiled at runtime in dev; for production, Tailwind is bundled by Vite
- Use `@/` alias for imports from project root

**Debugging SSE streams:**
- Check browser DevTools Network tab for `GET /api/scan` requests
- SSE messages appear as "EventStream" responses with real-time data chunks
- Server logs appear in the terminal where `npm run dev` is running

## Deployment (AI Studio)

- App is configured for Google Cloud Run deployment via AI Studio
- Secrets (GEMINI_API_KEY, APP_URL) are injected at runtime via AI Studio UI
- Production build uses precompiled assets from `dist/` directory
- Disable HMR in cloud environments via `DISABLE_HMR=true` env var
