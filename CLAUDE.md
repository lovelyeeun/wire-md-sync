# wire-md-sync — 에어스토어 스펙 리뷰어 개선 프로젝트

## 프로젝트 목적
`spec-reviewer-v2.html`을 개선하는 프로젝트.
현재는 문서를 번호 순서대로 나열하는 수준이며, 아래 4가지 문제를 해결하는 것이 목표.

1. 스펙 섹션을 읽어도 와이어프레임의 어느 영역을 봐야 하는지 알 수 없음
2. 원본 MD 문서의 어느 부분과 연결되는지 추적 불가
3. 화면 단위가 아닌 번호 순서 네비게이션
4. 확인 로직이 체크박스 이진 방식으로 너무 단순

---

## 파일 구조

```
wire-md-sync/
├── CLAUDE.md                                  ← 이 파일
├── spec-reviewer-v2.html                      ← 개선 대상 (메인)
├── spec-reviewer.html                         ← 구버전 (참고용)
├── airsupply-store-wireframe-v7-updated.html  ← 와이어프레임 (Anchor ID 추가 예정)
├── airsupply-store-spec-v1.1.docx             ← 최신 스펙 문서
├── airsupply-store-spec-v1.0.docx             ← 구버전 스펙
├── 에어서플라이_컬러사용_원칙.md               ← 디자인 시스템 (MD 참조 대상)
├── session-log-2026-04-06-store-spec.md       ← 작업 기록 (MD 참조 대상)
├── spec-data.json                             ← (생성 예정) SPEC_DATA 분리
└── anchor-map.json                            ← (생성 예정) 스펙↔와이어프레임 매핑
```

---

## 개선 우선순위 (P0 → P3)

### P0 — 환경 세팅 ✅ 완료 조건: GitHub push + 로컬 서버 정상 실행
- GitHub 레포: `wire-md-sync`
- 로컬 서버: `python3 -m http.server 3000`
- 접속: `localhost:3000/spec-reviewer-v2.html`

### P1a — 화면 기반 네비게이션
**목표:** 번호 순서 네비게이션 → 화면 탭 기반으로 전환

- `SPEC_DATA` 각 섹션에 `screen` 필드 추가
- 가능한 값: `"store-home"` | `"snack-package"` | `"industry"` | `"promotions"` | `"common"` | `"ia"`
- 툴바를 화면 탭으로 교체: `[스토어 홈] [간식 패키지] [업종별 탐색] [기획전] [공통] [IA]`
- 탭 클릭 시 해당 screen 값의 섹션만 필터링

### P1b — Anchor Map 연동 (P1a와 병렬 가능)
**목표:** 스펙 섹션 클릭 → 와이어프레임 자동 스크롤

- `airsupply-store-wireframe-v7-updated.html` 주요 섹션에 `id` 앵커 추가
  - 예: `id="wf-hero"`, `id="wf-repeat-purchase"`, `id="wf-industry-rec"` 등
- `anchor-map.json` 생성: `{ "sec4-1": "wf-hero", "sec4-2": "wf-repeat-purchase", ... }`
- `spec-reviewer`에서 섹션 클릭 시 `postMessage`로 iframe 스크롤 명령 전달
- ⚠️ postMessage는 반드시 로컬 서버(localhost) 환경에서만 동작

### P2 — MD 문서 출처 연결
**목표:** 스펙 섹션마다 관련 MD 문서 인라인 참조

- `SPEC_DATA` 각 섹션에 `refs` 배열 추가
  ```json
  "refs": [{ "file": "에어서플라이_컬러사용_원칙.md", "section": "Rule-9" }]
  ```
- 섹션 우측에 📄 아이콘 표시
- 클릭 시 해당 MD 파일을 `fetch`로 로드하여 관련 섹션만 슬라이드 패널로 표시
- ⚠️ `fetch()`는 로컬 서버 환경 필수 (file:// 프로토콜 불가)

### P3 — 리뷰 상태 고도화
**목표:** 체크박스 이진 → 다단계 상태 + 코멘트

| 현재 | 개선 후 |
|------|---------|
| ☐ / ☑ (이진) | ⚠️ 미확인 / 🔄 검토중 / ✅ 확정 / ❌ 블록 |
| 상태만 | 상태 + 코멘트 입력 필드 |

---

## 주요 구현 참고사항

- `SPEC_DATA`는 `spec-data.json`으로 분리해서 `fetch`로 불러오는 방식으로 전환
- 와이어프레임 iframe ↔ 스펙 패널 통신은 `window.postMessage` 사용
- 컬러 시스템은 `에어서플라이_컬러사용_원칙.md` 기준 준수
  - Primary CTA: `#FFB800`
  - 확정(초록): `#00B96B` / `#F0FFF4`
  - 경고(노랑): `#FFB800` / `#FFF8E1`
  - 블록(빨강): `#FF3131`
- 폰트: Noto Sans KR (이미 적용됨)

---

## 로컬 서버 실행

```bash
# 프로젝트 폴더에서
python3 -m http.server 3000

# 브라우저에서 접속
open http://localhost:3000/spec-reviewer-v2.html
```

---

## 작업 시작 방법

```bash
cd "/Users/eunseo/Documents/Claude/Projects/스토어 개편"
claude
```
