# WOLIN (月印千江) — Business Plan & Investment Memorandum

> **Tagline**: *"One Identity. Infinite Reflections."* (달은 하나이나 천 개의 강물에 비친다.)  
> **Company**: Entropy Paradox, Inc. (엔트로피패러독스 주식회사) · Co-CEOs: Yongcheol Choi, Jungmo Kang  
> **Business Reg. No.**: 264-88-01993 · **HQ**: 79-4 Duteopbawi-ro, Yongsan-gu, Seoul, Republic of Korea  
> **Service**: `https://wolin.id` · **Organization**: `https://github.com/wolin-id`  
> **Date**: November 15, 2026

---

## 1. Executive Summary (핵심 요약)

**Wolin(월인)**은 웹·3D 메타버스·AI 앱이 파편화될 때마다 유저가 매번 새로 가입하고 아바타·프로필·데이터를 처음부터 다시 쌓아야 하는 **플랫폼 종속(Platform Lock-in) 문제**를 해결하는 **차세대 Sovereign Pod Cloud & 메타버스 아이덴티티 인프라**다.

* **핵심 기술**: W3C Solid 0.11 + Solid-OIDC + DPoP(RFC 9449) + WAC/ACP 권한 엔진을 **단일 `<30MB` Rust 바이너리(`wolin-engine`)**로 구현해 기존 JVM/Node 기반 구현체 대비 압도적 메모리·지연시간 효율을 달성했다.
* **대중화 돌파구**: 기존 Solid(Inrupt 등)가 패스키/복잡한 개발자 UX로 일반인 유입에 실패한 것과 달리, Wolin은 **Passwordless 6대 소셜(Google·Apple·Kakao·Naver·GitHub·Discord) 1:N 자동 병합 + 링크트리형 디지털 여권(`{pod_id}.wolin.id`) + 5개 언어 공식 SDK**를 제공한다.
* **초고마진 인프라**: **Cloudflare 90%(Pages/Workers/D1/R2 Egress $0/Realtime SFU) + AWS 서울 Graviton EC2 10%** 하이브리드 구조로 Phase 1(가입자 3~5만 명) 기준 **월 ~$35~50**로 운영 가능하다.
* **다중 수익 모델**: ① SaaS 구독(Pro/Creator/Enterprise) ② 3자 레어 핸들 & 개인·기업 커스텀 도메인 ③ Stream Boost(방송 대역폭 후원) ④ **Wolin Spatial AdSense**(3D 메타버스 시야 기반 Zero-PII 광고 네트워크).

---

## 2. Problem & Market Opportunity (문제 정의 & 시장 기회)

1. **유저의 고통**: 메타버스·커뮤니티·커머스 앱이 바뀔 때마다 닉네임, 3D 아바타(`.glb`/VRM), 인맥, 구매 기록을 새로 만들어야 하며, 서비스가 종료되면 데이터도 함께 사라진다.
2. **개발자/앱사의 고통**:
   * 모든 신규 앱이 매번 자체 회원 DB, 소셜 로그인, 파일 서버, 개인정보보호(GDPR/개보법) 보안 책임을 떠안아야 한다.
   * 신규 유저가 3D 아바타와 프로필을 꾸미는 과정에서 이탈률(Cold Start Drop-off)이 높다.
3. **기존 대안(Inrupt / Web3)의 한계**:
   * **Inrupt(Solid 선구자)**: 2025년 `inrupt.net → PodSpaces` 강제 이주 과정에서 `.acl` 유실과 절대 URI 깨짐으로 신뢰를 잃었고, 일반인 친화 UX·메타버스 SDK가 부재하다.
   * **Web3/크립토 아이덴티티**: 가스비·지갑 서명·투기적 이미지로 인해 일반 웹·모바일·B2B 도입 장벽이 높다.
4. **Wolin의 해답**: **제로 코인·제로 가스비·순수 암호학(Ed25519/ES256 DPoP)** + **1초 소셜 로그인** + **원클릭 무손실 이사(`wolin export/import`)**.

---

## 3. Product & Technology Architecture (제품 및 기술 구조)

### 3.1 Open-Core 3-Repo + 5-SDK 생태계
* **`wolin-id/wolin` (Open Core · MIT/Apache-2.0)**: Rust AOT 엔진 (`wolin-core`, `wolin-identity`, `wolin-policy`, `wolin-storage`, `wolin-mcp`, `wolin-server`). 홈서버 셀프호스팅과 SaaS 데이터 플레인을 단일 코드베이스로 통합.
* **`wolin-id/cloud` (Private SaaS Control Plane)**: Cloudflare Workers + D1 기반 Passwordless Solid-OIDC Auth (`auth.wolin.id`), 1:N 소셜 계정 자동 병합, Consent UI, Airwallex 결제, 커스텀 도메인 관리.
* **5대 공식 SDK**: TypeScript/React (`sdk-js`), Unity C# (`sdk-unity`), Swift (`sdk-swift`), Kotlin (`sdk-kotlin`), Python (`sdk-python`).
* **AI 코딩 에이전트 친화 DevEx**: `/llms.txt`, `/llms-full.txt`, `wolin-id/skills`, Context7, `docs.wolin.id/mcp`를 통해 전 세계 개발자의 Claude/ChatGPT가 환각 없이 10줄 연동을 생성한다.

### 3.2 핵심 차별화 기능
1. **Wolin Passport (`{pod_id}.wolin.id`)**: 브라우저로 열면 링크트리형 3D 아바타·소셜 명함 + 앱 권한 1클릭 차단 스위치로 동작하고, 기계(`Accept: text/turtle`) 요청 시 표준 Solid RDF를 반환한다.
2. **무심사 5분 개발자 연동**: Solid-OIDC `Client ID Document (JSON-LD)` 방식으로 API 키 발급·앱 심사 없이 어떤 앱이든 즉시 연동 가능.
3. **`https://wolin.id/ns#` 메타버스 공식 RDF 스키마**: `avatarGlbUrl`, `avatarVrmUrl`, `worldLinks`, `wearableItem` 표준화로 서로 다른 앱 간 아바타·인벤토리 100% 호환.
4. **무손실 Pod 이주**: `.tar.zst` 번들 export/import 시 `.acl` 보존 + RDF 내부 URL 자동 치환 + `pim:storage` 포인터 전환으로 진정한 데이터 주권 보장.

---

## 4. Infrastructure & Unit Economics (인프라 & 단위 경제성)

### 4.1 4계층 SaaS 격리 (MicroVM 배제 → 고마진)
유저별 VM 격리(64~128MB/유저 RAM 폭탄) 대신 **① 서브도메인 Origin 격리(`{pod_id}.wolin.id`) ② Per-Pod Oxigraph DB 파일 분리 + LRU 캐시 ③ OpenDAL Cloudflare R2 `pods/{pod_id}/` Prefix + 단기 Presigned/HMAC 보안 서빙 ④ 쿼터·Rate Limit** 4계층으로 격리한다. *(B2B 대기업·공공만 Dedicated Single-Tenant 예외 제공)*

### 4.2 하이브리드 인프라 & 수용량 시나리오 (Cloudflare 90% + AWS 서울 EC2 10%)
* **바이너리/에셋**: **Cloudflare R2** (송신료 Egress **$0** — 10만 명 월 10TB 전송 시 S3 대비 월 ~$1,200 절감)
* **시맨틱 RDF 메타데이터**: **AWS 서울 Graviton EC2 + `gp3` EBS** (유저당 0.2~1MB → 10만 명 기준 20~100GB, 월 ~$12)
* **EC2 수용량**:
  * **Phase 1 (`t4g.medium` · ~$28/월)**: 동시 Hot Pod ~800개 / **총 가입자 3만~5만 명** (R2·EBS 포함 총 월 ~$35~50)
  * **Phase 2 (`m7g.large` · ~$65/월)**: 동시 Hot Pod ~2,000개 / **총 가입자 10만~15만 명**
  * **Phase 3 (`m7g.xlarge` × N · ~$130/대)**: 대당 **25만~30만 명** (Pod ID 샤딩 + 30일 미접속 휴면 Pod R2 Cold-Tier `.tar.zst` 오프로딩)

### 4.3 3단 비용 최적화 메타버스 미디어 전략
1. **일방향 대형 방송·공연·강의 ($0)**: 유튜브 라이브·치지직·트위치 URL 임베딩 (`CSS3DRenderer` / WebView)
2. **소규모(1:1~4인) 양방향 ($0)**: WebRTC P2P + 무료 STUN
3. **대규모·비공개 초저지연 방송**: **Cloudflare Realtime SFU** ($0.05/GB, 월 1TB 무료) + 방송자 선불 크레딧(Hard Cap) / 앱사 BYOK

---

## 5. Monetization & Go-To-Market (수익 모델 & 시장 전략)

### 5.1 4대 수익 엔진 (`Airwallex` 글로벌 결제 기반)
| 수익원 | 가격 / 구조 | 핵심 마진 포인트 |
| :--- | :--- | :--- |
| **1. Pod Cloud 구독** | Free ($0, 1GB, 4+자) / **Pro ($9/월, 50GB)** / **Creator ($29/월, 200GB+100GB SFU)** / Enterprise | R2 Egress $0 + Rust 경량 런타임으로 80%+ 매출총이익률 |
| **2. 핸들 & 커스텀 도메인** | **3자 레어 핸들**(유료 전용) + **개인·기업 커스텀 도메인** (Cloudflare for SaaS, 원가 $0.10/도메인) | 희소성 기반 고마진 업셀 + 브랜드 DNS TXT 소유권 검증 승인제 |
| **3. Stream Boost (방송 연장 후원)** | 시청자가 카드 결제로 **방송 방의 SFU 대역폭 크레딧을 직접 결제·충전** (Discord Boost형) | 현금 정산 규제 회피(1단계) → 추후 Airwallex Payouts/토스 지급대행(2단계) |
| **4. Wolin Spatial AdSense** | 3D 전광판·방송 배너·`.glb` 스폰서 오브젝트 (호스트 60~70% / 유저 크레딧 15~20% / 플랫폼 15~20%) | Zero-PII 공간 맥락 + Smart Pricing + `impression_token` + Idle/Trust Score 방어 |

### 5.2 서브도메인 보호 & 브랜드 승인 정책
* **무료 4~32자 / 유료 3자 / 1~2자 영구 잠금** (`xn--`, `--` 금지).
* **2계층 사전 잠금**: RFC 2142 시스템(~400개) + Tranco/Radar 활성 트래픽 Top 브랜드(일반 이름 오탐 방지를 위해 단순 `.com` 배제) + 한국 주요 기업/금융 + Leetspeak 정규화 + 탈퇴 계정 Tombstone 영구 동결.

### 5.3 광고주 모집 GTM (3단계)
1. **0단계 (빈 슬롯 0% 백필)**: Wolin 자체 홍보 + 제휴(CPA: 게이밍 기어·3D 에셋·AI 툴)로 초기 소규모 트래픽부터 수익화.
2. **1단계 (첫 유료 광고주)**: 기존 CPC 단가 상승에 고민하는 인디 게임·버튜버·AI/개발자 SaaS 대상 **10만 원 소액 CPC/CPA 테스트** 유치.
3. **2단계 (스케일업)**: 엔트로피패러독스 기존 브랜드 파트너 파일럿 레퍼런스 확보 후 **`ads.wolin.id` 셀프서브** 오픈.

---

## 6. Current Traction & Execution Roadmap (현재 진행 성과 & 로드맵)

### ✅ 완료된 핵심 마일스톤 (2026-11-15 기준)
* **Wolin Rust 엔진 (`wolin-id/wolin`)**: 6대 크레이트 구현, Rust 2024 + Oxigraph 0.5 + OpenDAL 0.59 최신화(63/63 테스트 통과), **홈서버 셀프호스팅 패키지(`t_a9fe24f9`)** 완성.
* **공식 패키지 네임스페이스 선점**: `crates.io` 7종(`wolin*`) + `npm` (`wolin`, `@wolin-id/client`) + `ghcr.io/wolin-id/wolin` 확보 완료.
* **4대 SDK 완성 (`sdk-{unity,swift,kotlin,python}`)**: Unity UPM, Swift SPM, Kotlin Gradle, Python 패키징·테스트·CI 푸시 완료 (`sdk-js` 및 `wolin-id/cloud` 최종 리뷰 마무리 단계).
* **웹 포털 & 결제 심사 준비 (`https://wolin.id`)**: Vercel/Stripe 스타일 공식 포털 라이브 운영 + Pricing/Terms/Privacy/Refund 페이지 및 법인 정보 반영 완료.

### 🚀 다음 마일스톤
1. **`wolin-id/cloud` 실전 가동**: Passwordless Auth(`auth.wolin.id`) + Per-Pod Oxigraph 격리(`t_962a0f95`) + 무손실 이주(`t_7a1e0e91`) + Airwallex 결제(`t_4703f0e1`).
2. **실서버 배포 & 생태계 실증**: AWS 서울 EC2 + R2 프로덕션 배포(`t_a71cd45a`) → 서로 다른 2개 웹/3D 앱 간 단일 WebID·아바타 공유 파일럿(`t_1f36a30c`).
3. **오픈소스 정식 Public 런칭 (`t_9e8b355f`)**: `wolin-id/{wolin,web,docs,sdk-*}` 일괄 Public 전환 및 `v0.1.0` 릴리즈.
