# 강남형 통합돌봄 G-care 웹앱 — 인수인계 문서 (2026-09-15 기준)

다른 개발 도구(Genspark, Lovable, 일반 개발자)에서 이 작업을 이어받을 때 필요한 모든 정보를 정리한 문서입니다.
이 문서와 함께 `webapp` 폴더 전체(단, `node_modules`, `.git` 제외)를 전달하면 됩니다.

---

## 1. 가장 먼저 알아야 할 것: 이 폴더에는 "원본 소스"가 없습니다

| 구분 | 내용 |
|---|---|
| 원본 프로젝트 | Lovable 로 생성된 TanStack Start(React + Vite + Tailwind) 프로젝트. 빌드 시 경로는 `/home/user/build-site/src/…` (`src/routes/__root.tsx`, `index.tsx`, `services.tsx`) |
| 이 폴더의 정체 | 그 프로젝트를 **빌드한 결과물**(Nitro, Cloudflare Workers용). `server/` = 서버 렌더링 코드, `public/assets/` = 브라우저 번들 |
| GitHub | `https://github.com/gnwf2014-afk/-260803` 브랜치 `genspark_ai_developer` (Genspark 가 빌드 결과물을 올리던 곳) |
| 2026-09-15 작업 방식 | 원본 소스가 없어 **빌드 결과물을 스크립트로 직접 패치**했습니다. 원본에서 다시 빌드하면 이 패치들은 사라지므로, 아래 3장의 변경 목록을 원본 소스에 옮겨 넣어야 합니다 |

> 결론: **원본 Lovable 프로젝트(또는 그 GitHub 저장소)를 먼저 확보**하세요. 확보되면 3장의 변경을 소스에 구현하고, 4장의 데이터·스크립트는 그대로 옮겨 쓰면 됩니다.

---

## 2. 폴더 구성과 "그대로 가져가야 할 것"

```
webapp/
├─ run.mjs                     로컬 실행 서버 (Node). 정적 파일 우선 + 서버 렌더링 + 뉴스 자동 수집(24h)
├─ 실행.command / 실행.bat      더블클릭 실행
├─ package.json                npm start / npm run data / data:check
├─ tools/
│  ├─ build-data.mjs           ★ 엑셀 → 기관 데이터 변환 (정제·요약 추출·파일명 해시 갱신)
│  ├─ fetch-news.mjs           ★ 보건복지부 보도자료 RSS 수집 → public/news.json
│  ├─ patch-ui.mjs             빌드 결과물 UI 패치 (변경 의도가 코드로 정리됨 — 소스 이식 시 참고)
│  ├─ update-program-images.mjs★ 특화사업 사진 교체(4:3 1200×900 JPEG 변환)
│  └─ make-logo-transparent.mjs★ 로고 흰 배경 → 투명
├─ data/
│  ├─ facilities.json          ★ 변환된 기관 데이터 155건 (앱이 쓰는 최종 스키마)
│  ├─ highlights-overrides.json★ 기관별 "주요 사업" 수동 요약 (40여 곳)
│  ├─ news-keywords.json       ★ 뉴스 수집 키워드·기준일·제외 목록
│  └─ backup/                  엑셀 원본, 로고 원본
├─ public/
│  ├─ news/index.html          ★ 정보나눔 페이지 (독립 정적 HTML, 사이트 디자인 재현)
│  ├─ news.json                ★ 수집된 뉴스 (자동 갱신)
│  ├─ logo.png                 ★ 투명 배경 로고
│  └─ assets/                  브라우저 번들 + 이미지 (program-N-*.jpg 7장 ★)
├─ server/                     서버 렌더링 코드 (빌드 결과물, 패치됨)
├─ client/                     public 과 동일한 복사본
├─ .github/workflows/update-news.yml ★ 매일 06:00 뉴스 수집 GitHub Actions
└─ README.md                   운영 안내
```
★ = 어떤 도구로 옮겨도 그대로 재사용 가능한 파일. 나머지는 빌드 결과물이라 원본 소스가 있으면 필요 없습니다.

상위 폴더에 있는 원본 데이터도 함께 전달하세요.
- `강남구 통합돌봄 사업대상기관 목록(0911).xlsx` — 기관 데이터의 단일 원본 (수정이력(0915) 시트 포함)
- `사진폴더/` — 특화사업 사진 7장 원본

---

## 3. 소스에 구현해야 할 변경 목록 (2026-09-15 작업분)

원본 소스에서 다시 빌드할 때 아래를 구현하면 현재 화면과 동일해집니다. 괄호는 원본에서 손댈 것으로 추정되는 파일입니다.

### 3-1 데이터 (`src/routes/services.tsx` 의 FACILITIES 상수 또는 별도 데이터 파일)
- 기관 데이터를 `data/facilities.json` 스키마로 교체 (155건). 필드:
  `id, name, target, phone, homepage, address, services[], detail, content, highlights[], targetDetail, zip, tags[]`
- 권장: 데이터를 코드에 넣지 말고 `/facilities.json` 을 fetch 하도록 변경 (엑셀 갱신 시 재빌드 불필요)
- 자료 기준일 상수 `FACILITY_SOURCE_DATE = "2026-09-11"`

### 3-2 기관 카드 (`services.tsx`)
- 원문(`content`) 문단 삭제 → **"주요 사업"** 소제목 + `highlights[]` 불릿 목록 (15px, 진한 회색 `#31535a`)
- 전화번호를 `<a href="tel:…">` 링크로
- 카드 하단: [기관 홈페이지 ↗] [자세히] [지도에서 보기 ↗] 3개
- **"자세히" 상세 팝업**: 이용대상(targetDetail 우선), 서비스 세부, 전화(tel), 주소 + 네이버 지도 링크, 홈페이지, 주요 사업 목록, 원문 전체(스크롤 영역), 자료 기준일과 수정 요청 안내. Esc·바깥 클릭으로 닫기, 열려 있는 동안 body 스크롤 잠금

### 3-3 기관 목록 필터·검색 (`services.tsx`)
- **이용대상 필터** 행 추가: 전체 / 노인 / 장애인 / 노인·장애인 / 지역주민 (건수 표시). 서비스유형 행 위에 배치
- **URL 파라미터**: `q`(검색어), `category`(서비스 대분류), `target`(이용대상), `tags`(쉼표 구분, 세부유형·대상·주요사업 중 하나라도 포함). 첫 렌더는 기본값으로 하고 `useEffect` 에서 URL 을 읽어 적용(서버·클라이언트 첫 렌더 일치 → 하이드레이션 오류 방지)
- `tags` 적용 중이면 "상황 필터: 방문진료 · 긴급돌봄 ✕" 해제 칩 표시
- 검색 대상에 `highlights`, `content` 포함
- 결과 요약 문구에 이용대상 포함, "검색 결과 · 자료 기준 2026-09-11" 라벨
- 하단 데이터 안내에 "정보가 실제와 다르면 강남구 통합돌봄 상담 02-3446-9736(tel 링크)으로 알려주세요"

### 3-4 홈 (`src/routes/index.tsx`)
- 상황 카드 5개 링크를 필터 URL 로:
  - 퇴원 후 돌봄 → `/services?tags=방문진료,긴급돌봄,재택의료,건강주치의`
  - 혼자 사는 어르신 → `/services?target=노인&tags=일상돌봄,마음돌봄,긴급돌봄,안부`
  - 거동이 불편한 분 → `/services?tags=방문목욕,방문진료,장기요양보험,활동지원`
  - 가족 돌봄 부담 → `/services?tags=장기요양보험,주간보호,데이케어,일시재가`
  - 생애 말기 돌봄 → `/services?tags=재가암,호스피스,방문진료,마음돌봄`
- **정보나눔 섹션** (3단계 섹션 뒤, 지도/상담 섹션 앞): `/news.json` 을 fetch 해 관련도 2(핵심) 글 최신 3건 카드(날짜 `YYYY.MM.DD`, 담당부서, 7일 이내 "새 글" 배지, 주제 태그 2개). "정보나눔 전체 보기 →" 버튼은 `/news`
- "소식·문의" 라벨 → **"상담·문의"** (메뉴와 홈 블록 모두)
- 특화사업 7번 **"강남 AI 포용케어" → "방문약물관리"** (문구는 `tools/patch-ui.mjs` 의 `PROGRAM7` 참고, 담당부서 확인 필요)
- 특화사업 이미지 7장 → `public/assets/program-N-*.jpg` (1200×900). 팝업 `<img width={1200} height={900}>`

### 3-5 공통 헤더 (`SiteHeader` 컴포넌트)
- 메뉴에 **"정보나눔"(`/news`)** 추가 (상담·문의 앞)
- **"어두운 화면" / "큰 글씨" 토글 버튼**: 아이콘 + 글자 라벨, 켜지면 진한 배경 + "켜짐" 표기. 데스크톱(≥1024px)은 헤더 우측, 모바일은 햄버거 메뉴 안(2열)
- `<html>` 클래스 `dark`, `gc-big` 로 상태 표현. localStorage `gc-theme`(dark|light), `gc-font`(big|normal). 첫 방문은 `prefers-color-scheme` 따름. 첫 화면 깜빡임 방지를 위해 인라인 스크립트로 `<head>` 또는 헤더 최상단에서 클래스 적용
- 로고 이미지는 투명 PNG(`public/logo.png`) 사용, `mix-blend-multiply` 제거

### 3-6 스타일 (Tailwind 설정/전역 CSS)
- **어두운 모드**: 색상 대응표는 `tools/patch-ui.mjs` 의 `DARK` 와 `DARK_CSS` 참고 (배경 `#0b1517`/카드 `#111f23`/글자 `#e7f1f0`/보조 `#a3b5b8`/청록 `#62d6cb`/오렌지 `#ffa27a`/테두리 `#274044`). 소스에서는 Tailwind `dark:` variant(`darkMode: "class"`)로 구현 권장
- 명도 대비 보정: 보조 회색 `#6a7f84→#516a70`, `#71858a→#566f76`, `#657c82→#4f666c`, 링크 `#07877f→#077b74`, 오렌지 태그 `#c56239→#b3502a`
- `word-break: keep-all; overflow-wrap: anywhere` (본문 제목·문단)
- 최소 글자: 10px→11px, 11px→12px. "큰 글씨" 모드 `html.gc-big { font-size: 112.5% }` + px 단위 글자 보정
- 포커스 링(2px 청록) 추가 권장 — 현재 미구현

### 3-7 정보나눔 (`/news`)
- 현재는 독립 정적 HTML(`public/news/index.html`). 소스가 있으면 라우트 `src/routes/news.tsx` 로 옮기고 같은 UI 유지: 검색, 주제 칩(전체/핵심 소식/키워드별), 15건씩 더 보기, 새 글 배지, 관련 소식 표시, 출처·공공누리 안내
- 데이터 `public/news.json` 스키마: `{ source, keywords, coreKeywords, updatedAt, count, items:[{ id, title, link, date, dept, summary, tags[], relevance(2=제목 일치, 1=본문 일치) }] }`

---

## 4. 데이터·자동화 운영 (도구와 무관하게 유지)

```bash
npm install                 # exceljs 만 필요 (뉴스 수집은 의존성 없음)
npm run data:check          # 엑셀 검증
npm run data                # 엑셀 → 데이터 반영 (+ 파일명 해시 갱신)
node tools/fetch-news.mjs   # 뉴스 수집 (--seed: 과거 글까지)
node tools/update-program-images.mjs  # 사진폴더 → 특화사업 이미지
```
- 엑셀 정제 규칙, 주요 사업 자동 요약 규칙, 수동 요약(`highlights-overrides.json`)은 `build-data.mjs` 에 있음
- 뉴스 자동 갱신: PC 실행 시 `run.mjs` 가 24시간마다, 배포 시 `.github/workflows/update-news.yml`(매일 06:00 KST)
- 미해결 데이터: 건강보험공단 주소(관내 지사 3곳 중 미확정), 세스코·인스케어코어 이용대상

---

## 5. 도구별 이어가기 방법

### Genspark
1. 이 `webapp` 폴더를 GitHub 저장소(`gnwf2014-afk/-260803`, 브랜치 `genspark_ai_developer`)에 커밋·푸시하면 Genspark 가 현재 상태를 이어받습니다.
   ```bash
   cd webapp && git add -A && git commit -m "feat: 2026-09-15 데이터 파이프라인, 정보나눔, UI 개선(어두운 모드 등)" && git push origin genspark_ai_developer
   ```
2. Genspark 에는 반드시 이렇게 요청하세요: **"빌드 결과물을 직접 고치지 말고, 원본 소스(`build-site`)에 HANDOVER.md 3장의 변경을 구현한 뒤 다시 빌드해 주세요. tools/, data/, public/news, public/news.json, public/logo.png, program-N-*.jpg 는 그대로 유지."**

### Lovable
1. Lovable 은 원본 소스 프로젝트에서만 작업합니다. 이 폴더를 그대로 올리면 안 되고, **기존 Lovable 프로젝트(build-site)를 열거나 그 프로젝트가 연결된 GitHub 저장소**를 찾아야 합니다. (Lovable 프로젝트 → Settings → GitHub 연결 확인)
2. 원본 프로젝트에 다음을 추가: `tools/`, `data/`, `public/news/`, `public/news.json`, `public/logo.png`, `public/assets/program-*.jpg`, `.github/workflows/update-news.yml`, 이 문서.
3. Lovable 채팅에 이 문서의 **3장을 그대로 붙여 넣고** "이 변경을 구현해 달라"고 요청하세요. 항목이 많으니 3-1 → 3-2 → … 순서로 나눠 요청하는 것이 안정적입니다.
4. 데이터 갱신은 Lovable 밖에서 `npm run data` 로 하고 결과 JSON 만 커밋하는 흐름을 권장합니다.

### 일반 개발자 / 다른 AI 도구
- 이 폴더 + 상위 폴더의 엑셀·사진폴더 + 이 문서를 전달. `npm start` 로 현재 화면을 바로 확인할 수 있습니다.
- 원본 소스가 끝내 없으면, 이 빌드 결과물을 기준으로 새 프로젝트를 만들 때 3장의 목록이 요구사항 명세가 됩니다.
