# Decision: local-first-storage

> **One-sentence summary.** All user data lives in `wx.storage` (weapp) or `localStorage` (H5); no API, no database, no auth, no cloud sync in v1.

**Decided:** 2026-08 (project inception)
**Decided by:** project owner
**Status:** active

## Context

The product is a personal reflection journal. v1 needed to ship quickly and validate whether the Did/Bad/Thinking methodology produces user value before any infra investment. The codebase inherited from earlier exploration included `apps/api/`, `packages/database/`, `docker-compose.yml`, and references to auth / sync, but none of them were part of the v1 plan.

## Decision

All user records are persisted client-side. On weapp: `wx.storage` via `Taro.setStorage` / `Taro.getStorage`. On H5: `localStorage`. No remote API is called. No database is provisioned. No login flow exists. Backup/export is a user-initiated local action (export Markdown via `shareFileMessage`).

## Why

- Validate product value before infra cost. The methodology is the hypothesis; the product cannot be evaluated without sustained usage, and sustained usage does not require remote sync.
- Avoid 备案 / 隐私合规 / data-residency work that would block launch in mainland China.
- AI calls — when they are added — are stateless and do not need a backend.
- The cost of adding cloud sync later is bounded; the cost of building it now and finding the product is not worth syncing is open-ended.

## Consequences

- v1 cannot sync across devices. A user who switches phones loses local data unless they exported.
- All "cloud" features are out of scope: shared journals, friends, leaderboards, server-side analytics.
- The export-Markdown flow becomes a first-class feature (see `patterns/production-before-consumption.md`).
- Future versions that add sync will need to design a migration path: how local records become cloud records without losing the user's existing data.

## When to revisit

- Daily active users (DAU) reaches a level where losing data on phone loss becomes a recurring support complaint.
- The product adds a feature whose value requires cross-device sync (e.g. shared journals).
- A backup/export feature exists and is being used by ≥ X% of users regularly — that is evidence the user actually wants persistence beyond a single device.
- Regulatory or platform requirements force server-side data.

## See also

- `apps/client/src/storage/` — current storage implementation.
- `docs/LOCAL-STORAGE.md` — storage schema and backup/export format.
- `docs/PRODUCT-BRIEF.md` — product scope and roadmap.
