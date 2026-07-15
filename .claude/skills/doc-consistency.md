# Skill: Doc Consistency (최종 점검)

## Objective

- 동일 회사 제출용 문서(Career/Resume/Coverletter) 간 사실 충돌 및 표현 불일치 제거.

## Procedure

1. **Target**: `_output/{company}/` 내 모든 문서 소스 추출 (`.md` / `.txt`).
   생성물(`.html` / `.pdf`)은 대상 아님 — 소스만 본다.
2. **Extract**: 수치 성과, 트러블슈팅 원인/해결, 기술 선택 이유, 재직 기간/역할.
3. **Audit**:
   - **사실 충돌 (Critical)**: 수치나 사건 내용이 다른 경우 (반드시 수정).
   - **표현 불일치 (Minor)**: 강도, 어조, 용어 차이 (사용자 선택).
4. **Report**: 충돌 내역을 `[사실 충돌]` / `[표현 불일치]` 섹션으로 구분하여 제시.
5. **Fix**: 사용자 승인 후 지정 파일 일괄 수정.

## Rules

- 요약/생략에 따른 길이 차이는 불일치로 간주하지 않음.
- 정량적 수치(Before/After)의 동일성 보증에 집중.