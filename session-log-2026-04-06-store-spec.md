# 세션 기록 — 에어스토어 스토어 개편 기획 명세서

> 날짜: 2026-04-06
> 세션 ID: ecstatic-upbeat-keller
> 세션 폴더 경로: `/Users/eunseo/Library/Application Support/Claude/local-agent-mode-sessions/ecstatic-upbeat-keller/`
> 작업 파일 위치: 사용자 선택 폴더 → `스토어 개편/`

---

## 1. 작업 요약

와이어프레임 HTML 파일(`airsupply-store-wireframe-v7-updated.html`)을 분석하여 개발자용 기능 기획 명세서(docx)를 v1.0으로 작성했다. 이후 은서님 확인을 통해 비로그인 케이스 없음, 업종 개인화 방향, 간식 추천 상품 DB 연동 방식, URL 라우팅 정책 등을 확정했고, AirSupply PostgreSQL DB를 직접 조회(`product_purchase_frequency`, `company_tags_entity`, `product_entity` 등)하여 API 명세를 구체화한 v1.1을 최종 저장했다.

---

## 2. 주요 결정사항

- **비로그인 케이스 없음**: `airsupply.kr`에서 로그인 후 `office.airsupply.kr` 진입 구조이므로, 비로그인 진입 자체가 불가. 관련 분기 처리 불필요.
- **개인화 방향 = 고객사(회사) 단위**: 개인 유저 수준이 아닌, 회사 단위 업종/구매 데이터 기반으로 개인화. `company_tags_entity` + `company_tag_bridge_entity` 시스템 활용.
- **반복구매 상품 = 전체 에어서플라이 고객 기준**: `product_purchase_frequency` 테이블 집계. 구매주기는 `(endDate - startDate) ÷ purchaseCount` days로 계산.
- **간식 추천 상품**: `product_entity`에 `snackType` 컬럼(enum: sweet/salty/health/drink/null) 추가하는 백엔드 작업 필요. `isStore=true` 상품 중 snackType not null인 것을 간식 풀로 관리. MVP는 프론트에서 예산/비율 기반으로 선택하는 방식 권장.
- **URL 라우팅**: 스토어 탭 전환 시 URL 변경 확정. 탭 내 서브 페이지(빌더 위자드, 기획전 상세 등) URL 변경 여부는 개발팀 협의 필요.

---

## 3. 수정된 파일 목록

| 파일 | 변경 내용 | 현재 상태 |
|------|----------|----------|
| `스토어 개편/airsupply-store-wireframe-v7-updated.html` | 읽기 전용 (분석 대상, 수정 없음) | 정상 |
| `스토어 개편/airsupply-store-spec-v1.0.docx` | 최초 기획 명세서 (와이어프레임 기반) | 정상 (v1.1로 대체됨) |
| `스토어 개편/airsupply-store-spec-v1.1.docx` | DB 확인 후 API 명세 구체화, 확인 항목 업데이트 | **최신 버전** |
| `스토어 개편/session-log-2026-04-06-store-spec.md` | 이 파일 | 정상 |

> ⚠️ JS 생성 스크립트(`create_spec_doc.js`)는 세션 내부 임시 폴더(`/sessions/ecstatic-upbeat-keller/`)에만 있어 다음 세션에서는 접근 불가. 필요 시 아래 "현재 파일 구조" 참고하여 재작성.

---

## 4. 기획 명세서 구조 (v1.1 기준)

`airsupply-store-spec-v1.1.docx` 총 12개 섹션, ~800 단락 분량.

| 섹션 | 내용 |
|------|------|
| 1. 문서 개요 | 목적, 범위, 관련 리소스, 주의사항 범례 |
| 2. 전체 IA | 앱 레이아웃 구조, 스토어 페이지 트리, URL 패턴 |
| 3. 공통 컴포넌트 | 사이드바, 상품 카드(prod-card), 장바구니 버튼, 토스트 |
| 4. 스토어 홈 탭 | Hero(검색+캐러셀), SEC1(반복구매), SEC2(업종별), SEC3(카테고리BEST), SEC4(브랜드), SEC5(간식CTA) |
| 5. 간식 패키지 탭 | Page0(목록), Page Detail(세트상세), Page1(빌더 3-Step 위자드) |
| 6. 업종별 상품탐색 탭 | 트리 사이드바 + 상품 영역 |
| 7. 기획전 탭 | 목록 + 상세 |
| 8. 브랜드 검색 결과 | 오버레이 페이지 |
| 9. 모달 명세 | 상품폴더 추가 모달 |
| 10. 개발 우선순위 | P0/P1/P2 (MVP ~ Phase 2) |
| 11. PM 확인 필요 사항 | 확정 5개 ✅ / 미확정 7개 ⚠️ |
| 12. 변경 이력 | v1.0 → v1.1 |

---

## 5. 적용된 스펙 값

### 기획문서 컬러 시스템 (docx)
```js
primary: '222222'   // 제목
gray1:   '5F5F5F'   // 본문
amber:   'FF9F0F'   // 미확정 경고
green:   '00B96B'   // 확정 표시
blue:    '1A73E8'   // API/링크
lightAmber: 'FFF8EB' // 경고 박스 배경
lightGreen: 'F0FFF4' // 확정 박스 배경
```

### docx 페이지 설정
- US Letter (12240 × 15840 DXA), 여백 1080 DXA (0.75인치)
- 콘텐츠 폭: 9360 DXA
- 폰트: Malgun Gothic (한글), Arial fallback

### DB 주요 테이블 (이번 세션에서 확인)
| 테이블 | 용도 | 비고 |
|--------|------|------|
| `product_purchase_frequency` | 반복구매 집계 | 25,678행, 2026-01~03 |
| `company_tags_entity` | 업종 태그 | Expose: IT, 코워킹스페이스, 병원, 교육 |
| `company_tag_bridge_entity` | 회사-태그 연결 | |
| `product_entity` | 상품 | `isStore` boolean 이미 있음. `snackType` 컬럼 추가 필요 |
| `category_1` | 1단계 카테고리 | 식품(id=9), 문구/사무용품(id=7) 등 |
| `order_info_entity` | 주문 (companyId 포함) | |

### 와이어프레임 핵심 탭 구조
```
스토어 탭바: 스토어 홈 | 간식 패키지 | 업종별 상품탐색 | 기획전
업종 6개: fb(F&B) / space(공간운영) / it(IT오피스) / mfg(제조) / ecom(이커머스) / pro(전문서비스)
간식 유형 5개: daily / health / meeting / energy / hosting
간식 카테고리 4개: sweet(달달) / salty(짭짤) / health(건강) / drink(음료)
```

---

## 6. 미완료 작업

- [ ] 섹션 11 미확정 항목 7개 결정 필요:
  - [ ] 업종 태그(company_tags) → 와이어프레임 6개 클러스터 매핑 기준 정의
  - [ ] `product_entity.snackType` 컬럼 추가 및 기존 상품 일괄 입력 방법
  - [ ] 캐러셀 자동 슬라이드 여부 및 어드민 관리 여부
  - [ ] 업종별 카테고리 구조 DB 관리 vs 하드코딩
  - [ ] 기획전 배너 어드민 관리 여부
  - [ ] 간식 CTA 배너 수치 확정 (3분/25종/87% 고정 카피 vs 실데이터)
  - [ ] 검색 최근 검색어 저장 방식 (로컬 vs 서버)
- [ ] 탭 내 서브 페이지(빌더 위자드, 기획전 상세) URL 변경 여부 개발팀 협의
- [ ] `product_purchase_frequency` 배치 갱신 주기 백엔드 확인
- [ ] 상품 상세 페이지 URL 구조 정의 (별도 명세서)
- [ ] 와이어프레임에 없는 검색 결과 페이지 명세 작성

---

## 7. 알려진 이슈

- `company_tags_entity`의 현재 Expose 태그(IT, 코워킹, 병원, 교육)가 와이어프레임 6개 업종 클러스터와 1:1로 매핑되지 않음. 매핑 로직 별도 정의 필요.
- `product_purchase_frequency` 데이터가 2026-03-31까지만 있음. 현재 날짜 기준으로 최신 데이터 갱신 주기 확인 필요.
- docx 생성 스크립트(`create_spec_doc.js`)가 세션 임시 폴더에만 있어 다음 세션에서 문서 수정 시 재작성 또는 파일 이관 필요.

---

## 8. 다음 세션에서 바로 시작하려면

### 이어서 해야 할 작업
1. 섹션 6 미완료 항목 중 우선순위 높은 것부터 결정 후 `airsupply-store-spec-v1.1.docx` → v1.2로 업데이트
2. 업종 태그 매핑 로직: `company_tags_entity`의 태그명을 6개 클러스터로 연결하는 매핑 테이블 작성 (코드 or DB)
3. `snackType` 컬럼 추가 → 어드민 UI 변경 명세 작성

### 다음 세션에서 바로 실행할 프롬프트 예시
```
이 세션 로그를 읽고, 에어스토어 기획 명세서 작업을 이어서 해줘.
최신 파일은 airsupply-store-spec-v1.1.docx야.
오늘은 [업종 태그 매핑 기준 / snackType 어드민 명세 / 기타]부터 시작하고 싶어.
```

---

### 파일 위치 (Finder에서 찾기)

이 세션의 작업 파일은 아래 경로에 저장되어 있습니다:

```
/Users/eunseo/Library/Application Support/Claude/local-agent-mode-sessions/ecstatic-upbeat-keller/mnt/스토어 개편/
```

주요 파일:
- `airsupply-store-spec-v1.1.docx` — 최신 기획 명세서
- `airsupply-store-wireframe-v7-updated.html` — 원본 와이어프레임
- `session-log-2026-04-06-store-spec.md` — 이 파일

### 다음 세션에서 이어서 작업하는 방법

⚠️ 주의: 이전 세션 폴더를 새 세션에 직접 "폴더 연결"하지 마세요.
폴더가 새 세션에 마운트되면 경로가 바뀌어 이전 세션 데이터에 영향을 줄 수 있습니다.

권장하는 방법:
1. Finder에서 위 경로를 열어 필요한 파일 확인
2. 작업 파일(`airsupply-store-spec-v1.1.docx`, `session-log-...md`)을 새 세션의 outputs 폴더로 복사
3. 새 세션에서 "복사해둔 세션 로그 읽고 이어서 작업해줘"라고 요청

또는 더 간단하게:
1. 새 세션에서 세션 로그 내용을 직접 채팅에 붙여넣기
2. "이 맥락으로 이어서 작업해줘"라고 요청
