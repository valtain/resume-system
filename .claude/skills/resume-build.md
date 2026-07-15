# Skill: Resume Build (JD 맞춤형 빌드)

## Objective

- JD 분석 결과에 따라 모듈 카드를 선별하고 회사별 맞춤 **Markdown 소스** 작성 → 빌드.

## 문서 형식

- **소스**: `.md` (사람이 쓰고 읽고 리뷰하는 대상)
- **생성물**: `.html` / `.pdf` — `tools/build_doc.py` 가 만든다. **직접 작성 금지.**
- CSS 는 `_template/assets/` 한 곳에만 있다. 출력물에 CSS 를 복사하지 않는다.
- 페이지 번호·프로젝트 번호·목차는 렌더러가 계산한다. md 에 쓰지 않는다.
- md 규약: `.claude/rules/writing-style.md` §3 + `tools/build_doc.py` 상단 docstring.

## Procedure

0. **Context Load**: `_output/{company}/_context.md` 존재 시 로드해 JD 요약·카드 선택 근거·세션 노트 파악. 없으면 Step 1 후 생성.
1. **JD Analysis**: 필수/우대 스택 및 포지션 핵심 맥락(툴/라이브/시스템 등) 추출.
2. **Selection**: `_corpus/project_modules.md`에서 JD와 매칭되는 카드 선별 및 사유 제시.
3. **Strategy**: 매칭 우선순위 및 트러블슈팅 성과 중심의 강조 순서 결정.
4. **Write**: `_template/{doc_type}_basic.md` 를 참고해 `_output/{company}/{company}_{doc_type}.md` 작성.
   `_context.md` 없으면 이 단계에서 생성.
5. **Build**: `python tools/build_doc.py _output/{company}/{company}_{doc_type}.md`
   실패하면 md 규약 위반이므로 md 를 고친다. 생성된 html 을 손대지 않는다.
6. **Validation**: `.claude/rules/writing-style.md` 준수 여부 최종 확인.
7. **Archive**: JD 분석 결과를 `_workspace/past_jd_analyses/{company}_{YYYY-MM-DD}.md`에 저장.

## Rules

- **Naming**: `_output/{company}/{company}_{doc_type}.md` (소문자 영문).
  빌드 시 같은 이름의 `.html` 이 나온다.
- **Integrity**: 카드 데이터 외 임의 성과 추가 금지.
- **Confirmation**: 카드 선별 결과는 작성 전 반드시 사용자 승인 필요.
- **Media Links**: 선별된 카드에 `## 미디어 링크` 섹션이 있으면 해당 링크를 출력물에 포함. 링크는 카드 내용과 직접 연관된 섹션에만 배치 (내용과 무관한 페이지에 배치 금지).
