# Skill: Apply Workflow (JD → 지원 완료)

## Objective

JD 공유 시 Fit 검토 → 지원 결정 → 문서 작업 → 품질 검증 → 상태 업데이트까지 단일 흐름으로 진행. 중단 시 재개 가능.

## Invocation

```
/apply-workflow <slug>
```

- `slug`: 이 지원 건을 식별하는 임의 식별자. 회사명에 국한되지 않음.
  - 예: `mintrocket`, `kakao_client`, `kakao_server`, `ncsoft_2025`, `startup_unity`
- 슬러그는 `_output/{slug}/` 폴더명이자 파일 prefix로 사용됨.
- 같은 회사에 복수 지원 시 → 슬러그로 구분: `nexon_client`, `nexon_server`
- 슬러그 없이 호출 시 → AI가 JD에서 적절한 슬러그 제안 후 확인.

---

## Procedure

### Phase 0: Session Load + 재개 판단

**토큰 전략**: `CURRENT_TARGET.md` + 파일 목록만 로드. 코퍼스 전체 로드 금지.

1. `_workspace/CURRENT_TARGET.md` 로드.
2. `_output/{slug}/_context.md` 존재 확인.
   - 존재하고 `## Workflow State`의 `last_completed_phase` 값이 있으면 → 재개 제안:
     ```
     [{slug}] 이전 작업이 Phase {N} 완료 후 중단되었습니다.
     Phase {N+1}부터 재개할까요? (예 / 아니오 — 아니오 선택 시 처음부터)
     ```
   - 재개 선택 시 해당 Phase로 점프. `selected_cards`, `pending_docs` 복원.
3. 신규 진행 시: `_workspace/past_jd_analyses/` 에서 유사 도메인/포지션 키워드 매칭 → 있으면 해당 파일만 로드.

---

### Phase 1: Fit Check

**토큰 전략**: `_modules/project_modules.md` 카드 제목 + 요약 첫 줄만 스캔. 전체 카드 본문 로드 금지.

1. JD에서 추출: 회사명, 포지션, 필수/우대 스택, 팀 규모·문화.
2. 지원자 프로필(`CURRENT_TARGET.md`)과 매칭 점수 추정.
3. 아래 형식으로 Fit Report 제시:

```
## 회사 개요
[회사명 / 팀 규모 / 도메인]

## 포지션 적합도
| 항목 | JD 요구 | 지원자 현황 | 판단 |
|------|---------|-----------|------|
| ...  | ...     | ...       | O/△/X |

## 강점 / 리스크
- 강점: ...
- 리스크: ...

## 추천 의견
[지원 권장/보류/비추천 + 한 줄 근거]
```

**→ Gate 1**: "지원하시겠습니까?" — 미지원 시 종료.

---

### Phase 2: 문서 범위 확인

1. `_output/{slug}/` 존재 여부 확인.
2. 필요 문서 목록 제시 (이력서·경력기술서·자기소개서).
3. 기존 파일 있으면 → 업데이트 vs 신규 분류해서 제시.

**→ Gate 2**: 문서 작업 범위 확인.

---

### Phase 3: Context 초기화 + 카드 선별

1. `_modules/project_modules.md` 전체 로드 (이 단계에서만 1회).
2. JD와 매칭되는 카드 선별 및 사유 제시.
3. `_output/{slug}/_context.md` 생성 또는 업데이트:
   - JD 요약, Fit 분석, 카드 선택 근거 기록.
   - `## Workflow State` 섹션 작성:
     ```
     ## Workflow State
     slug: {slug}
     last_completed_phase: 3
     completed: [0, 1, 2, 3]
     pending_docs: [resume, coverletter]
     selected_cards: [카드명1, 카드명2, ...]
     ```

**→ Gate 3**: 카드 선별 결과 승인 후 빌드 진행.

---

### Phase 4: 문서 빌드

**단일 문서**: `resume-build` 스킬 직접 실행.

**복수 문서 (2종 이상)**: 문서별 병렬 Subagent 분리.
- 각 Subagent 주입 데이터: JD 요약, `selected_cards` 목록, 대상 문서 타입, `slug`.
- 코퍼스 중복 로드 방지: Phase 3에서 선택 완료된 카드 내용만 전달.
- Subagent별 작업: `_base/` 기반으로 `_output/{slug}/{slug}_{doc_type}.html` 생성.

완료 후 `_context.md` Workflow State 업데이트:
```
last_completed_phase: 4
pending_docs: []
built_docs: [resume, coverletter]
```

---

### Phase 5: 품질 검증

조건부 실행 (해당 조건만 수행):

| 조건 | 실행 스킬 |
|------|---------|
| 이력서 포함 | `resume-review` |
| 문서 2종 이상 | `doc-consistency` |
| 두 조건 모두 | 병렬 Subagent로 동시 실행 |

---

### Phase 6: 마무리

1. `_workspace/CURRENT_TARGET.md` 업데이트 — 슬러그 + 회사명 추가 또는 상태 변경.
2. JD 분석 아카이브: `_workspace/past_jd_analyses/{slug}_{YYYY-MM-DD}.md` 저장.
3. `sync-module` 트리거 조건 충족 여부 확인 → 해당 시 실행 제안.
4. `_context.md` Workflow State 최종 업데이트:
   ```
   last_completed_phase: 6
   status: done
   ```

---

## Rules

- **슬러그 일관성**: 모든 경로·파일명·State 기록에서 동일 슬러그 사용.
- **재개 우선**: Phase 0에서 중단 기록 확인 전 새 작업 시작 금지.
- **코퍼스 로드 시점**: Phase 3에서만 1회. Phase 1은 요약부(제목+첫 줄)만 허용.
- **Subagent 주입**: Phase 4 Subagent에 코퍼스 전체 로드 지시 금지 — 선택 카드 내용만 전달.
- **허구 금지**: 카드 데이터 외 임의 성과 작성 금지.
- **템플릿 보호**: `_base/` 파일 수정 금지.
