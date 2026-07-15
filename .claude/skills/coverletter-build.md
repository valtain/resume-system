# Skill: Coverletter Build (JD 맞춤형 자기소개서 빌드)

## Objective

`_template/coverletter_basic.md`를 기반으로 JD 도메인에 맞는 도입부·사례를 선택해 plain text 자기소개서를 출력.

## Procedure

0. **Context Load**: `_output/{company}/_context.md` 존재 시 로드 (JD 요약·도메인·선별 카드 참조). 없으면 JD 직접 분석해 도메인 태그 추출.

1. **JD Mapping**: JD 필수/우대 스택·포지션 성격(tool-dev / architecture / live-service / unity / automation 등) → `_template/coverletter_basic.md`의 `domain` 태그·`intro` 태그와 매칭.

2. **Selection**: 아래 선택 기준 제시 후 사용자 승인 대기.
   - **intro**: JD 성격에 맞는 도입부 1개 선택 (선택 근거 명시).
   - **키워드별 사례**: 도메인 매칭 우선순위에 따라 1~2개 선택. 관련도 낮은 사례는 제거. 사례가 2개 모두 관련 있으면 관련도 높은 것을 먼저 배치.

3. **Write**: 선택된 블록만 남겨 **md 소스**를 작성.
   - 저장 경로: `_output/{company}/{company}_coverletter.md`
   - front-matter: `doc_type: coverletter` + `person` / `company` / 선택한 `intro`
   - 선택하지 않은 intro·사례 블록은 삭제. `domain` 주석 태그는 남겨도 무방(빌드 시 제거됨).
   - 구분선(`---`) 및 섹션 헤더(`## 1.`) 유지.
   - 마무리 수치(Python 독학, 작업 효율 40%, 데이터 손실 제로화) 중 JD 관련 성과를 문장 앞쪽으로 조정 가능.

4. **Build**: `python tools/build_doc.py _output/{company}/{company}_coverletter.md`
   → `{company}_coverletter.txt` (제출물. 채용 폼에 붙여넣는 plain text)
   front-matter 와 `<!-- ... -->` 주석은 렌더러가 제거한다.

5. **Validation**: 출력물 점검.
   - 사례 제거로 인한 맥락 단절(앞뒤 문장 연결 이상) 여부 확인.
   - 카드 데이터 외 허구 내용 추가 여부 확인.
   - `.claude/rules/writing-style.md` 명사체 종결 준수 확인.

## Rules

- **소스**: `.md` / **제출물**: `.txt` (생성물 — 직접 수정 금지). HTML 태그 금지.
- **템플릿 보호**: `_template/coverletter_basic.md` 수정 절대 금지.
- **허구 금지**: 템플릿 원본 데이터 외 임의 성과·사례 추가 금지.
- **승인 게이트**: Step 2 선택 결과를 사용자에게 제시하고 승인 후 빌드.
- **Naming**: `_output/{company}/{company}_coverletter.md` (소문자 영문).
