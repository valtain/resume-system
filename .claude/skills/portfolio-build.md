# Skill: portfolio-build

## 목적

슬러그(`{slug}`)별 포트폴리오 **Markdown 소스** 작성 → 빌드.

`{slug}` 예시: `gearsecond`, `gearsecond_client`, `kakao_unity` 등. 동일 회사라도 직군·포지션이 다르면 별도 슬러그 사용.

## 문서 형식

- **소스**: `_output/{slug}/{slug}_portfolio.md`
- **생성물**: `.html` (A4 landscape) — `tools/build_doc.py` 가 만든다. **직접 작성 금지.**
- CSS 는 `_template/assets/portfolio.css` 한 곳. 출력물에 CSS 를 복사하지 않는다.
- **슬라이드 번호·페이지 번호·목차는 렌더러가 계산한다. md 에 쓰지 않는다.**
- 타임라인 간트의 블록 위치·명암도 연도에서 계산된다. 퍼센트를 손으로 찍지 않는다.
- md 규약: `_template/portfolio_basic.md` + `tools/build_doc.py` 상단 docstring.

## 전제 조건

- `_output/{slug}/{slug}_career.md` 존재 (없으면 `resume-build` 먼저 실행) — 슬라이드 내용 원천

## 실행 절차

### Step 1: 선행 학습

1. `_output/{slug}/{slug}_career.md` — 슬라이드 내용 원천
2. `_template/portfolio_basic.md` — 슬라이드 타입·meta 키 규약
3. `_corpus/project_modules.md` — 보충 데이터 필요 시 해당 카드만

### Step 2: 슬라이드 구성 결정

JD 에 맞춰 어떤 프로젝트를 어떤 타입으로 낼지 정한다.

| type | 용도 |
|------|------|
| `philosophy` | 업무 태도 3원칙 |
| `detail` | 성과·트러블슈팅 중심 (기본) |
| `visual` | 다이어그램·큰 지표로 보여줄 프로젝트 |
| `timeline` | 경력 간트 + 시기별 요약 |
| `contact` | 마지막 연락처 |

강조하고 싶은 프로젝트 1~2개를 `visual` 로, 나머지를 `detail` 로 두는 구성이 기본.

### Step 3: md 작성

`_output/{slug}/{slug}_portfolio.md` 작성.

- `## 제목` 하나가 한 슬라이드이자 목차 항목
- 첫 문단 = 슬라이드 설명(`detail`) 또는 부제(`visual`)
- 단독 줄 `**라벨**` = 성과 목록 라벨
- `- [TS] ` = 트러블슈팅 뱃지, 하위 `> ` = 보충 설명
- 목록 밖 `> ` 한 덩이 = 마무리 강조 박스
- 미디어 링크(영상·저장소)는 카드에 `## 미디어 링크` 가 있을 때만, 관련 슬라이드에만

### Step 4: 빌드

```
python tools/build_doc.py _output/{slug}/{slug}_portfolio.md
```

실패하면 md 규약 위반이므로 md 를 고친다. 생성된 html 을 손대지 않는다.

### Step 5: 결과 확인

브라우저에서 열어 레이아웃·타임라인·링크 확인 요청.
