# Wolin Avatar & Server Integration Specification (`wolin.id/ns#` v0.2)

> **Status**: Active Specification · Applies to `https://wolin.id/ns#`  
> **Scope**: Multi-proportion 3D avatars (`8-head`, `6-head`, `chibi`), editable appearance recipes, ownership vs. equipment separation, and authoritative game server integration.

---

## 1. Core Principle: One Identity, Multiple Body Variants

There is **no international 3D standard for head-to-body ratios** (`8-head` vs `3-head`). Trying to force a jacket built for an 8-head realistic fighter onto a 3-head chibi mesh will always break skinning.

Following proven open-metaverse patterns (**RMRK ERC-5773 Multi-Asset** + **ERC-6220 Catalog/Slot** + **VRM 1.0 Humanoid**):
- **Identity (`webid`) is singular**: `https://alice.wolin.id/profile/card#me`
- **Body representations (`wolin:avatarVariants`) are plural**: A user can keep a default global avatar (`wolin:avatarVrmUrl` / `wolin:avatarGlbUrl`) **plus** app/proportion-specific variants inside their Pod.
- **Discovery first**: Always resolve the user's Pod root from `pim:storage` on their WebID profile card — never hardcode URL string concatenation.

---

## 2. Avatar & Appearance Vocabulary (`https://wolin.id/ns#`)

### 2.1 Profile-Level Avatar Variants (`wolin:avatarVariants`)
Apps read `wolin:avatarVariants` (falling back to `wolin:avatarVrmUrl` / `wolin:avatarGlbUrl`) and pick the best match:

```json
{
  "default": {
    "url": "https://alice.wolin.id/public/avatars/default.vrm",
    "format": "vrm-1.0",
    "skeleton": "vrm-1.0",
    "heightMeters": 1.65
  },
  "variants": [
    {
      "app": "jongno",
      "catalogId": "jongno-humanoid-v1",
      "format": "vrm-1.0",
      "skeleton": "vrm-1.0",
      "heightMeters": 1.80,
      "url": "https://alice.wolin.id/public/apps/jongno/avatar.vrm"
    }
  ]
}
```

### 2.2 3-Tier Cross-World Fallback (Graceful Degradation)
1. **Tier 1 — Exact Match (`app` or `catalogId` matches)**: Load the user's compiled VRM/GLB and/or editable `appearance.json` directly.
2. **Tier 2 — Skeleton-Compatible (`skeleton: "vrm-1.0"` matches, `catalogId` differs)**: Load the compiled VRM as a whole mesh and apply **VRM Humanoid animation retargeting**; do **not** mix incompatible clothing parts.
3. **Tier 3 — Incompatible Body / Style**: Use your world's default base body and import only compatible **headgear/props/titles + `signedCredential` badges** from the Pod.

---

## 3. Editable Appearance Recipes vs. Compiled Avatars

Keep editable part recipes private per app (`pim:storage` + `/private/apps/<app>/appearance.json`), and publish compiled models optionally:

```
<pim:storage>/
  private/apps/<app>/appearance.json   # Editable slot & color recipe (owner-only)
  public/apps/<app>/avatar.vrm         # Optional compiled VRM for cross-world display
  public/badges/<badge-id>.json        # Optional server-signed achievement credential
```

- **Catalog versioning**: Include `schemaVersion`, `app`, `catalogId`, and `catalogVersion` in `appearance.json`. If a world drops an old item ID, render a default item **without** overwriting the user's Pod JSON.
- **ETag concurrency**: Create new files with `If-None-Match: *`; update with `If-Match: <etag>`. On `412 Precondition Failed`, fetch the latest Pod state and resolve conflict rather than blindly overwriting.
- **Opaque resource URLs**: Store canonical Pod URLs (`https://alice.wolin.id/...`), never short-lived signed R2 redirect URLs, and never forward `Authorization`/`DPoP` headers across external redirects.

---

## 4. `equipped` Choice vs. Proof of Ownership (`signedCredential`)

- **`wolin:equipped` / `appearance.json` slots** express **user preference**, NOT proof of purchase or achievement.
- **Authoritative Economy & Progression**: Keep XP, levels, in-game currency, inventories, and match results in your own game server database.
- **Cross-World Badges & Tickets**: When an achievement or paid item must be verifiable in *another* world, issue a **server-signed JSON credential** (`subject: webid`, `issuer`, `issuedAt`, `expiresAt`, `signature`) for the client to store in their Pod. Other worlds verify the issuer's public key before trusting it.

---

## 5. Server-Side WebID Binding & Pod Write Patterns

### 5.1 Connecting Your Game DB to `webid`
Never trust a raw `webid` string sent from the browser:
1. Verify the user's OIDC `id_token` against `https://auth.wolin.id/.well-known/jwks.json` (or exchange `authorization_code` + PKCE on your backend).
2. Read the verified `webid` claim (e.g. `https://alice.wolin.id/profile/card#me`) and use it as the `PRIMARY KEY` / unique foreign key in your server DB (`player_economy(webid TEXT PRIMARY KEY, gold INT, xp INT)`).

### 5.2 Two Ways Game Servers Write to a User's Pod
1. **User-Delegated Access Token + DPoP**: Your backend exchanges the OIDC code with `DPoP` and stores the user's scoped `access_token` / `refresh_token` to `PUT` into `<pim:storage>/private/apps/<app>/...` on the user's behalf.
2. **Server-Signed Credential (Recommended for Badges/Tickets)**: Your backend signs a JSON credential with its own Ed25519/ES256 key and returns it to the client; the client writes it to their Pod using their existing browser session, requiring **zero** backend-to-Pod token storage.
