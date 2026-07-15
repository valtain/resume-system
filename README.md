# Resume Workspace — AI 협업 이력서 관리 시스템

> 경력 데이터를 AI와 협업해, 지원 상황에 따라 이력서를 재조합·생성하는 시스템

이력서·경력기술서를 AI와 협업하여 관리하는 워크스페이스.  
단순히 AI에게 문서를 생성시키는 방식이 아니라, AI가 일관된 규칙과 판단 기준을 갖고 작동하도록 **시스템으로 설계**한 구조다.  
"지시 → 생성"의 1단계 프롬프트 방식 대신, 모듈화·역할 분리·검증 단계를 갖춘 워크플로우를 구성했다.

**RAG 코퍼스 → 파생 지식(wiki) → 서브에이전트 오케스트레이션**의 AI-Native 레이어로 구성. 역할 분리는 규칙·프로시저 수준을 넘어, 분석·빌드·검증을 각각 전용 서브에이전트로 위임해 **에이전트 단위로 컨텍스트를 격리**하는 데까지 확장된다.

---

## 핵심 설계 원칙

- **Single Source of Truth** — `_corpus/project_modules.md`가 유일한 원본. 모든 회사별 출력물은 이 카드 라이브러리에서 파생된다. 원본 수정 없이 출력물만 고치는 행위는 구조적으로 금지.
- **규칙 명문화** — 작문 구조, 수치 표기 방식, 금지 표현 목록, Markdown 마크업 규칙을 `.claude/rules/`에 코드처럼 문서화하여 AI에 주입. "좋은 문장을 써줘"가 아닌 검증 가능한 기준으로 제어.
- **소스/생성물 분리** — 문서는 Markdown 으로 쓰고 `tools/build_doc.py` 가 HTML·PDF 를 만든다. CSS 는 `_template/assets/` 한 곳. 페이지 번호·프로젝트 번호·목차는 렌더러가 계산하므로 사람이 세지 않는다.
- **파생 지식 레이어(Wiki)** — `_corpus/` 카드에서 정체성 태그·강점 맵·경력 아크·카드 관계도를 집계한 `_wiki/`. `_corpus/` → `_wiki/` **단방향**이며 `/wiki-update`로만 재컴파일. 직접 수정 금지. Fit 판단·문서 서술 시 근거 레이어로 참조.
- **서브에이전트 역할 분리 + 컨텍스트 격리** — 분석·빌드·검증을 읽기전용/쓰기 스코프가 다른 전용 에이전트로 위임. 각 에이전트는 코퍼스 전체를 재로드하지 않고 **주입받은 선택 카드만** 사용(토큰 절약·환각 표면적 축소). 실제 수정 승인 게이트는 메인 스레드가 보유.
- **JD 매칭 프로세스** — 채용공고 분석 → 모듈 카드 선택 → 강조 순위 결정의 체계적 절차. 임의 선택이 아닌 JD 요구사항 대비 근거 설명을 포함.
- **다문서 일관성 검증** — 회사별로 resume·career·coverletter·portfolio 4종 문서를 생성한 후, 수치·서술·경력 정보의 사실 충돌을 자동으로 감지하고 수정 승인 프로세스를 거침.
- **비허구 원칙** — 모듈 카드에 없는 성과는 절대 작성하지 않도록 명시. AI hallucination을 구조적으로 차단.

---

## 디렉토리 구조

```
resume-workspace/
│
├── CLAUDE.md                        ← AI 워크스페이스 설정 (정체성, 워크플로우, 세션 규칙)
│
├── _corpus/                         ← (비공개) RAG Corpus — 프로젝트 카드 라이브러리 (Single Source of Truth)
│   ├── project_modules.md           ← 전체 경력 카드 목차 및 주요 내용
│   ├── module_a.md                  ← 프로젝트 상세 카드
│   ├── module_b.md                  ← 프로젝트 상세 카드
│   └── module_c.md                  ← 프로젝트 상세 카드
│
├── _wiki/                           ← (비공개) Wiki Layer — corpus에서 파생된 지식 집계 (/wiki-update로만 재생성)
│   ├── identity.md                  ← 정체성 태그 집계 + 근거 카드
│   ├── strengths.md                 ← cross-cutting 강점 테마 집계
│   ├── timeline.md                  ← 커리어 아크 서사
│   ├── relationships.md             ← 카드 간 개념 상속·기술 재사용·방법론 연속성
│   └── tech_map.md                  ← 기술 스택 집계 + 아키텍처 패턴 이력
│
├── _template/                       ← 템플릿 · 스타일 (직접 수정 금지)
│   ├── resume_basic.md              ← 이력서 템플릿 (Markdown 소스)
│   ├── career_basic.md              ← 경력기술서 템플릿
│   ├── portfolio_basic.md           ← 포트폴리오 템플릿
│   ├── coverletter_basic.md         ← 자기소개서 템플릿
│   └── assets/                      ← CSS 단일 원천 (base · career · resume · portfolio)
│
├── tools/
│   └── build_doc.py                 ← .md → .html(+--pdf) / .txt
│                                       번호·목차·타임라인 좌표 자동 계산
│
├── _output/                         ← (비공개) 회사별 문서
│   └── {company}/
│       ├── _context.md              ← JD 요약·카드 선택 근거·세션 노트
│       ├── {company}_career.md      ← 소스 (git 추적)
│       ├── {company}_career.html    ←   생성물 (gitignore)
│       ├── {company}_resume.md
│       ├── {company}_portfolio.md
│       └── {company}_coverletter.md ←   → .txt (채용 폼 붙여넣기용)
│
├── _workspace/                      ← (비공개) Active State — 현재 지원 현황 및 JD 분석 아카이브
│   ├── CURRENT_TARGET.md            ← 현재 지원 현황·집중 직군·다음 액션
│   └── past_jd_analyses/            ← 과거 JD 분석 아카이브 (패턴 재활용)
│
├── _portfolio/
│   └── TurnBasedCombat_Portfolio.cs ← 코드 포트폴리오 샘플
│
└── .claude/                         ← AI 행동 규칙 정의 (핵심)
    ├── agents/                      ← 서브에이전트 — 역할별 위임 (컨텍스트 격리)
    │   ├── jd-analyst.md            ← JD 요구 추출·Fit 판단·카드 매칭 (읽기전용)
    │   ├── corpus-curator.md        ← 소스 → card-spec 카드 정제 (Write)
    │   ├── doc-builder.md           ← 선별 카드 → 문서 소스 1종 조립·빌드 (Write)
    │   ├── interviewer-reviewer.md  ← 시니어 면접관 페르소나 리뷰 (읽기전용)
    │   └── consistency-auditor.md   ← 다문서 사실 충돌 감사 (읽기전용)
    ├── commands/                    ← 사용자 직접 호출 슬래시 커맨드
    │   ├── apply-workflow.md        ← JD → 지원 완료 단일 플로우
    │   ├── ingest.md                ← 소스 문서 → corpus 카드 정제 (+ wiki 자동 체인)
    │   ├── wiki-update.md           ← corpus → _wiki/ 재컴파일
    │   └── git-commit.md
    ├── skills/                      ← 내부 서브 프로시저
    │   ├── resume-build.md          ← resume · career 빌드
    │   ├── coverletter-build.md     ← 자기소개서 빌드
    │   ├── portfolio-build.md       ← 포트폴리오 빌드
    │   ├── resume-review.md         ← 면접관 페르소나 리뷰
    │   └── doc-consistency.md
    └── rules/                       ← 작문·출력 규칙
        ├── writing-style.md         ← 문체·마크업 규칙
        └── card-spec.md             ← corpus 카드 작성 규격
```

---

## .claude/ — AI 행동 규칙 정의

이 폴더가 이 워크스페이스의 핵심이다. AI가 매 세션마다 일관되게 동작하도록 역할·규칙·절차를 모두 문서화했다. 최근에는 커맨드·프로시저·규칙에 더해, 작업을 역할별 **서브에이전트**로 위임하는 오케스트레이션 계층을 추가했다.

### agents/ — 서브에이전트

분석·정제·빌드·검증을 각각 전용 에이전트로 분리한 오케스트레이션 계층. 메인 스레드가 워크플로우를 지휘하고, 각 단계를 스코프가 제한된 서브에이전트에 위임한다.

핵심 설계는 세 가지다.

- **컨텍스트 격리** — 각 에이전트는 코퍼스 전체를 재로드하지 않고 메인 스레드가 주입한 선택 카드만 사용. 중복 로드로 인한 토큰 낭비와 환각 표면적을 줄인다.
- **SSOT 준수** — 에이전트는 대응하는 스킬·규칙(예: `resume-build.md`, `card-spec.md`, `doc-consistency.md`)을 선행 학습하고 절차를 **재서술하지 않고 그대로** 따른다. 규칙은 한 곳에만 존재한다.
- **승인 게이트 중앙화** — 읽기전용 에이전트는 판단·보고만 반환하고, 실제 파일 수정은 메인 스레드가 사용자 승인 후 수행. 쓰기 권한을 가진 에이전트도 드래프트 제시 → 승인 후에만 Write.

| 에이전트 | 역할 | 스코프 | 모델 | 위임 지점 |
| --- | --- | --- | --- | --- |
| `jd-analyst` | JD 요구 추출·Fit 판단·카드 매칭 → Fit Report 반환 | 읽기전용 | sonnet | apply-workflow Phase 1·3 |
| `corpus-curator` | 소스 문서 → card-spec 규격 카드로 정제 (드래프트 → 승인 후 Write) | Write (`_corpus/`까지) | opus | `/ingest` |
| `doc-builder` | 선별 카드 → 문서 소스 1종 조립 후 빌드 | Write (`_output/`) | sonnet | Phase 4 (문서 2종+ 병렬) |
| `interviewer-reviewer` | 시니어 면접관 페르소나로 규격 위반·과장 탐지, 심각도 분류 | 읽기전용 | opus | Phase 5 |
| `consistency-auditor` | 다문서 간 수치·사건·재직기간 사실 충돌 감사 | 읽기전용 | sonnet | Phase 5 (문서 2종+) |

`apply-workflow`가 Phase별로 이 에이전트들을 호출한다. Phase 4에서 문서를 2종 이상 만들 때는 문서당 하나씩 `doc-builder`를 병렬로 띄우고, Phase 5의 리뷰·감사도 병렬로 동시 실행한다. 코퍼스 로드는 Phase 3에서 한 번만 일어나고, 이후 에이전트에는 선택된 카드 내용만 전달된다.

---

### commands/ — 사용자 직접 호출 슬래시 커맨드

**`apply-workflow.md`** (`/apply-workflow <slug>`)  
JD 공유 시점부터 지원 완료까지 7단계(Phase 0~6) 단일 플로우로 진행. 중단 시 재개 가능. Phase 1·3·4·5는 전용 서브에이전트에 위임한다.

```
Phase 0: Session Load + 재개 판단  — CURRENT_TARGET.md + _context.md Workflow State 확인
Phase 1: Fit Check               — jd-analyst 위임 → 적합도 리포트 → 지원 여부 Gate
Phase 2: 문서 범위 확인            — 필요 문서 목록 제시, 기존 파일 재활용 분류
Phase 3: Context 초기화 + 카드 선별 — _corpus/ 1회 로드, _context.md 생성
Phase 4: 문서 빌드                 — 2종 이상 시 doc-builder 병렬 위임
Phase 5: 품질 검증                 — interviewer-reviewer + consistency-auditor 조건부 병렬
Phase 6: 마무리                   — CURRENT_TARGET.md 업데이트, JD 아카이브
```

slug는 `_output/{slug}/` 폴더명이자 파일 prefix. 같은 회사에 복수 지원 시 → `nexon_client`, `nexon_server`처럼 구분.

---

**`ingest.md`** (`/ingest [카드명]`)  
신규 소스 문서(설계서·CONTEXT.md·README·변경 내역 등)를 corpus 카드로 정제하고, wiki까지 자동 갱신하는 통합 파이프라인. 카드 작성은 `corpus-curator` 에이전트에 위임.

```
1. 소스 문서 수신
2. card-spec.md 규격으로 카드 드래프트 작성 → 사용자 확인
3. _corpus/ 카드 반영 (승인 후 Write)
4. /wiki-update 자동 체인 — 카드 변경 → wiki 재컴파일은 원자적 단위
5. 완료 보고 (신규/업데이트, 변경 wiki 파일)
```

카드 형식은 `.claude/rules/card-spec.md`가 규정: 요약 → 기술 스택 → 핵심 설계 → 트러블슈팅 → 성과 → 동기화 트리거. "무엇을 만들었다"가 아닌 "왜 그 판단을 했는가"를 중심으로 서술하도록 강제.

---

**`wiki-update.md`** (`/wiki-update`)  
`_corpus/` 전체를 기반으로 `_wiki/` 5개 파일을 재컴파일한다. **`_wiki/` 쓰기 권한은 이 커맨드에만 있다.**

```
방향: _corpus/ → _wiki/ 단방향 (역방향 절대 금지)
대상: identity / strengths / timeline / relationships / tech_map (전체 재컴파일)

- 허구 금지: 카드 데이터 외 내용 추가 금지 (해석은 가능, 사실 추가 불가)
- 근거 명시: 각 항목에 "근거 카드: [카드명]" 표기
- 사용자 확인 필수: 변경 내역 제시 후 Write
```

---

**`git-commit.md`** (`/git-commit`)  
`git diff` 결과를 분석하여 변경 파일을 논리적 그룹으로 분류하고 구조화된 커밋 메시지를 제안한다.

```
분류 기준:
- rename/remove  — 항상 단독 커밋
- feat           — 신규 기능 (기능 단위 그룹핑)
- refactor       — 구조 변경
- fix            — 버그 수정
- docs/chore     — 문서·설정 변경

메시지 형식: [category] 한국어 명사체 요약
```

---

### skills/ — 내부 서브 프로시저

`apply-workflow`에서 조건부로 호출되는 품질 검증·빌드 프로시저. 직접 호출도 가능.

**`resume-build.md`**  
JD 기반 이력서 생성의 5단계 절차를 정의한다.

```
1. JD 분석    — 필수 기술 스택, 우대 조건, 포지션 맥락 추출
2. 모듈 선택  — 카드 라이브러리에서 JD와 매칭되는 카드 선정 + 근거 설명
3. 확인 요청  — 사용자에게 선택 결과 제시, 승인 후 진행
4. MD 작성    — _template/ 템플릿 기반, writing-style.md 규칙 적용
5. 빌드       — python tools/build_doc.py _output/{company}/{company}_{type}.md
6. 검토 요청  — 카드 선택의 정확성, 사실 여부, 누락 맥락 확인 요청
```

이 스킬이 담당하는 문서 타입은 두 종류: `career`(경력기술서, 제한 없음)와 `resume`(이력서, A4 1-2장 압축). 자기소개서·포트폴리오는 아래 별도 스킬이 담당하여, 전체로는 4종 문서를 생성한다.

---

**`coverletter-build.md`**  
자기소개서(`.txt` 출력) 빌드 절차. 채용 폼에 붙여넣기 좋도록 `tools/build_doc.py`가 `.md` → `.txt`로 변환한다.

**`portfolio-build.md`**  
포트폴리오(A4 landscape `.html` 출력) 빌드 절차. 슬라이드 번호·목차·타임라인 좌표는 렌더러가 계산하므로 소스에 쓰지 않는다.

---

**`resume-review.md`**  
"이력서를 많이 본 경력직 면접관" 페르소나로 이력서를 리뷰한다.

```
지적 대상:
- 읽기가 멈추는 문장 (설명 없는 코드명·약어)
- 수치 없는 강조어 ("획기적으로", "대폭")
- 과밀한 정보 블록 / 부정적 어감 / 코드 수준 세부사항

심각도 분류: High (읽기 중단) / Medium (어색함) / Low (개선 권장)
```

---

**`doc-consistency.md`**  
회사별 문서 세트(resume / career / coverletter / portfolio) 간 사실 충돌을 검증한다.

```
1. _output/{company}/ 내 모든 문서 소스(.md / .txt) 로드
2. 비교: 수치, 트러블슈팅 서술, 기술 선택 이유, 재직 기간/직책
3. Fact conflict (수치 불일치) → 반드시 수정
   Expression variance (표현 차이) → 선택적 수정
4. 수정 목록 제시 후 승인 시 실행
```

---

### rules/ — 작문·출력 규칙

**`writing-style.md`**  
모든 문서에 적용되는 작문 기준. AI가 스스로 판단하지 않고 이 규칙을 따르도록 설계되었다.

| 항목 | 규칙 |
|------|------|
| 서술 구조 | 문제 → 판단 → 결과 (기능 나열 금지) |
| 수치 표기 | before/after 형식 필수: `9시간 → 1시간 (9배 개선)` |
| 문체 | 명사체 종결 (`원인 특정 및 수정`) / 합니다체 금지 |
| 기간 단독 나열 | 금지 — 반드시 맥락과 결과 동반 |
| 기술 스택 | 선택 이유 서술 필수: `Addressables — 번들 단위 로딩으로 초기 메모리 점유 감소` |
| 트러블슈팅 | 현상 + **원인 특정 과정** + 결과 3요소 필수 |
| 금지 표현 | `완벽한`, `최고의`, `획기적인`, `담당했습니다` 등 |

마크업·페이징 규칙(트러블슈팅 뱃지 `- [TS]`, 페이지 경계 `##`, 번호·목차는 렌더러 계산)도 이 파일이 규정한다. 소스 md 에 HTML/CSS 직접 작성 금지.

---

**`card-spec.md`**  
`_corpus/` 카드 작성 규격. `/ingest`·`corpus-curator`가 이 규격을 단일 원천으로 따른다.

```
카드 6-섹션 (고정):
1. 요약        — 배경 → 판단 → 방향 (3-5문장, 명사체)
2. 기술 스택   — 선택 이유가 명확한 핵심 스택만
3. 핵심 설계   — what / why / how 구조의 설계 결정
4. 트러블슈팅  — 원인 특정 과정 필수 (선택 섹션)
5. 성과        — Before/After 수치 또는 구조적 달성
6. 동기화 트리거 — 변경 감지 조건 명시
```

---

## 워크플로우

> `/apply-workflow <slug>` — 아래 전체 플로우를 하나의 명령으로 실행하는 엔트리포인트

```
채용공고 입수
    │
    ▼
Phase 1  Fit Check                    → jd-analyst (읽기전용)
    JD 분석 + 지원자 Fit 리포트 → 지원 여부 Gate
    │
    ▼
Phase 2  문서 범위 확인
    resume / career / coverletter / portfolio 중 선택
    │
    ▼
Phase 3  Context 초기화 + 카드 선별     → jd-analyst
    _corpus/ 1회 로드 + 모듈 카드 매칭 (근거 포함) → 사용자 승인
    │
    ▼
Phase 4  문서 빌드                      → doc-builder (문서당 1, 병렬)
    _output/{slug}/{slug}_{doc_type}.md   (소스, 주입 카드만 사용)
    → tools/build_doc.py → .html / .txt   (생성물)
    writing-style.md 규칙 적용
    │
    ▼
Phase 5  품질 검증 (병렬)
    interviewer-reviewer  면접관 시점 리뷰 → High/Medium/Low 분류
    consistency-auditor   다문서 사실 충돌 감사 → Fact conflict
    │  (수정은 메인 스레드가 승인 후 수행)
    ▼
Phase 6  마무리
    CURRENT_TARGET.md 업데이트 · JD 아카이브
    │
    ▼
/git-commit
    변경 파일 논리 그룹핑 → 구조화 커밋
```

> 별도 파이프라인: `/ingest` (소스 문서 → corpus 카드 정제 → `/wiki-update` 자동 체인). 카드가 바뀌면 `_wiki/` 파생 지식이 재컴파일된다.

---

## 이 구조가 보여주는 것

**AI를 도구가 아닌 협업자로 설계**  
커맨드(commands), 프로시저(skills), 규칙(rules)를 모두 명문화하여 AI가 매 세션마다 일관되게 동작하도록 구성. "잘 물어보는" 수준이 아닌, 재현 가능한 시스템을 구축.

**Prompt Engineering이 아닌 System Design**  
일회성 프롬프트가 아닌 재사용 가능한 skill, 검증 가능한 rule, 논리적으로 분리된 command로 구조화. 워크플로우가 바뀌어도 각 컴포넌트는 독립적으로 수정 가능.

**멀티에이전트 오케스트레이션**  
단일 프롬프트에 모든 일을 맡기는 대신, 분석·정제·빌드·검증을 역할별 서브에이전트로 분리. 각 에이전트가 컨텍스트를 격리(코퍼스 전체 대신 주입 카드만 로드)해 토큰 효율을 높이고 환각 표면적을 줄임. 수정 승인 게이트는 메인 스레드에 중앙화하여 자동화와 통제를 양립.

**소프트웨어 엔지니어링 원칙 적용**  
- SRP (단일 책임): agents · skills · commands · rules 각자 하나의 역할만 담당  
- DRY (중복 제거): 프로젝트 내용은 `_corpus/`에만 존재, 출력물·`_wiki/`는 파생  
- 검증 가능성: 일관성 감사(consistency-auditor), 규칙 위반 검출(interviewer-reviewer)을 단계로 내장  

---

*Powered by [Claude Code](https://claude.ai/code) — Anthropic*
