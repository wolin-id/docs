# Wolin (月印千江) Official SDK Quickstart
> **One Identity. Infinite Reflections. — Zero-Dependency Solid-OIDC, RFC 9449 ES256 DPoP & Solid 0.11 LDP Clients Across 5 Platforms**

**Web Portal**: [`https://wolin.id/docs.html`](https://wolin.id/docs.html) · [`https://wolin.id/about.html`](https://wolin.id/about.html)  
**Core Specifications**: [`ARCHITECTURE.md`](./ARCHITECTURE.md) · [`POD_CLOUD_SPEC.md`](./POD_CLOUD_SPEC.md) · [`SAAS_AND_SELF_HOSTING_STRATEGY.md`](./SAAS_AND_SELF_HOSTING_STRATEGY.md) · [`INRUPT_INTEROP.md`](./INRUPT_INTEROP.md)

이 문서는 **Wolin(월인천강 · 月印千江)** 소버린 팟 클라우드(`https://auth.wolin.id` · `https://pod.wolin.id` · `https://{handle}.wolin.id/`) 및 로컬 샌드박스(`wolin-server --in-memory`)와 연동하기 위한 **5대 공식 SDK**(`@wolin-id/client` JS/TS, Unity C#, Swift 6, Kotlin/JVM, Python 3.11+) 빠른 시작 가이드입니다.

---

## 1. Official SDK Matrix

| 플랫폼 / 언어 | 패키지 식별자 | 최소 런타임 | 저장소 | 핵심 기능 |
| :--- | :--- | :--- | :--- | :--- |
| **JavaScript / TypeScript** | `@wolin-id/client` (+ `@wolin-id/react`) | Browser, Node 20+, Edge Workers | [`wolin-id/sdk-js`](https://github.com/wolin-id/sdk-js) | 무의존성 Web Crypto `ES256` DPoP, Solid-OIDC PKCE, WebID 파서, `ETag` LDP CRUD, 안전 `302` 에셋 리다이렉트, React 18/19 훅 |
| **Unity C#** | `id.wolin.unity` | Unity 2022.3+ (`.NET Standard 2.1`) | [`wolin-id/sdk-unity`](https://github.com/wolin-id/sdk-unity) | 무의존성 `HttpClient` + `ECDsa` P-256 DPoP, WebID `pim:storage` 탐색, `If-Match` 동시성 제어, 안전 `302` 에셋 수신 |
| **Swift 6** | `WolinKit` | iOS 16+ / macOS 13+ / visionOS 1+ | [`wolin-id/sdk-swift`](https://github.com/wolin-id/sdk-swift) | Apple `CryptoKit` P-256 DPoP, `actor WolinClient` 동시성 격리, `SPARQL-Update` PATCH, 안전 `302` 리다이렉트 |
| **Kotlin / Android / JVM** | `id.wolin:sdk-kotlin` | Kotlin 2.1+ / JVM 17+ | [`wolin-id/sdk-kotlin`](https://github.com/wolin-id/sdk-kotlin) | 표준 라이브러리 전용 `SHA256withECDSA` IEEE P1363 DPoP, WebID Turtle 파서, `If-Match` / `createIfAbsent` |
| **Python** | `wolin-id` | Python 3.11+ (`asyncio`) | [`wolin-id/sdk-python`](https://github.com/wolin-id/sdk-python) | 엄격 타입(`py.typed`) `httpx` 비동기 LDP CRUD, `DpopSigner`, `provision_pod()`, 안전 `302` 에셋 수신 |

---

## 2. Prerequisites: Zero-Review App Registration & Sandbox

Wolin은 API 키 발급이나 사전 앱 심사를 요구하지 않습니다. 정적 JSON-LD 문서 1개만으로 즉시 Solid-OIDC 인증을 시작할 수 있습니다.

### 2.1 Static Client ID Document (`<origin>/client-id.jsonld`)

웹 앱 또는 3D 월드의 퍼블릭 루트(예: `public/client-id.jsonld`)에 아래 문서를 배포합니다. **`client_id`와 `redirect_uris`의 오리진(`scheme + host + port`)은 반드시 일치해야 합니다.**

```json
{
  "@context": ["https://www.w3.org/ns/solid/oidc-context.jsonld"],
  "client_id": "https://app.example.com/client-id.jsonld",
  "client_name": "My Wolin World",
  "redirect_uris": ["https://app.example.com/callback"],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"],
  "scope": "openid webid offline_access",
  "token_endpoint_auth_method": "none",
  "dpop_bound_access_tokens": true
}
```

### 2.2 Canonical Endpoints (`$0` Free Tier & Local `--in-memory`)

| 환경 | 인증 서버 (Solid-OIDC Issuer) | Pod 데이터 플레인 | 비고 |
| :--- | :--- | :--- | :--- |
| **Production (`$0` Free Tier)** | `https://auth.wolin.id` | `https://{handle}.wolin.id/` (`https://pod.wolin.id`) | 무료 `1 GiB` 쿼터, `4~32`자 핸들 즉시 발급 |
| **Local Ephemeral Sandbox** | 로컬 테스트 픽스처 / 연동 | `http://127.0.0.1:4000` | `wolin-server --in-memory --port 4000` (~6ms 기동) |

---

## 3. JavaScript / TypeScript (`@wolin-id/client` & `@wolin-id/react`)

### 설치

```bash
pnpm add @wolin-id/client
# React 18/19 프로젝트인 경우:
pnpm add @wolin-id/client @wolin-id/react
```

### Core Client (`@wolin-id/client`)

```ts
import {
  DEFAULT_WOLIN_OIDC_ISSUER,
  WolinPodClient,
  createSolidAuthRequest,
  discoverOidcProvider,
  exchangeCodeForTokens,
  fetchWebIdProfile,
  generateDPoPKeyPair,
  isValidPodId,
} from '@wolin-id/client';

// 1. 핸들 형식 검증 (3..32자, 소문자 [a-z0-9-], '--' 및 'xn--' 및 예약어 차단)
console.assert(isValidPodId('alice') === true);

// 2. WebID 프로필 역참조 -> pim:storage & solid:oidcIssuer 탐색
const profile = await fetchWebIdProfile('https://alice.wolin.id/profile/card#me');
const oidc = await discoverOidcProvider(
  profile.primaryOidcIssuer ?? DEFAULT_WOLIN_OIDC_ISSUER,
);

// 3. 비추출성(Non-extractable) ES256 DPoP 키페어 및 PKCE(S256) 인증 URL 생성
const dpopKeyPair = await generateDPoPKeyPair();
const authReq = await createSolidAuthRequest({
  authorizationEndpoint: oidc.authorization_endpoint,
  clientId: 'https://app.example.com/client-id.jsonld',
  redirectUri: 'https://app.example.com/callback',
});

// 4. 콜백 인가 코드를 DPoP 바인딩 액세스 토큰(at+jwt)으로 교환
const tokens = await exchangeCodeForTokens({
  tokenEndpoint: oidc.token_endpoint,
  code: 'authorization-code-from-callback',
  codeVerifier: authReq.codeVerifier,
  redirectUri: authReq.redirectUri,
  clientId: authReq.clientId,
  dpopKeyPair,
});

// 5. ETag 조건부 LDP CRUD 및 서명된 302 R2 에셋 안전 조회
const pod = new WolinPodClient({
  podUrl: profile.primaryStorage ?? 'https://alice.wolin.id/',
  webId: profile.webId,
  accessToken: tokens.accessToken,
  dpopKeyPair,
});

const created = await pod.putResource(
  '/private/notes.ttl',
  '<#note1> <http://purl.org/dc/terms/title> "Hello Wolin (月印千江)" .',
  { createOnly: true },
);

const note = await pod.getResource('/private/notes.ttl');
await pod.putResource(
  '/private/notes.ttl',
  '<#note1> <http://purl.org/dc/terms/title> "Updated Safely" .',
  { ifMatch: note.etag ?? created.etag ?? undefined },
);

// 302 HMAC-SHA256 서명 리다이렉트 시 외부 오리진으로 Authorization/DPoP 헤더를 유출하지 않고 안전 수신
const avatarBlob = await pod.getBlob('/public/avatar.glb');
```

### React Hooks (`@wolin-id/react`)

```tsx
import { WolinProvider, usePodResource, useWolinSession } from '@wolin-id/react';

export function App() {
  return (
    <WolinProvider
      clientId="https://app.example.com/client-id.jsonld"
      redirectUri="https://app.example.com/callback"
      defaultOidcIssuer="https://auth.wolin.id"
    >
      <NotesView />
    </WolinProvider>
  );
}

function NotesView() {
  const { isAuthenticated, session, login, logout } = useWolinSession();
  const { data, etag, save } = usePodResource(isAuthenticated ? '/private/notes.ttl' : null);

  if (!isAuthenticated) {
    return <button onClick={() => void login()}>Continue with Wolin</button>;
  }

  return (
    <div>
      <p>{session?.webId} ({etag})</p>
      <pre>{data}</pre>
      <button onClick={() => void save('<#note1> <#done> true .')}>Save</button>
      <button onClick={logout}>Sign out</button>
    </div>
  );
}
```

---

## 4. Unity C# (`id.wolin.unity`)

### 설치 (Unity 2022.3+ UPM)

`Packages/manifest.json`의 `dependencies`에 추가하거나 **Window > Package Manager > Add package from git URL...**에 입력합니다:

```json
{
  "dependencies": {
    "id.wolin.unity": "https://github.com/wolin-id/sdk-unity.git#v0.1.0"
  }
}
```

### Quickstart

```csharp
using System;
using System.Threading.Tasks;
using Wolin.Unity;

public static class SaveGameSync
{
    public static async Task SyncSaveAsync(string webId, string accessToken)
    {
        WolinClient.ValidatePodId("alice");

        using var client = new WolinClient();
        client.SetAccessToken(accessToken, webId);

        // 1. WebID 프로필 역참조, Solid-OIDC 발급자 검증 및 pim:storage 탐색
        WebIdProfile profile = await client.FetchWebIdProfileAsync(webId);
        profile.VerifyOidcIssuer("https://auth.wolin.id/");
        Uri storageRoot = await client.DiscoverStorageAsync(webId);
        Uri saveUri = new Uri(storageRoot, "games/my-rpg/save.ttl");

        // 2. 플레이어 Pod에 세이브 RDF 기록
        PodResource created = await client.PutAsync(
            saveUri,
            "@prefix ex: <https://example.org/rpg#> .\n<#player> ex:level 5 ; ex:gold 1200 .\n",
            contentType: "text/turtle");

        // 3. ETag 기반 낙관적 동시성(If-Match) 업데이트
        PodResource loaded = await client.GetAsync(saveUri);
        PodResource updated = await client.PutAsync(
            saveUri,
            "@prefix ex: <https://example.org/rpg#> .\n<#player> ex:level 6 ; ex:gold 1550 .\n",
            contentType: "text/turtle",
            ifMatch: loaded.ETag);

        // 4. 조건부 삭제
        await client.DeleteAsync(saveUri, ifMatch: updated.ETag);
    }
}
```

---

## 5. Swift 6 (`WolinKit`)

### 설치 (Swift Package Manager · iOS 16+ / macOS 13+ / visionOS 1+)

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/wolin-id/sdk-swift.git", from: "0.1.0")
],
targets: [
    .target(
        name: "MyApp",
        dependencies: [
            .product(name: "WolinKit", package: "sdk-swift")
        ]
    )
]
```

### Quickstart

```swift
import Foundation
import CryptoKit
import WolinKit

try WolinClient.validatePodID("alice")

// 1. CryptoKit P-256 기반 RFC 9449 DPoPSigner 및 Actor 안전 WolinClient 생성
let signer = DPoPSigner(privateKey: P256.Signing.PrivateKey())
let client = WolinClient(
    accessToken: "<solid-oidc-dpop-bound-access-token>",
    webID: "https://alice.wolin.id/profile/card#me",
    dpopSigner: signer
)

// 2. WebID 역참조, 발급자 검증 및 pim:storage 탐색
let profile = try await client.fetchWebIDProfile()
precondition(profile.authorizes(oidcIssuer: WolinClient.defaultAuthURLString))
let storageURL = try await client.discoverStorage()

// 3. If-None-Match: * (.createOnly) 리소스 생성
let noteURL = storageURL.appending(path: "private/notes.ttl")
try await client.putTurtle(
    at: noteURL,
    turtle: """
    @prefix dct: <http://purl.org/dc/terms/> .
    <#note1> dct:title "Hello from WolinKit" .
    """,
    etagMatch: .createOnly
)

// 4. ETag 조회 후 조건부 SPARQL-Update PATCH
let resource = try await client.getResource(at: noteURL)
if let etag = resource.etag {
    try await client.patchResource(
        at: noteURL,
        sparqlUpdate: "INSERT DATA { <#note1> <http://purl.org/dc/terms/subject> \"Solid\" . }",
        etagMatch: .ifMatch(etag)
    )
}
```

---

## 6. Kotlin / Android / JVM (`id.wolin:sdk-kotlin`)

### 설치 (Gradle Kotlin DSL · Kotlin 2.1+ / JVM 17+)

```kotlin
dependencies {
    implementation("id.wolin:sdk-kotlin:0.1.0")
}
```

### Quickstart

```kotlin
import id.wolin.sdk.DPoPSigner
import id.wolin.sdk.WolinClient

fun main() {
    check(WolinClient.isValidPodId("alice"))

    // 1. P-256 DPoP 키페어 및 클라이언트 초기화
    val signer = DPoPSigner.generate()
    val client = WolinClient(
        accessToken = "<solid-oidc-dpop-bound-access-token>",
        dpopSigner = signer,
    )

    // 2. WebID 프로필 역참조, Solid-OIDC 발급자 검증 및 pim:storage 탐색
    val webId = "https://alice.wolin.id/profile/card#me"
    val profile = client.fetchWebIdProfile(webId)
    check(profile.verifyOidcIssuer(WolinClient.DEFAULT_AUTH_URL))
    val storageRoot = client.discoverStorage(webId)

    // 3. If-None-Match: * 생성 및 If-Match 원자적 업데이트
    val noteUrl = client.resolveResourceUrl(storageRoot, "notes/hello.ttl")
    client.createIfAbsent(
        url = noteUrl,
        body = """
            @prefix dct: <http://purl.org/dc/terms/> .
            <#it> dct:title "Hello from Wolin Kotlin SDK" .
        """.trimIndent(),
    )

    val current = client.get(noteUrl)
    val updated = client.put(
        url = noteUrl,
        body = """
            @prefix dct: <http://purl.org/dc/terms/> .
            <#it> dct:title "Updated atomically with If-Match" .
        """.trimIndent(),
        ifMatch = current.etag,
    )

    client.delete(noteUrl, ifMatch = updated.etag)
}
```

---

## 7. Python 3.11+ (`wolin-id`)

### 설치

```bash
pip install wolin-id
```

### Quickstart

```python
import asyncio
import os
from wolin_id import (
    DEFAULT_AUTH_URL,
    DEFAULT_POD_URL,
    DpopSigner,
    WolinClient,
    is_valid_pod_id,
)


async def main() -> None:
    assert is_valid_pod_id("alice")
    signer = DpopSigner.generate()

    async with WolinClient(
        base_url=DEFAULT_POD_URL,  # https://pod.wolin.id
        access_token=os.environ["WOLIN_ACCESS_TOKEN"],
        dpop_signer=signer,
    ) as client:
        # 1. WebID 프로필 역참조 및 Solid-OIDC 발급자 신뢰 체인 검증
        profile = await client.fetch_webid_profile(
            "https://alice.wolin.id/profile/card#me"
        )
        profile.verify_oidc_issuer(DEFAULT_AUTH_URL)  # https://auth.wolin.id
        storage_url = profile.primary_storage
        assert storage_url is not None

        # 2. If-None-Match: * 신규 생성 (이미 존재하면 412 반환)
        note_url = f"{storage_url}private/note.ttl"
        created = await client.put(
            note_url,
            '@prefix dc: <http://purl.org/dc/terms/> .\n<> dc:title "Hello Wolin" .\n',
            content_type="text/turtle",
            create_only=True,
        )

        # 3. 리소스 조회 및 If-Match 낙관적 동시성 업데이트
        res = await client.get(note_url)
        updated = await client.put(
            note_url,
            '@prefix dc: <http://purl.org/dc/terms/> .\n<> dc:title "Updated safely" .\n',
            content_type="text/turtle",
            if_match=res.etag or created.etag,
        )

        # 4. 조건부 삭제
        await client.delete(note_url, if_match=updated.etag)


if __name__ == "__main__":
    asyncio.run(main())
```

---

## 8. Core Protocol Guardrails (Built into All 5 SDKs)

1. **`at+jwt` 액세스 토큰 강제**: `wolin-server`는 `id_token` 혼동 공격을 막기 위해 리소스 요청 시 JWT 헤더 `typ: "at+jwt"` 및 `cnf.jkt` 바인딩을 검증합니다(`401` 거부). 항상 `access_token`만 전달하십시오.
2. **RFC 9449 `htu` 정규화 & `DPoP-Nonce` 자동 재시도**: 5대 SDK 모두 서명 시 URL의 `?query`와 `#fragment`를 자동 제거하고, 서버의 `DPoP-Nonce` 챌린지 응답 시 1회 자동 재서명합니다.
3. **크레덴셜 무유출 `302 Found` 에셋 리다이렉트**: 대용량 바이너리(`>= 256 KiB`) 조회 시 반환되는 60초 만료 `HMAC-SHA256` 서명 URL(`https://auth.wolin.id/assets/{pod_id}/*`)을 따라갈 때 `Authorization` 및 `DPoP` 헤더를 자동 제거하여 교차 오리진 토큰 유출을 원천 차단합니다.
4. **LDP 컨테이너 후행 슬래시(`/`) 및 핸들 규칙**:
   * 컨테이너 URI는 반드시 `/`로 끝나야 합니다(예: `https://alice.wolin.id/private/`).
   * `pod_id` 핸들은 `3~32`자(무료 티어 `4~32`자) 소문자 영숫자·단일 하이픈만 허용하며, `xn--`·`--`·약 180개 예약어(`400`) 및 영구 Tombstone 처리된 탈퇴 핸들(`409 Conflict`)은 생성이 차단됩니다. 자세한 명세는 [`POD_CLOUD_SPEC.md`](./POD_CLOUD_SPEC.md) 및 [`ARCHITECTURE.md`](./ARCHITECTURE.md)를 참고하십시오.
