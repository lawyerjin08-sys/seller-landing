# CLAUDE.md

이 파일은 Claude Code가 이 저장소에서 작업할 때 참고하는 프로젝트 가이드입니다.

## 서비스 개요

**온라인 셀러 계정 정지·정산금 법률 상담 랜딩 페이지** (변호사 유환진).

쿠팡·스마트스토어·11번가 등 온라인 판매 플랫폼에서 **계정이 정지되거나 정산금이 보류된 셀러**를 대상으로, 변호사의 법적 대응 서비스를 소개하고 상담을 접수받는 한국어(`lang="ko"`) 단일 페이지 사이트입니다.

- 배포 도메인: https://www.myseller.kr
- 핵심 전환 흐름: 진단 퀴즈(플랫폼/사유/금액/시도 여부) → 결과 안내 → 이름·연락처·희망 시간 입력 → EmailJS로 상담 신청 전송

## 프로젝트 구조

**빌드 도구·프레임워크·패키지 매니저 없음.** 순수 정적 사이트입니다.

```
seller-landing/
├── index.html      # 전체 사이트 (마크업 + CSS + JS 모두 인라인, ~1,530줄)
├── vercel.json     # Vercel 배포 설정 (cleanUrls + 보안 헤더/CSP)
├── favicon.svg
├── profile.jpg     # 변호사 프로필 (fallback)
├── profile.webp    # 변호사 프로필 (우선)
└── CLAUDE.md
```

- `package.json`, `node_modules`, 빌드 산출물 **없음** → `npm install`이나 빌드 단계가 필요 없습니다.
- 모든 CSS는 `index.html`의 `<style>`, 모든 JS는 하단 `<script>`에 인라인으로 들어 있습니다.

## 배포

**Vercel** — 별도 빌드 없이 저장소 파일을 그대로 서빙합니다.

- `main` 브랜치에 push하면 Vercel이 자동 배포합니다. (파일만 올리면 됨)
- `vercel.json`:
  - `cleanUrls: true`
  - 보안 헤더: `X-Frame-Options: DENY`, `X-Content-Type-Options`, HSTS, `Referrer-Policy`, `Permissions-Policy` 등
  - **CSP(Content-Security-Policy)** 설정 있음 → 아래 "주의사항" 참고

## 외부 의존성 (모두 CDN / 외부 서비스, 로컬 설치 없음)

| 용도 | 서비스 | 식별자 / 로드 방식 |
|------|--------|-------------------|
| 웹폰트 | **Pretendard** | jsDelivr CDN (`orioncactus/pretendard@v1.3.9`) |
| 상담 신청 전송 | **EmailJS** | `@emailjs/browser@4`, public key `I2i588ZHaaUSH_jQA`, service `service_myseller`, template `template_gog2y3c` |
| 분석 | **Google Analytics (gtag)** | `G-301RW9CTW7` |
| 세션 분석 | **Microsoft Clarity** | `w5mapb4l56` |

## 수정 시 주의사항

- **단일 파일 구조**: 모든 코드가 `index.html` 하나에 있습니다. 편집 위치를 찾을 때는 섹션 주석을 기준으로 하세요.
  - CSS 섹션: `/* ===== 1. HERO ===== */`, `/* ===== 5. REVIEWS ===== */` 형태
  - HTML 섹션: `<!-- ===== 1. HERO ===== -->`, `<!-- ===== 4. INLINE QUIZ SECTION ===== -->` 형태
  - **CSS 섹션 번호와 HTML 섹션 번호가 서로 다를 수 있으니** 주석 텍스트로 매칭하세요.
- 페이지 순서(HTML): HERO → 변호사 소개 → 리뷰 → 진단 퀴즈(인라인) → 왜 변호사인가 → 비용 안내 → 월정액 자문 → FAQ → 마감 CTA. 진단 **퀴즈 모달**과 플로팅 **카카오** 버튼도 포함.
- **CSP 주의**: 새로운 외부 스크립트/스타일/폰트/이미지 도메인을 추가하려면 `vercel.json`의 CSP `Content-Security-Policy` 값도 함께 업데이트해야 합니다. 그렇지 않으면 브라우저에서 차단됩니다.
  - 현재 허용 도메인: `cdn.jsdelivr.net`(script/style/font), `cdn.emailjs.com`, `www.clarity.ms`, `api.emailjs.com`(connect) 등. 인라인 스타일/스크립트는 `'unsafe-inline'`으로 허용됨.
- **EmailJS 폼**: 상담 신청은 `emailjs.send("service_myseller", "template_gog2y3c", {...})`로 전송됩니다. 폼 필드(`platform`, `reason`, `amount`, `experience`, `name`, `phone`, `preferred_time`, `result`, `timestamp`)를 바꾸면 EmailJS 대시보드의 템플릿 변수와 반드시 맞춰야 합니다.
- **키 노출**: EmailJS public key, GA/Clarity ID는 클라이언트 코드에 노출되는 값입니다(정상). 다만 EmailJS 대시보드에서 도메인 제한 등 보안 설정을 유지하세요.
- **프로필 이미지**: `.webp` 우선 + `.jpg` fallback 구조이므로 교체 시 두 파일을 함께 관리하세요.
- **로컬 미리보기**: 빌드가 없으므로 브라우저로 `index.html`을 직접 열거나 간단한 정적 서버(`npx serve`, `python -m http.server`)로 확인하면 됩니다.

## 코드 실제 값 (index.html JS 확인 결과)

수정 전에 아래 현재 동작을 반드시 인지하세요. (모두 `index.html` 인라인 JS에서 확인)

- **EmailJS**: `emailjs.send("service_myseller", "template_gog2y3c", {...})`
  - 템플릿 ID: **`template_gog2y3c`**
  - **수신 이메일 주소는 코드에 없습니다.** EmailJS 대시보드의 템플릿(`template_gog2y3c`) "To" 필드에 설정되어 있습니다. 수신 주소 변경은 코드가 아니라 EmailJS 대시보드에서 합니다.
  - 전송 필드: `platform`, `reason`, `amount`, `experience`, `name`, `phone`, `preferred_time`, `result`, `timestamp`
- **히어로 카운트업 숫자** (스크롤 진입 시 IntersectionObserver로 1.5초 애니메이션):
  - `count-sellers` → **600** (`명+`) — "셀러가 먼저 찾았습니다"
  - `count-amount` → **11** (`억+`) — "반환받은 정산금"
  - `count-free` → **무료** (애니메이션 없이 표시) — "첫 상황확인"
  - 숫자를 바꾸려면 JS의 `countUp(... , 600, 1500)` / `countUp(... , 11, 1500)` 인자를 수정.
- **상담폼 제출 결과 화면** (폼 영역 `innerHTML`을 교체, 성공/실패 분리):
  - **성공 `showComplete()`**: 초록 체크 + **"신청이 접수되었습니다"** + "카톡으로 <mark>신청 완료</mark> 남겨주시면 확인 후 변호사가 바로 전화드립니다"('신청 완료'에 연노랑 형광펜) + 노란(#FEE500) **"카카오톡으로 '신청 완료' 남기기"** 버튼 + 작게 "영업시간 내 2시간 이내 연락드립니다"
  - **실패 `showError()`** (`.catch`에서 호출): 빨간 ! 아이콘 + **"일시적인 오류로 접수가 안 됐습니다"** + "아래 카톡으로 <mark>신청</mark>이라고 남겨주시면 변호사가 바로 확인합니다" + 노란(#FEE500) **"카카오톡으로 '신청' 남기기"** 버튼 + **전화 문의 02-6214-1114**(`tel:0262141114`)
  - 설계 의도: 전송이 실패해도 고객이 **카톡·전화로 계속 이어질 수 있게** 하는 것. (실패를 그냥 성공처럼 감추지 않음)
- **퀴즈 자동 진행**: **예, 답변(옵션) 클릭 시 자동으로 다음 문항으로 넘어갑니다.** (`.quiz-option` 클릭 → 선택 표시 후 `setTimeout` **300ms** 뒤 다음 스텝으로 이동, 마지막 문항이면 결과로 이동.) "다음" 버튼을 누를 필요가 없으므로, 문항을 추가/수정할 때 이 자동 진행 로직(`data-step`, `questionSteps`)과의 정합성을 확인하세요. 인라인 퀴즈와 모달 퀴즈 두 인스턴스가 같은 엔진(`createQuizEngine`)을 공유합니다.

## 카카오 오픈채팅

- 문의 링크는 **카카오 채널이 아니라 오픈채팅**입니다: `https://open.kakao.com/o/sq0Svxoi`
- 페이지 내 여러 곳(플로팅 버튼, 히어로, 인라인 퀴즈, 완료 화면)에서 이 URL을 사용합니다. 변경 시 `index.html` 전체에서 일괄 교체하세요.

## 카피(문구) 규칙

- **"전문" 표현 금지 → "전담"** 으로 씁니다. (변호사 광고 규정상 "전문" 문구는 지양)
- **"상담 무료" 표기는 유지**합니다. (히어로의 "무료 / 첫 상황확인" 등) 이 무료 표기를 **변경하려면 반드시 대표 확인**을 받으세요. 임의 변경 금지.

## 제약 — 다른 프로젝트와 절대 섞지 말 것

- 별도 프로젝트 **"따박따박"** (`C:\Users\user\chaekwon-site`, **React** 기반)이 있습니다.
- **따박따박 코드를 이 저장소에 복사·붙여넣기 하지 마세요.** 이 프로젝트는 빌드 없는 순수 정적 HTML이고, 따박따박은 React라 구조가 완전히 다릅니다.
- **두 폴더를 절대 섞지 마세요.** 파일 이동/참조/컴포넌트 이식 금지. 각각 독립적으로 작업합니다.

## index.html 섹션별 라인 위치 (목차)

편집 시작 지점을 빠르게 찾기 위한 대략적 라인 번호입니다. (파일 수정으로 달라질 수 있으니 `grep`으로 섹션 주석 재확인 권장)

**`<head>` / 설정 (~1–623)**
- `<head>` 시작, 메타/OG/canonical: ~3–17
- 인라인 `<style>` 시작: ~18
- 외부 스크립트(GA `G-301RW9CTW7`, EmailJS init, Clarity `w5mapb4l56`): ~607–622

**CSS 섹션 (`<style>` 내, `/* ===== ... ===== */`)**
- 1. HERO: ~96 · QUIZ(공용): ~171 · 3. WHY LAWYER: ~299 · 4. 변호사 소개: ~338
- 5. REVIEWS: ~387 · 6. 비용 안내: ~444 · 7. 월정액: ~498 · 8. FAQ: ~519
- 마감 CTA(다크): ~542 · FLOATING KAKAO: ~562 · RESPONSIVE: ~575

**HTML 본문 섹션 (`<!-- ===== ... ===== -->`)**
- 플로팅 카카오 버튼: ~627
- 퀴즈 모달: ~640
- 1. HERO(카운트업 프루프 포함): ~757 (`hero-proof` ~785)
- 2. 변호사 소개 (ATTORNEY): ~804
- 3. REVIEWS(후기): ~856
- 4. INLINE QUIZ SECTION(진단 퀴즈): ~933
- 5. WHY LAWYER: ~1055
- 6. 비용 안내(가격): ~1090
- 7. 월정액 자문: ~1145
- 8. FAQ: ~1181
- 마감 CTA: ~1222

**인라인 JS (`<!-- ===== SCRIPTS ===== -->` ~1241 이하)**
- 퀴즈 엔진 `createQuizEngine` / 자동 진행 로직: ~1327 부근
- EmailJS 전송 `emailjs.send(...)`: ~1406
- 제출 완료 화면 `showComplete()`: ~1424
- 히어로 카운트업 애니메이션: ~1448–1495

> CSS 섹션 번호와 HTML 섹션 번호가 서로 다릅니다(예: CSS "3. WHY LAWYER" ≠ HTML "3. REVIEWS"). 반드시 **주석 텍스트**로 매칭하세요.
