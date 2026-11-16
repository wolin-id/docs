# Retired Proposal: Public Devnet (Superseded by Local `--in-memory` & `$0` Free Tier)

> **Status**: Retired on 2026-11-15 under YAGNI. Wolin (月印千江) does **not** operate a separate public devnet domain.

---

## Canonical Developer & Agent Testing Paths

1. **Local Ephemeral Sandbox (`wolin-server --in-memory` — `$0`, Offline-Capable)**:
   ```bash
   wolin-server --in-memory --port 4000
   curl -fsS http://127.0.0.1:4000/healthz
   curl -fsS http://127.0.0.1:4000/.well-known/wolin
   ```
   Starts a full W3C Solid 0.11 LDP + WAC/ACP + Native MCP runtime in ~6 ms (~15.3 MiB RSS) with zero external services or disk state.

2. **Cloud `$0` Free Tier (`1 GiB` Sovereign Pod)**:
   - **Solid-OIDC Issuer**: `https://auth.wolin.id`
   - **Pod Cloud Engine**: `https://pod.wolin.id` & `https://{handle}.wolin.id/`
   - Every developer can claim a free `4–32` character handle with a `1 GiB` storage quota and integrate directly via a static `<origin>/client-id.jsonld` document.
