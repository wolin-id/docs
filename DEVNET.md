# Wolin Devnet Guide (`dev-pod.wolin.id` / `dev-auth.wolin.id`)

> **Purpose**: An isolated, zero-cost sandbox environment for developers, metaverse builders, and AI coding agents to test Solid-OIDC, DPoP, Wolin Passport, and Per-Pod LDP storage before touching Production (`wolin.id`).

---

## 1. Environment Endpoints (Production vs. Devnet)

| Component | Production (`wolin.id`) | Devnet (Sandbox) |
| :--- | :--- | :--- |
| **Pod Cloud Engine** | `https://pod.wolin.id` | `https://dev-pod.wolin.id` |
| **Passwordless Auth / Passport** | `https://auth.wolin.id` | `https://dev-auth.wolin.id` |
| **User Pod Origin** | `https://{pod_id}.wolin.id` | `https://dev-pod.wolin.id/pods/{pod_id}/` |
| **Discovery Endpoint** | `https://pod.wolin.id/.well-known/wolin` | `https://dev-pod.wolin.id/.well-known/wolin` |
| **OIDC Discovery** | `https://auth.wolin.id/.well-known/openid-configuration` | `https://dev-auth.wolin.id/.well-known/openid-configuration` |
| **Healthcheck** | `https://pod.wolin.id/healthz` | `https://dev-pod.wolin.id/healthz` |

---

## 2. Hosting Architecture (Home Server + Cloudflare Tunnel)

* **Compute**: Home Linux host (`wolin-server` native `x86_64` binary, bound strictly to `127.0.0.1:14000` with Per-Pod RocksDB isolation + `WOLIN_MAX_OPEN_PODS=128`).
* **Ingress**: Dedicated `cloudflared` tunnel (`wolin-devnet`) exposing **only** `dev-pod.wolin.id` (`127.0.0.1:14000`) and `dev-auth.wolin.id` (`127.0.0.1:14001`) over outbound QUIC/HTTP2 — zero inbound ports opened on the home router.
* **Isolation**: Completely separate storage directory (`/var/lib/wolin-devnet/data`), separate D1/SQLite state, and separate OIDC signing keys from Production (`13.125.191.189` / Cloudflare Production).

---

## 3. Sandbox Rules & Data Retention Policy

1. **No Real PII / No Production Secrets**: Never store real user personal data, production credentials, or paid assets on Devnet.
2. **Quota & Rate Limits**:
   * Default Devnet Pod quota: **100 MB** per Pod (`WOLIN_DEFAULT_POD_QUOTA_BYTES=104857600`).
   * Inactive Devnet Pods older than **14 days** may be periodically purged during sandbox resets.
3. **Reserved Sandbox Handles**:
   * `alice`, `sora`, `nova`, `luna` are pre-seeded or reserved for documentation/CI examples. Use unique test IDs such as `dev-alice-01` or `test-{your_handle}`.

---

## 4. Quickstart (5-Minute Devnet Test)

### A. Verify Devnet Engine Health
```bash
curl -s https://dev-pod.wolin.id/healthz
curl -s https://dev-pod.wolin.id/.well-known/wolin
```

### B. Provision a Devnet Test Pod (`POST /api/pods`)
```bash
curl -X POST https://dev-pod.wolin.id/api/pods \
  -H "Content-Type: application/json" \
  -d '{"pod_id":"dev-demo-01","display_name":"Devnet Demo"}'
```

### C. Connect SDKs to Devnet
* **TypeScript (`@wolin-id/client`)**:
  ```ts
  import { WolinClient } from '@wolin-id/client';
  const wolin = new WolinClient({
    oidcIssuer: 'https://dev-auth.wolin.id',
    podBaseUrl: 'https://dev-pod.wolin.id',
  });
  ```
* **Unity (`id.wolin.unity`)**: Set `WolinClientOptions.OidcIssuer = "https://dev-auth.wolin.id"` and `PodBaseUrl = "https://dev-pod.wolin.id"`.
* **Python (`wolin-id`)**: `WolinClient(oidc_issuer="https://dev-auth.wolin.id", pod_base_url="https://dev-pod.wolin.id")`.

---

## 5. For AI Coding Agents (`llms.txt` / `SKILL.md`)

When scaffolding a new metaverse or Solid app with `@wolin-id/metaverse-starter`:
1. Default `.env.development` to `VITE_WOLIN_ISSUER=https://dev-auth.wolin.id` and `VITE_WOLIN_POD_URL=https://dev-pod.wolin.id`.
2. Switch to `https://auth.wolin.id` and `https://pod.wolin.id` only in `.env.production` after E2E testing passes.
