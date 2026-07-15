# Command: /wiki-update

## Objective

`_corpus/` 전체를 기반으로 `_wiki/` 5개 파일을 재컴파일.
`_wiki/` 쓰기 권한은 이 커맨드에만 있음.

## Invocation

```
/wiki-update
```

## Procedure

### Step 1: 소스 로드

1. `_corpus/project_modules.md` 전체 로드.
2. `_corpus/` 내 개별 상세 카드 파일 목록 확인 후 로드:
   - `ai_resume_system.md`
   - `zoneflow.md`
   - `prototype_combat.md`
   - `portrait_conversation.md`
   - `tmpkit.md`
   - `live_service_webboard.md`

### Step 2: 재컴파일 대상 파일

아래 5개 파일을 재컴파일:

| 파일 | 컴파일 기준 |
|------|-----------|
| `_wiki/identity.md` | 전체 카드에서 반복 패턴 → 정체성 태그 4~5개 도출 |
| `_wiki/strengths.md` | 전체 카드에서 cross-cutting 강점 테마 집계 |
| `_wiki/timeline.md` | 카드의 연도 정보 기반 커리어 아크 5기 서사 |
| `_wiki/relationships.md` | 카드 간 개념 상속·기술 재사용·방법론 연속성 |
| `_wiki/tech_map.md` | 기술 스택 집계 + 아키텍처 패턴 이력 + 기술 선택 패턴 |

### Step 3: 컴파일 원칙

- **허구 금지**: 카드 데이터 외 내용 추가 금지. 해석은 가능하나 사실 추가 불가.
- **근거 명시**: 각 항목에 "근거 카드: [카드명 (번호)]" 형식으로 출처 표기.
- **헤더 유지**: 각 파일 상단 `> 소스: ... | last_compiled: YYYY-MM-DD` 헤더를 오늘 날짜로 업데이트.

### Step 4: 변경 내역 제시 및 확인

1. 각 파일의 주요 변경 사항 요약 제시 (추가·수정·삭제 항목).
2. 사용자 확인 후 Write.

### Step 5: README 업데이트

`_wiki/README.md`의 `last_compiled` 날짜를 오늘 날짜로 업데이트.

## Rules

- **단방향 흐름**: `_corpus/` → `_wiki/`. 역방향(wiki → corpus) 절대 금지.
- **전체 재컴파일**: 부분 업데이트 금지. 5개 파일 전체 재컴파일.
- **사용자 확인 필수**: 변경 내역 제시 전 Write 금지.
