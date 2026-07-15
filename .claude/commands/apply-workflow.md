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

## 문서 타입 정의

모든 문서의 소스는 `.md`. 생성물은 `tools/build_doc.py` 가 만든다.

| 타입 | 설명 | 빌드 스킬 | 소스 (관리 대상) | 생성물 |
|------|------|---------|---------|---------|
| `resume` | 이력서 | `resume-build` | `{slug}_resume.md` | `.html` |
| `career` | 경력기술서 | `resume-build` (career 모드) | `{slug}_career.md` | `.html` |
| `coverletter` | 자기소개서 | `coverletter-build` | `{slug}_coverletter.md` | `.txt` |
| `portfolio` | 포트폴리오 | `portfolio-build` | `{slug}_portfolio.md` | `.html` |

생성물(`.html` / `.txt` / `.pdf`)은 손대지 않는다. 고칠 일이 있으면 `.md` 를 고치고 다시 빌드한다.

---

## Procedure

### Phase 0: Session Load + 재개 판단

**토큰 전략**: `CURRENT_TARGET.md` + 파일 목록만 로드. 코퍼스 전체 로드 금지.

1. `_workspace/CURRENT_TARGET.md` 로드.
2. `_wiki/` 폴더 존재 확인 → 있으면 **wiki 강화 모드** 플래그 설정 (없으면 기존 흐름 동일하게 진행).
3. `_output/{slug}/_context.md` 존재 확인.
   - 존재하고 `## Workflow State`의 `last_completed_phase` 값이 있으면 → 아래 형식으로 재개 제안:
     ```
     [{slug}] 이전 작업이 Phase {N} 완료 후 중단되었습니다.

     문서 현황:
       완료: {built_docs}
       미완료: {pending_docs}

     Phase {N+1}부터 재개할까요? (예 / 아니오 — 아니오 선택 시 처음부터)
     ```
   - 재개 선택 시 해당 Phase로 점프. `selected_cards`, `selected_docs`, `pending_docs`, `built_docs` 복원.
   - **Phase 4 재개 시**: `pending_docs` 중 첫 번째 문서부터 이어서 빌드.
4. 신규 진행 시: `_workspace/past_jd_analyses/` 에서 유사 도메인/포지션 키워드 매칭 → 있으면 해당 파일만 로드.

---

### Phase 1: Fit Check

**토큰 전략**: `_corpus/project_modules.md` 카드 제목 + 요약 첫 줄만 스캔. 전체 카드 본문 로드 금지.

**[wiki 강화 모드]** `_wiki/identity.md` + `_wiki/strengths.md` 추가 로드 (합산 약 2~3KB).

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

## 관련 정체성 태그 [wiki 강화 모드 시 추가]
- [_wiki/identity.md에서 이 JD와 매칭되는 태그명]: [한 줄 근거]

## 추천 의견
[지원 권장/보류/비추천 + 한 줄 근거]
```

**→ Gate 1**: "지원하시겠습니까?" — 미지원 시 종료.

---

### Phase 2: 문서 범위 확인

1. `_output/{slug}/` 내 기존 파일 목록 확인.

2. 아래 형식으로 문서 선택 제시:

   ```
   어떤 문서를 작성할까요? (복수 선택 가능)

   [ ] resume      — 이력서         ({slug}_resume.md → .html)
   [ ] career      — 경력기술서     ({slug}_career.md → .html)
   [ ] coverletter — 자기소개서     ({slug}_coverletter.md → .txt)
   [ ] portfolio   — 포트폴리오     ({slug}_portfolio.md → .html)

   기존 파일:
     {slug}_resume.md — 업데이트 또는 재작성 선택 가능
     (없으면 이 줄 생략)
   ```

3. 선택된 문서 목록을 `selected_docs`로 확정, `pending_docs`에 전체 복사.

**→ Gate 2**: 문서 범위 최종 확인.

---

### Phase 3: Context 초기화 + 카드 선별

**[wiki 강화 모드]** `_wiki/relationships.md` 추가 로드.

1. `_corpus/project_modules.md` 전체 로드 (이 단계에서만 1회).
2. JD와 매칭되는 카드 선별 및 사유 제시.
3. **[wiki 강화 모드]** 선별된 카드 간 관계 엣지 확인 (`_wiki/relationships.md`의 "JD 매칭 시 카드 간 연결 활용 포인트" 참조) → 해당 연결이 있으면 `_context.md`에 "카드 간 연결 포인트" 섹션 추가 기록.
4. `_output/{slug}/_context.md` 생성 또는 업데이트:
   - JD 요약, Fit 분석, 카드 선택 근거 기록.
   - `## Workflow State` 섹션 작성:
     ```
     ## Workflow State
     slug: {slug}
     last_completed_phase: 3
     completed: [0, 1, 2, 3]
     selected_docs: [resume, career, coverletter]
     pending_docs: [resume, career, coverletter]
     built_docs: []
     selected_cards: [카드명1, 카드명2, ...]
     ```

**→ Gate 3**: 카드 선별 결과 승인 후 빌드 진행.

---

### Phase 4: 문서 빌드

`pending_docs`의 문서를 순서대로 빌드. 각 문서 완료 시 즉시 State 업데이트 후 다음 문서로 진행.

**문서별 스킬 매핑**: 위 "문서 타입 정의" 표를 그대로 따른다 (여기서 재서술하지 않음).

모든 문서는 `.md` 소스 작성 후 반드시 빌드까지 수행:
`python tools/build_doc.py _output/{slug}/{slug}_{doc_type}.md`

**단일 문서**: 해당 스킬 직접 실행.

**복수 문서 (2종 이상)**: 문서별 병렬 Subagent 분리.
- 각 Subagent 주입 데이터: JD 요약, `selected_cards` 목록, 대상 문서 타입, `slug`.
- 코퍼스 중복 로드 방지: Phase 3에서 선택 완료된 카드 내용만 전달.
- **[wiki 강화 모드]** Subagent 추가 주입: `_wiki/identity.md`에서 선택 카드 관련 태그 발췌 + `_wiki/timeline.md`에서 해당 카드들이 속하는 기(期) 정보 → 자기소개서 도입부 및 경력기술서 요약문 서술에 활용.

**문서 완료 시 State 즉시 업데이트** (예: resume 완료 후):
```
last_completed_phase: 4
pending_docs: [career, coverletter]
built_docs: [resume]
```

모든 문서 완료 시:
```
last_completed_phase: 4
pending_docs: []
built_docs: [resume, career, coverletter]
```

---

### Phase 5: 품질 검증

조건부 실행 (해당 조건만 수행):

| 조건 | 실행 스킬 |
|------|---------|
| `resume` 또는 `career` 포함 | `resume-review` |
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
- **문서 상태 즉시 기록**: 각 문서 빌드 완료 직후 `_context.md`의 `pending_docs` / `built_docs` 업데이트. 세션 중단에 대비.
- **코퍼스 로드 시점**: Phase 3에서만 1회. Phase 1은 요약부(제목+첫 줄)만 허용.
- **Subagent 주입**: Phase 4 Subagent에 코퍼스 전체 로드 지시 금지 — 선택 카드 내용만 전달.
- **허구 금지**: 카드 데이터 외 임의 성과 작성 금지.
- **템플릿 보호**: `_template/` 파일 수정 금지.
