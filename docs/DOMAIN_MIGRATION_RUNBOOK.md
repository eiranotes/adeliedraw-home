# Adelie Draw Domain Migration Runbook

상태: 실행 절차 / 2026-09-15

정본 구조는 `DOMAIN_ARCHITECTURE_CANONICAL.md`를 따른다. 이 문서는 실제 전환 작업의 순서와 검증 절차만 관리한다.

## 1. 현재 확인된 상태

| 대상 | 현재 상태 | 다음 동작 |
|---|---|---|
| `adeliedraw.com` | Cafe24 공식몰 응답 | 마지막 단계에서 Brand Home으로 전환 |
| `www.adeliedraw.com` | Cafe24 edge | 마지막 단계에서 apex로 수렴 |
| `app.adeliedraw.com` | GitHub Pages 정상 | 유지 |
| `app.adeliedraw.com/pages/` | GitHub Pages 정상 | 유지 |
| `portfolio.adeliedraw.com` | `adeliedraw.com`을 CNAME으로 따라감, HTTPS 실패 | GitHub Pages custom domain + DNS를 묶어서 전환 |
| `shop.adeliedraw.com` | `adeliedraw.com`을 CNAME으로 따라감, HTTPS 실패 | Cafe24 연결 도메인 등록 및 SSL 발급이 선행 |

2026-09-15 직접 확인 시 `portfolio`와 `shop`은 TLS handshake 단계에서 실패했다. 현재 공개 링크로 사용하면 안 된다.

## 2. 준비 완료 항목

- Brand Home 소스: `eiranotes/adeliedraw-home`
- Brand Home 로컬: `/Volumes/DevDrive/Projects/adeliedraw-home`
- Portfolio 전환 브랜치: `eiranotes/portfolio`의 `domain/portfolio-subdomain`
- Portfolio 전환 브랜치는 canonical, hreflang, Open Graph, Twitter image URL, sitemap, Organization JSON-LD를 `https://portfolio.adeliedraw.com/` 기준으로 변경한 상태다.
- Portfolio 전환 브랜치 검증: `python3 check_site.py`, `node --check public/app.js` 통과.
- Adelie Pages 모바일 모조지 보강은 별도 커밋으로 이미 라이브 반영했다.

## 3. 전환 순서

### Phase A — Shop 먼저 독립

Cafe24 관리자에서 수행한다.

1. `쇼핑몰 설정 > 기본 설정 > 쇼핑몰 정보 > 도메인 설정 > 도메인 관리`로 이동한다.
2. `shop.adeliedraw.com`을 현재 한국어 공식몰의 연결 도메인으로 등록한다.
3. Cafe24가 해당 호스트의 SSL 인증서를 발급하도록 한다.
4. 현재 `shop`이 apex를 따라가는 임시 CNAME에 의존하지 않도록 Cafe24가 요구하는 연결 상태를 확정한다.
5. 다음을 실제 브라우저와 HTTP 요청으로 검증한다.
   - `/` 홈
   - 상품 상세 1개
   - 카테고리 1개
   - 장바구니 진입
   - 로그인 진입
   - 주문서 진입 전 단계
6. 실제 결제나 회원정보 변경은 이 검증에 포함하지 않는다.

완료 조건: `https://shop.adeliedraw.com/`이 유효한 인증서로 열리고 상품/장바구니 경로가 같은 호스트에서 유지된다.

### Phase B — Portfolio 독립

Shop이 독립된 뒤 진행한다.

1. GitHub Pages `eiranotes/portfolio`의 custom domain을 `portfolio.adeliedraw.com`으로 설정한다.
2. Cafe24 DNS에서 `portfolio`를 `eiranotes.github.io`로 직접 CNAME 연결한다. repository 이름을 CNAME target에 넣지 않는다.
3. GitHub Pages DNS health 및 HTTPS 인증서가 정상 상태가 될 때까지 확인한다.
4. `domain/portfolio-subdomain` 브랜치를 main에 반영한다.
5. KR `/`, JA `/ja/`, EN `/en/` 세 주소를 검증한다.
6. canonical, hreflang, OG URL, sitemap을 실제 응답에서 다시 확인한다.

중요: GitHub Actions Pages에서는 repository의 `CNAME` 파일이 custom domain 설정을 대신하지 않는다. Pages 설정/API에서 custom domain을 별도로 지정해야 한다.

### Phase C — 기존 Portfolio 주소 처리

Portfolio custom domain 적용 직후 아래 구주소의 실제 HTTP 상태와 `Location`을 먼저 확인한다.

- `https://app.adeliedraw.com/portfolio/`
- `https://app.adeliedraw.com/portfolio/ja/`
- `https://app.adeliedraw.com/portfolio/en/`

GitHub가 새 custom domain으로 영구 이동을 제공하면 그 동작을 그대로 사용한다. 자동 이동이 없거나 목적 경로가 언어별로 보존되지 않으면 `eiranotes/eiranotes.github.io`에 별도 전환 경로를 구현한다. 실제 확인 없이 자동 이동을 가정하지 않는다.

### Phase D — 기존 apex Shop URL 인벤토리

루트를 Brand Home으로 바꾸기 전에 현재 `adeliedraw.com`의 외부 유입 URL을 수집한다.

2026-09-15 현재 Cafe24 sitemap 기준 119개 URL을 `LEGACY_SHOP_URL_INVENTORY.csv`에 저장했다.

- 상품: 51
- 카테고리: 62
- 게시판: 2
- 회원/약관: 2
- 쇼핑몰 정보: 1
- 홈: 1

홈 `/`은 Brand Home으로 바뀌므로 리다이렉트 대상에서 제외한다. 나머지 sitemap URL은 우선 동일 path를 `shop.adeliedraw.com`에 보존하는 정책으로 기록했다.

현재 sitemap의 모든 URL host는 `adeliedraw.cafe24.com`이며, `adeliedraw.com`으로 접속한 상품 상세의 canonical도 `adeliedraw.cafe24.com`으로 확인됐다. `shop.adeliedraw.com`을 대표/연결 도메인으로 확정할 때 sitemap과 canonical host가 새 shop 주소로 바뀌는지 반드시 검증한다.

최소 범위:

- 상품 상세 URL
- 카테고리 URL
- 게시판/공지 URL
- 검색 결과에 노출된 기타 Cafe24 URL

목표 규칙은 대부분 `https://adeliedraw.com/<path>`에서 `https://shop.adeliedraw.com/<path>`로 path/query를 보존해 영구 이동하는 것이다. 개별 예외가 있는지만 인벤토리에서 확인한다.

### Phase E — Brand Home 배포

Phase A–D가 끝난 뒤에만 진행한다.

1. `eiranotes/adeliedraw-home`을 선택한 production host에 배포한다.
2. production host는 `/`, `/ja/`, `/en/` 같은 Brand Home 경로를 제공해야 한다.
3. 기존 Cafe24 상품/카테고리 경로에는 `shop.adeliedraw.com`으로의 HTTP 301/308 path-preserving redirect를 제공할 수 있어야 한다.
4. 이 요구조건 때문에 GitHub Pages 단독 사용 여부는 기존 URL 인벤토리 결과를 보고 최종 결정한다. 정적 JavaScript redirect를 SEO용 301/308과 동일하게 취급하지 않는다.
5. production host가 준비된 뒤 apex DNS를 마지막에 전환한다.
6. `www`는 apex의 동일 path/query로 301/308 수렴시킨다.

## 4. 전환 후 검증

다음 항목이 모두 통과해야 완료로 처리한다.

- `adeliedraw.com/` → Brand Home 200
- `www.adeliedraw.com/...` → apex 동일 경로로 영구 이동
- `portfolio.adeliedraw.com/`, `/ja/`, `/en/` → 200 + 유효 HTTPS
- `shop.adeliedraw.com/` → 200 + 유효 HTTPS
- `app.adeliedraw.com/` → 200
- `app.adeliedraw.com/pages/` 및 기존 packs/privacy/terms/support/data/info → 정상
- 기존 Portfolio 구주소 → 새 Portfolio 대응 경로
- 기존 apex 상품 URL → 동일 path의 `shop` URL 또는 명시 매핑
- 각 사이트 canonical/OG/sitemap이 자기 canonical host와 일치
- 모바일 390px에서 가로 넘침/텍스트 잘림 없음

## 5. 롤백 기준

아래 중 하나라도 발생하면 루트 전환을 중지하거나 직전 DNS로 되돌린다.

- Shop HTTPS 미발급 또는 상품/장바구니 경로 오류
- Portfolio custom domain이 인증서 발급에 실패
- 기존 상품 URL이 대량 404
- apex와 www가 서로 순환 redirect
- 결제/로그인 경로의 host 또는 cookie 동작 이상

DNS 롤백이 필요한 경우 변경 전 DNS 스냅샷을 기준으로 복구하고, 실제 응답이 돌아온 뒤 다음 원인을 수정한다.
