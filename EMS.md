# Trading Execution Management System (EMS)

## Overview
An EMS for equities, interest rates, FX, and commodities across US venues with 1–10 ms end-to-end latency.  
Built in C++ for performance, modularity, and fault tolerance.

---

## Core Services

### Order Gateway and Validation
- Ingress: FIX 4.4/5.0, REST/gRPC, trader UI.
- Canonical order schema across asset classes.
- Fast pre-trade validation (price bands, notional, quantity, short-sale).
- Deterministic state machine with replay safety.

### Strategy Engine and Smart Routing
- Algos: TWAP, VWAP, POV, liquidity-seeking, dark/block routing.
- Smart Order Router (SOR): venue scoring (price, queue, fees, toxicity, latency).
- Market-aware pacing and throttles.
- In-memory market snapshots for decisioning.

### Market Connectivity
- Equities: OUCH (orders), ITCH (market data), drop-copy ingestion.
- Futures (rates/commodities): CME BOE (orders), MDP 3.0 (market data).
- FX: FIX + proprietary ECN APIs, last-look handling.
- Session manager: HA FIX/native engines, heartbeat/resend, sequence recovery.

### Data Plane and Storage
- Append-only commit log for order events.
- In-memory KV for hot order state.
- WORM/object-lock storage for audit.
- Analytics: TCA, slippage, venue scorecards, algo performance.

### Risk, Compliance, and Security
- Pre-trade risk: limits, exposure caps, fat-finger, Reg NMS checks.
- Entitlements: RBAC/ABAC for strategies and venues.
- Surveillance hooks: spoofing/layering detection, replay feeds.
- Security: mTLS, HSM-backed key storage, minimal PII.

### User Experience
- Trader console: blotter, live fills, child-order trees, hotkeys.
- Strategy studio: versioned deployment, sandboxed backtests.
- Risk dashboard: real-time limit utilization, alerts, quick actions.

---

## Latency Targets
- Ingress + normalization: 0.3–1.0 ms
- Pre-trade risk: 0.5–2.0 ms
- Strategy decision: 0.5–2.0 ms
- Adapter send: 0.2–0.8 ms
- Network + venue ack: 1–3 ms
- End-to-end client ack: ~3–8 ms (p50/p99)

---

## C++ Implementation Choices
- Networking: io_uring/epoll, optional DPDK/kernel bypass.
- Serialization: FlatBuffers/Cap’n Proto, SBE for hot paths.
- FIX engine: QuickFIX or custom minimal FIX.
- Concurrency: lock-free ring buffers, CPU affinity, NUMA pinning.
- Timing: monotonic clocks, PTP/NTP discipline.
- Storage: folly::F14 maps, Kafka/NATS JetStream off hot path, Parquet for analytics.
- Observability: Prometheus metrics, OpenTelemetry tracing, structured logs.

---

## Market-Specific Notes
### Equities
- Direct feeds + SIP, OUCH entry, ITCH data.
- NBBO-aware SOR, dark/ATS routing, Reg NMS compliance.

### Interest Rates & Commodities (Futures)
- CME BOE for orders, MDP 3.0 for market data.
- Depth-aware POV/TWAP, roll/expiry calendars.

### FX
- FIX/proprietary ECNs, last-look handling.
- Liquidity-seeking algos, venue health scoring.

---

## Deployment Slices
- **Slice A:** Equities + CME futures, OUCH/BOE adapters, FIX gateway, pre-trade risk, audit ledger.
- **Slice B:** Add FX, drop-copy reconciliation, TCA, multi-venue SOR.
- **Slice C:** Active-active co-lo (Carteret/Secaucus/CHI), surveillance hooks, advanced algos.

---

## Operational Practices
- Backpressure: per-venue queues with shed/fallback.
- Resilience: stateless workers + durable log, deterministic replay.
- Governance: versioned configs, change approvals, algo review boards.
