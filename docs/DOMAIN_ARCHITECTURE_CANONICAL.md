# Adelie Draw Domain Architecture — Canonical

문서 상태: **정본 / Source of Truth**  
최초 확정: 2026-09-15  
적용 범위: Adelie Draw의 공개 웹, 포트폴리오, 커머스, 앱, 향후 공개 서비스의 도메인·URL 구조  

이 문서는 Adelie Draw 웹 주소 체계의 단일 정본이다. 실제 DNS 값, 호스팅 사업자, 배포 방식은 바뀔 수 있지만 **각 주소가 맡는 역할과 확장 규칙은 이 문서를 기준으로 변경한다.**

---

## 1. 핵심 원칙

1. `adeliedraw.com`은 제품을 직접 판매하거나 한 앱을 설명하는 곳이 아니라 **브랜드 전체의 공식 입구**다.
2. 서로 목적이 다른 큰 서비스만 서브도메인으로 분리한다. 현재 1차 경계는 `portfolio`, `shop`, `app`이다.
3. 한 서비스 안의 개별 콘텐츠와 제품은 새 서브도메인을 만들지 않고 **경로(path)** 로 확장한다.
4. 언어만 다르다는 이유로 새 서브도메인을 만들지 않는다. 기본은 `/ja/`, `/en/` 같은 언어 경로다.
5. 같은 콘텐츠를 여러 호스트에서 동시 제공하지 않는다. 각 콘텐츠에는 한 개의 canonical host가 있어야 한다.
6. 운영·재고·관리 도구는 공개 브랜드 트리에 섞지 않는다. 외부 접근이 필요해져도 인증·접근제어를 전제로 별도 운영 영역으로 둔다.
7. DNS를 먼저 바꾸지 않는다. 목적지 서비스와 HTTPS, 리다이렉트, canonical을 준비한 뒤 마지막에 진입점을 전환한다.

---

## 2. 확정 도메인 트리

```text
adeliedraw.com
├─ /                         Brand Home / Info
├─ /ja/                      Brand Home — Japanese [reserved]
└─ /en/                      Brand Home — English  [reserved]

www.adeliedraw.com
└─ → adeliedraw.com          Alias only; path/query preserving redirect

portfolio.adeliedraw.com
├─ /                         Portfolio — Korean
├─ /ja/                      Portfolio — Japanese
└─ /en/                      Portfolio — English

shop.adeliedraw.com
└─ /...                      Official commerce; Cafe24 product/category/member/order paths

app.adeliedraw.com
├─ /                         Adelie Draw app directory
├─ /pages/                   Adelie Pages product site
│  ├─ /packs/...             Sticker pack detail
│  ├─ /privacy/              Privacy
│  ├─ /terms/                Terms
│  ├─ /support/              Support
│  ├─ /data/                 Data management
│  └─ /info/                 App information
└─ /<future-app>/             Future app product site
```

### 공개 1차 내비게이션

브랜드 레벨에서 사용자가 이동할 수 있는 최상위 목적지는 아래 네 개만 둔다.

| 표시 | canonical URL | 역할 |
|---|---|---|
| Adelie Draw | `https://adeliedraw.com/` | 브랜드 홈·공식 안내 |
| Portfolio | `https://portfolio.adeliedraw.com/` | 작품·컬렉션·행사·B2B 검토 자료 |
| Shop | `https://shop.adeliedraw.com/` | 공식 온라인 구매 |
| Apps | `https://app.adeliedraw.com/` | 앱 목록과 디지털 문구 |

`store`, `works`, `apps`, `about`처럼 동일 역할을 반복하는 별도 서브도메인은 만들지 않는다.

---

## 3. 서비스별 소유권과 확장 규칙

### 3.1 Brand Home — `adeliedraw.com`

목적은 사용자가 **Portfolio / Shop / Apps 중 어디로 갈지 빠르게 선택**하게 하는 것이다.

- 현재 `app.adeliedraw.com`의 선택형 목록 구조를 상위 브랜드 허브로 확장한다.
- 시각 언어는 Adelie Pages의 모조지 배경, LINE Seed Sans KR, 인디고 `#123F8D`, 웜그레이 계열을 브랜드 공통 웹 톤으로 사용한다.
- 상세 작품 목록, 상품 카탈로그, 앱 기능 설명을 중복해서 싣지 않는다.
- 브랜드 소개는 짧게 유지하고 각 전문 서비스로 넘긴다.
- 향후 다국어는 `/ja/`, `/en/` 경로로 확장한다.

### 3.2 Portfolio — `portfolio.adeliedraw.com`

- 현재 포트폴리오 저장소와 콘텐츠를 독립 서비스로 유지한다.
- `KR / JA / EN`은 각각 `/`, `/ja/`, `/en/`으로 유지한다.
- 행사 주최사, 바이어, 입점처 등 검토자가 대상이다.
- 작품·컬렉션·제품·약력·행사 이력은 이 호스트가 canonical이다.
- 향후 PDF press kit, wholesale guide가 생겨도 원칙적으로 `/press/`, `/wholesale/` 같은 경로를 우선한다.

### 3.3 Shop — `shop.adeliedraw.com`

- 기존 Cafe24 쇼핑몰이 canonical commerce host다.
- 상품 상세, 옵션, 장바구니, 회원, 주문, 결제 경로는 Cafe24 구조를 보존한다.
- 브랜드 홈 전환 때문에 기존 상품 URL을 전부 홈으로 보내지 않는다. 대응되는 `shop` URL로 이동시킨다.
- 해외몰은 실제 Cafe24 멀티몰 구조와 결제·재고·SEO 분리가 확인되기 전에는 새 호스트를 만들지 않는다.
- 국가별 몰이 기술적으로 완전히 분리되어 별도 호스트가 필요할 때만 `jp.shop.adeliedraw.com` 같은 계층을 후보로 검토한다. 사전 생성하지 않는다.

### 3.4 Apps — `app.adeliedraw.com`

- 루트 `/`는 앱 디렉터리다.
- 개별 앱은 `/pages/`, `/<future-app>/`처럼 경로로 확장한다.
- 앱 하나가 추가될 때마다 `pages.adeliedraw.com` 같은 새 호스트를 만들지 않는다.
- 앱별 개인정보처리방침·지원·데이터 관리는 각 앱 경로 아래에 둔다.
- 인증, 별도 백엔드, 쿠키 격리처럼 명확한 기술적 경계가 생기는 경우에만 별도 호스트를 검토한다.

---

## 4. 향후 확장용 예약 규칙

다음 이름은 **필요가 생길 때만** 사용할 수 있는 예약 후보이며 현재 DNS에 미리 만들지 않는다.

| 후보 | 생성 조건 | 기본 대안 |
|---|---|---|
| `assets.adeliedraw.com` | 여러 서비스가 동일 정적 자산을 공유하고 CDN 운영이 필요할 때 | 각 서비스 내부 `/assets/` |
| `api.adeliedraw.com` | 외부 앱이 실제 공용 API를 사용하게 될 때 | 앱별 내부 API |
| `go.adeliedraw.com` | QR·캠페인용 관리형 단축 URL과 추적이 필요할 때 | canonical URL 직접 사용 |
| `status.adeliedraw.com` | 외부 사용자에게 공개할 상태 페이지가 생길 때 | 각 서비스 공지 |
| `ops.adeliedraw.com` | 운영 시스템의 원격 웹 접근이 실제 필요하고 SSO/Zero Trust가 준비됐을 때 | 로컬/사설망 운영 |

예약 후보를 실제 생성할 때도 5절의 서브도메인 생성 판정을 통과해야 한다.

---

## 5. 새 서브도메인 생성 판정

아래 질문 중 **둘 이상이 명확하게 Yes**이고, 경로로 해결하는 것보다 운영상 이점이 클 때만 새 서브도메인을 만든다.

1. 사용자의 목적과 서비스 성격이 기존 호스트와 독립적인가?
2. 별도 호스팅·배포·보안 정책이 필요한가?
3. 별도 검색 색인/canonical 관리가 필요한가?
4. 별도 인증·쿠키·CSP/CORS 경계가 필요한가?
5. 장애·릴리스 주기를 독립시키는 가치가 있는가?

다음 사유만으로는 새 서브도메인을 만들지 않는다.

- 메뉴가 하나 늘었다.
- 언어가 추가됐다.
- 이벤트 페이지가 하나 필요하다.
- 앱 안의 기능이 하나 추가됐다.
- 캠페인 랜딩을 잠시 운영한다.

이 경우 각각 기존 서비스의 경로를 우선한다.

---

## 6. URL·언어 규칙

### canonical

- 루트 브랜드: `https://adeliedraw.com/...`
- Portfolio: `https://portfolio.adeliedraw.com/...`
- Shop: `https://shop.adeliedraw.com/...`
- Apps: `https://app.adeliedraw.com/...`

`www`는 별도 콘텐츠 호스트가 아니다.

### 언어

- 한국어 기본 경로: `/`
- 일본어: `/ja/`
- 영어: `/en/`
- 언어 전환은 가능한 경우 동일 콘텐츠의 대응 경로로 이동한다.
- 서비스가 해당 언어를 실제 제공하지 않으면 존재하지 않는 번역 URL을 추정해 만들지 않는다.

### URL naming

- 영문 소문자와 하이픈을 기본으로 한다.
- 이미 공개된 안정 경로는 미관상의 이유만으로 변경하지 않는다.
- 개별 앱의 slug는 출시 후 바꾸지 않는 영구 식별자로 취급한다.
- 쿼리스트링은 필터·추적 등 일시 상태에 사용하고 영구 콘텐츠 식별에는 사용하지 않는다.

---

## 7. 리다이렉트 정본

| From | To | 정책 |
|---|---|---|
| `www.adeliedraw.com/<path>` | `adeliedraw.com/<path>` | 301 또는 308, path/query 보존 |
| 기존 `app.adeliedraw.com/portfolio/` | `portfolio.adeliedraw.com/` | 영구 이동 |
| 기존 `app.adeliedraw.com/portfolio/ja/` | `portfolio.adeliedraw.com/ja/` | 영구 이동 |
| 기존 `app.adeliedraw.com/portfolio/en/` | `portfolio.adeliedraw.com/en/` | 영구 이동 |
| 기존 루트 쇼핑 상품/카테고리 URL | 대응 `shop.adeliedraw.com` URL | URL별 매핑, 홈 일괄 이동 금지 |

DNS는 경로 리다이렉트를 할 수 없다. 301/308이 필요한 이동은 HTTP 라우팅 계층에서 구현한다.

---

## 8. DNS·TLS 운영 규칙

현재 권한 네임서버는 Cafe24 계열이다. 네임서버 이전은 이 구조의 필수 조건이 아니다.

- 서비스 호스트는 가능한 한 명시 레코드로 관리한다. 설명되지 않은 wildcard DNS에 기대지 않는다.
- 루트 변경 전 `shop`, `portfolio`, `app`을 각각 독립 목적지로 먼저 연결한다.
- 새 호스트는 HTTPS 인증서 발급과 자동 갱신이 확인된 뒤 공개 링크에 추가한다.
- DNS 변경 전 MX/TXT/SPF/DKIM/DMARC 등 메일 관련 레코드를 스냅샷으로 보존한다.
- 마이그레이션 직전에는 필요 시 TTL을 낮추고 안정화 후 정상 수준으로 되돌린다. 고정 TTL 값은 호스팅/DNS 사업자의 실제 운영 제약을 확인해 정한다.
- `CNAME` 파일 존재만으로 GitHub Pages 사용자 지정 도메인 설정 완료라고 판단하지 않는다. Pages 설정과 실제 HTTPS 응답을 함께 확인한다.

---

## 9. SEO·검색 정본

각 서비스는 자신의 canonical host를 메타데이터 전체에서 일관되게 사용한다.

- `<link rel="canonical">`
- `hreflang`
- Open Graph URL과 이미지 URL
- sitemap
- 구조화 데이터 URL
- 내부 링크

도메인 이전 시에는 구주소를 유지한 채 새 주소로 영구 이동시키고, 사이트맵과 Search Console 속성을 새 구조에 맞게 갱신한다. 실제 HTTP 상태와 `Location`을 확인하기 전에는 이전 완료로 표시하지 않는다.

---

## 10. 보안·애널리틱스 경계

- 쿠키의 `Domain=.adeliedraw.com` 사용은 기본값이 아니다. 서비스 간 공유가 정말 필요한 쿠키만 명시적으로 검토한다.
- Shop 로그인/주문 쿠키를 Portfolio나 Apps에서 공유하지 않는다.
- CORS는 필요한 origin만 허용한다. 단순히 같은 브랜드 도메인이라는 이유로 모든 서브도메인을 wildcard 허용하지 않는다.
- 운영 도구는 공개 내비게이션과 검색 색인에서 제외한다.
- 분석 도구를 통합하더라도 서비스별 hostname을 구분해 성과를 볼 수 있게 한다.
- 결제·회원·주문 흐름에는 브랜드 공통 메뉴를 억지로 삽입해 이탈이나 보안 경계를 흔들지 않는다.

---

## 11. 배포 단위 정본

| 서비스 | 소스/배포 단위 | 변경 원칙 |
|---|---|---|
| Brand Home | `adeliedraw-home` | 가장 작은 정적 허브. 다른 서비스 콘텐츠를 복제하지 않음 |
| Portfolio | 기존 `eiranotes/portfolio` | 콘텐츠와 3개 언어 유지, custom domain만 독립 |
| Shop | 기존 Cafe24 쇼핑몰 | 상품·주문·회원 시스템 유지 |
| Apps directory | 기존 `eiranotes/eiranotes.github.io` | `app.adeliedraw.com` custom domain 유지 |
| Adelie Pages | 기존 `eiranotes/pages` | `/pages/` 경로 및 하위 정책 문서 유지 |

GitHub 사용자 사이트의 `app.adeliedraw.com` 설정을 루트 도메인으로 바꾸지 않는다. 프로젝트 Pages의 상속 경로에 영향을 줄 수 있기 때문이다.

---

## 12. 변경 절차

도메인 구조 변경은 아래 순서를 지킨다.

1. 이 정본 문서에서 역할과 URL을 먼저 수정한다.
2. 현재 DNS/SSL/canonical/redirect 상태를 스냅샷한다.
3. 새 목적지 서비스와 배포를 먼저 준비한다.
4. 새 호스트의 HTTPS와 콘텐츠를 검증한다.
5. canonical/hreflang/sitemap/내부 링크를 갱신한다.
6. 구주소 리다이렉트를 준비한다.
7. 마지막에 DNS 또는 대표 도메인을 전환한다.
8. 데스크톱·모바일·검색봇 관점으로 실제 HTTP 응답과 핵심 사용자 흐름을 재검증한다.

---

## 13. 현재 전환 체크리스트

- [x] `adeliedraw-home` 작업 디렉터리 생성
- [x] 브랜드 홈 정보 구조와 시각 방향 확정
- [x] `Portfolio / Shop / Apps` 목적지 확정
- [x] Brand Home의 폰트·모조지 자산을 자체 `assets/`로 복제
- [x] Brand Home 데스크톱/모바일 렌더 검수
- [ ] `shop.adeliedraw.com`을 Cafe24 공식몰에 독립 연결
- [ ] `portfolio.adeliedraw.com`을 기존 Portfolio 배포에 독립 연결
- [ ] Portfolio의 canonical/hreflang/OG/sitemap 변경
- [ ] 기존 `app.adeliedraw.com/portfolio/...` 영구 이동 구현
- [ ] 기존 루트 상품·카테고리 URL 목록 작성 및 `shop` 매핑
- [ ] `adeliedraw.com` Brand Home 배포 및 HTTPS 검증
- [ ] `www` → apex 영구 이동 검증
- [ ] 각 서비스의 브랜드 공통 링크 반영
- [ ] Search Console/사이트맵 전환 검증

---

## 14. 변경 기록

### 2026-09-15

- `adeliedraw.com`을 Brand Home으로 정의.
- `portfolio`, `shop`, `app` 3개를 1차 공개 서브도메인으로 확정.
- 앱 확장은 `app` 아래 경로 우선 원칙으로 확정.
- 언어 확장은 경로 우선 원칙으로 확정.
- 운영·CDN·API·단축 URL은 조건부 예약 이름으로만 정의하고 선점 생성하지 않기로 함.
