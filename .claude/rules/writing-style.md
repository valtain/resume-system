# Writing Style Rules

## 1. Logic & Narrative
- **Structure**: `Problem(필요성)` → `Judgment(근거)` → `Outcome(결과)`
- **Tech Choice**: 기술명 나열 지양. 선택의 당위성(Why) 병기.
- **Troubleshooting**: `현상` - `원인 특정(데이터/로그)` - `해결` 순서 엄수.

## 2. Diction & Tone
- **Terminals**: 명사형 종결 처리 (예: 설계 및 구현 / 원인 특정).
- **Prohibited**:
    - 과장어 (최고, 혁신적, 무조건, 완벽한)
    - 성과 없는 기간 나열 (2주 완성, 1인 개발 등)
    - 합니다/했다체 등 동사형 종결
- **Metrics**: 정량적 지표는 반드시 `Before → After` 형식으로 표기.

## 3. Markup & Output

출력물 소스는 **Markdown**. HTML/CSS 는 `tools/build_doc.py` 가 생성하므로 직접 쓰지 않는다.

- **Troubleshooting**: 목록 항목을 `- [TS] ` 로 시작. → Troubleshooting 뱃지
- **보충 설명**: 해당 항목 하위에 `> ` blockquote. → `.note`
- **Focus**: `**굵게**` 는 핵심 성과 수치에만 최소한으로 적용.
- **성과 라벨**: 단독 줄의 `**라벨**` (예: `**구축**`, `**트러블슈팅**`).
- **마무리 강조**: 목록 밖의 `> ` 한 덩이 (portfolio 콜아웃 박스).
- **Paging**: `## ` 이 페이지/슬라이드 경계 (career·portfolio 기준).
  **페이지 번호·프로젝트 번호·슬라이드 번호·목차·타임라인 좌표는 렌더러가 계산한다.
  문서에 쓰지 않는다.**
- **금지**: 소스 md 에 `<span>`·`<div>`·`style=` 등 HTML 직접 작성.
  표현이 필요하면 CSS(`_template/assets/`)를 고칠 일이지 문서를 고칠 일이 아니다.