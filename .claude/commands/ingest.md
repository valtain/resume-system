# Command: /ingest

## Objective

신규 소스 문서(설계서, CONTEXT.md, README 등)를 corpus 카드로 정제하고, wiki까지 자동 업데이트하는 통합 ingest 파이프라인.

## Invocation

```
/ingest [카드명 또는 slug]
```

- 카드명/slug 생략 시: 소스 문서에서 프로젝트명을 추론해 제안.
- 기존 카드 업데이트: `/ingest nexusframe` → NexusFrame 카드 업데이트 흐름.
- 신규 카드 생성: `/ingest new_project` → 신규 카드 생성 흐름.

## Procedure

### Step 1: 소스 문서 수신

소스 문서 제시 요청:
```
소스 문서를 공유해 주세요. (README, 설계서, CONTEXT.md, 변경 내역 등)
```

### Step 2: 카드 작성/수정

1. 수신한 소스 문서를 기반으로 `.claude/rules/card-spec.md` 규격에 따라 카드 작성/수정.
2. 카드 구조 준수:
   - 요약 (배경 → 판단 → 방향)
   - 기술 스택 (선택 이유 포함)
   - 핵심 설계 (무엇을 / 왜 / 어떻게)
   - 트러블슈팅 (원인 특정 과정 포함, 선택 섹션)
   - 성과 (Before/After 수치)
   - 동기화 트리거
3. 드래프트 제시 → 사용자 확인.

### Step 3: Corpus 업데이트

사용자 확인 완료 후 `_corpus/project_modules.md` 또는 개별 상세 카드 파일 Write.

### Step 4: Wiki 자동 업데이트 (`/wiki-update` 체인)

카드 반영 완료 후:
```
카드 반영이 완료되었습니다. _wiki/ 재컴파일을 진행합니다.
```
→ `/wiki-update` 실행.

### Step 5: 완료 보고

```
## Ingest 완료

- 카드: [카드명] ([신규/업데이트])
- 변경된 wiki 파일: [파일 목록]
```

## Rules

- **소스 없는 허구 금지**: 소스 문서에 없는 내용 카드 작성 금지.
- **wiki 자동 체인**: Step 4 생략 금지. 카드 변경 → wiki 업데이트는 원자적 단위.
- **순서 고정**: corpus 업데이트 완료 확인 후에만 wiki-update 실행.
