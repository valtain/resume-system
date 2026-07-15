# Skill: Module Card (데이터 정제)

## Objective

- `_corpus/project_modules.md`에 프로젝트 카드를 정석 구조로 추가/수정.

## Card Structure (Strict)

1. **요약**: 배경 → 판단 → 방향 (3~5문장, 명사체).
2. **기술 스택**: 선택 이유가 명확한 핵심 스택만 포함.
3. **핵심 설계**: "무엇을 / 왜 / 어떻게" 구조의 설계 결정들.
4. **트러블슈팅**: 원인 특정 과정 필수 포함 (선택 섹션).
5. **성과**: Before/After 수치 및 구조적 달성 내용.
6. **동기화 트리거**: 변경 감지 조건 명시.

## Procedure

1. **Drafting**: 소스 문서(README, 설계서 등) 매핑 가이드에 따라 섹션 작성.
2. **Self-Check**: 작성 완료 체크리스트(명사체, 수치 포함 여부 등) 검증.
3. **Review**: 사용자에게 변경 내용 제시 및 최종 확인.
4. **Write**: `project_modules.md` 업데이트 (기존 번호 유지/밀어내기).

## Rules

- 소스에 없는 내용 허구 작성 금지.
- 외부 도구(HeidiSQL 등) 제외, 핵심 구현 스택 집중.
- 카드 Write 완료 후 → `/wiki-update` 실행 제안.
