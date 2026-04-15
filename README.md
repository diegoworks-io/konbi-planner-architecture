This repository is private to protect sensitive organizational data. This documentation serves as a technical whitepaper for the underlying architecture.

# Konbi Planner

## Executive Summary: Solving Legacy Spreadsheet Dependency

Konbi Planner is engineered as an Operational OS for a 24/7, mission-critical staffing environment. It orchestrates the migration from legacy spreadsheets into a high-integrity digital system where scheduling, payroll adjacencies, and coverage decisions are executed with auditability and real-time clarity. The platform centralizes rosters, approvals, and operational dashboards so leadership can run staffing operations with deterministic data rather than manual reconciliation.

## Tech Stack

- **UI:** React 19, TypeScript, Vite 7, Tailwind CSS 4, Headless UI, Heroicons, Framer Motion, GSAP, React Router 7.
- **Data Layer:** Supabase (Postgres, Auth, Realtime) via `@supabase/supabase-js`, with PostgreSQL Row Level Security (RLS) enforcing privacy and transactional integrity across payroll and staffing data.
- **Tooling:** ESLint (flat config), Prettier, Vitest + Testing Library, Playwright, pnpm/npm scripts.
- **Automation:** Node-based pipelines under `scripts/pipelines/` for snapshots and Google Drive uploads, orchestrated by `scripts/run_internal.sh`.

### Structural Integrity: End-to-End Type Safety

I engineered a DB-to-UI contract by generating `src/types/database.types.ts` from the live Supabase schema via `scripts/generate-types.ts`. This keeps the data layer and UI synchronized, prevents type drift, and hardens staffing and attendance workflows against silent schema regressions.

## Deterministic State Machines

The biometric ingestion path is modeled as deterministic state machines with a strict 1-to-1 synchronization contract between device events and shift records. This logic enforces idempotent transitions, prevents ghost clock-ins, and ensures every staffing event is verifiable end-to-end from capture to payroll impact.

## CI/CD & Observability

The internal scripts function as an ETL pipeline that extracts technical telemetry (Git state, build health, and UI state snapshots), transforms it into business-readable Markdown, and loads it into an executive reporting channel. These commands act as Safe State gates that enforce merge discipline and protect mainline stability:

- `pnpm combo`: stages, commits, tags, and runs internal pipelines with upstream checks.
- `pnpm verify`: runs lint, tests, build, and format checks to validate merge readiness.
- `pnpm merge:main`: fast-forwards `main`, creates a no-ff merge commit, and pushes the canonical branch.
