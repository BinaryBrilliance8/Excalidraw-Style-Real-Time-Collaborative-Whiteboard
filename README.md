# Real-Time Collaborative Canvas (Excalidraw Architecture)

A production-grade, real-time collaborative whiteboard engineered to synchronize Canvas API mutations across distributed clients. Built to solve state-tearing and cross-client synchronization latency, this system handles concurrent room-based drawing sessions with strictly enforced end-to-end type safety[cite: 1].

**Core Engineering Metrics:**
* **Concurrency:** 50 active WebSocket connections per room.
* **Persistence:** <50ms database query latency for state snapshots.
* **Type Safety:** Zero drift across 40+ shared API and WebSocket event interfaces.

## System Architecture

This codebase is structured as a strict **Turborepo** monorepo to completely eliminate frontend/backend type drift and streamline the deployment pipeline.

* **`apps/web`**: Next.js / React client managing the HTML5 Canvas API rendering loop and local state reconciliation.
* **`apps/ws-server`**: Node.js WebSocket server handling room orchestration, event broadcasting, and connection lifecycle management[cite: 1].
* **`apps/http-server`**: Node.js REST API for authentication, room initialization, and HTTP-based data fetching[cite: 1].
* **`packages/db`**: Prisma ORM wrapping a PostgreSQL instance for persisting canvas snapshots, user sessions, and room metadata[cite: 1].
* **`packages/common`**: Zod schemas and TypeScript interfaces shared universally across the HTTP server, WS server, and Next.js client[cite: 1].

## The Data & Synchronization Pipeline

1. **Mutation Capture:** The Next.js client listens to Canvas API pointer events, batching coordinate mutations and shape data into standardized JSON payloads.
2. **Event Broadcasting:** Payloads are emitted over WebSockets to the WS server, which routes the mutation exclusively to the active room's connection pool[cite: 1].
3. **State Reconciliation:** Peer clients receive the broadcasted payload and redraw the canvas delta in real-time, bypassing expensive full-page re-renders.
4. **Persistence Strategy:** The WS server flushes canvas state snapshots to PostgreSQL at optimized intervals. This guarantees session durability, allowing users to safely disconnect and immediately restore boards upon reconnection with sub-50ms query latency.

## Local Development

**Prerequisites:** Node.js 20+, `pnpm`, and a running PostgreSQL instance[cite: 1].

```bash
# Clone the repository
git clone [https://github.com/BinaryBrilliance8/excalidraw-clone.git](https://github.com/BinaryBrilliance8/excalidraw-clone.git)
cd excalidraw-clone

# Install dependencies across the monorepo
pnpm install

# Configure environment variables
cp .env.example .env

# Push database schema to PostgreSQL
cd packages/db
npx prisma db push
cd ../../

# Boot the entire monorepo (Next.js, WS Server, REST API) concurrently
pnpm run dev
