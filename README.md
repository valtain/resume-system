# Resume Workspace — AI 협업 이력서 관리 시스템

> 경력 데이터를 AI와 협업해, 지원 상황에 따라 이력서를 재조합·생성하는 시스템

이력서·경력기술서를 AI와 협업하여 관리하는 워크스페이스.  
단순히 AI에게 문서를 생성시키는 방식이 아니라, AI가 일관된 규칙과 판단 기준을 갖고 작동하도록 **시스템으로 설계**한 구조다.  
"지시 → 생성"의 1단계 프롬프트 방식 대신, 모듈화·역할 분리·검증 단계를 갖춘 워크플로우를 구성했다.

---

## 핵심 설계 원칙

- **Single Source of Truth** — `_modules/project_modules.md`가 유일한 원본. 모든 회사별 출력물은 이 카드 라이브러리에서 파생된다. 원본 수정 없이 출력물만 고치는 행위는 구조적으로 금지.
- **규칙 명문화** — 작문 구조, 수치 표기 방식, 금지 표현 목록, HTML 마크업 규칙을 `.claude/rules/`에 코드처럼 문서화하여 AI에 주입. "좋은 문장을 써줘"가 아닌 검증 가능한 기준으로 제어.
- **JD 매칭 프로세스** — 채용공고 분석 → 모듈 카드 선택 → 강조 순위 결정의 체계적 절차. 임의 선택이 아닌 JD 요구사항 대비 근거 설명을 포함.
- **다문서 일관성 검증** — 회사별로 career / resume / coverletter 세 문서를 생성한 후, 수치·서술·경력 정보의 사실 충돌을 자동으로 감지하고 수정 승인 프로세스를 거침.
- **비허구 원칙** — 모듈 카드에 없는 성과는 절대 작성하지 않도록 명시. AI hallucination을 구조적으로 차단.

---

## 디렉토리 구조

```
resume-workspace/
│
├── CLAUDE.md                        ← AI 워크스페이스 설정 (정체성, 워크플로우, 세션 규칙)
│
├── _modules/                        ← (비공개) 프로젝트 카드 라이브러리 (Single Source of Truth)
│   ├── project_modules.md           ← 전체 경력 카드 목차 및 주요 내용
│   ├── module_a.md                  ← 프로젝트 상세 카드
│   ├── module_b.md                  ← 프로젝트 상세 카드
│   └── module_c.md                  ← 프로젝트 상세 카드
│
├── _base/                           ← HTML 템플릿 (직접 수정 금지)
│   ├── career_basic.html            ← 경력기술서 기본 템플릿
│   └── resume_basic.html            ← 이력서 기본 템플릿
│
├── _output/                        ← (비공개) 회사별 생성 문서
│   └── {company}/
│       ├── {company}_career.html
│       ├── {company}_resume.html
│       └── {company}_coverletter.html
│
├── _portfolio/
│   └── TurnBasedCombat_Portfolio.cs ← 코드 포트폴리오 샘플
│
├── _corpus/                         ← JD 분석 아카이브 (향후 사용)
│
├── _workspace/                      ← 지원 상태 추적
│   ├── CURRENT_TARGET.md            ← 현재 활성 지원 건 + Workflow 상태
│   └── past_jd_analyses/            ← 완료 JD 아카이브
│
└── .claude/                         ← AI 행동 규칙 정의 (핵심)
    ├── skills/                      ← 재사용 가능한 프로시저
    │   ├── resume-review.md
    │   ├── resume-build.md
    │   └── doc-consistency.md
    ├── commands/                    ← 커스텀 명령어
    │   ├── apply-workflow.md
    │   ├── module-card.md
    │   └── git-commit.md
    └── rules/                       ← 작문·출력 규칙
        └── writing-style.md
```

---

## .claude/ — AI 행동 규칙 정의

이 폴더가 이 워크스페이스의 핵심이다. AI가 매 세션마다 일관되게 동작하도록 역할·규칙·절차를 모두 문서화했다.

### skills/ — 재사용 가능한 프로시저

**`resume-build.md`**  
JD 기반 이력서 생성의 5단계 절차를 정의한다.

```
1. JD 분석    — 필수 기술 스택, 우대 조건, 포지션 맥락 추출
2. 모듈 선택  — 카드 라이브러리에서 JD와 매칭되는 카드 선정 + 근거 설명
3. 확인 요청  — 사용자에게 선택 결과 제시, 승인 후 진행
4. HTML 작성  — _base/ 템플릿 기반, writing-style.md 규칙 적용
5. 검토 요청  — 카드 선택의 정확성, 사실 여부, 누락 맥락 확인 요청
```

문서 타입은 두 종류: `career`(경력기술서, 제한 없음)와 `resume`(이력서, A4 1-2장 압축).

---

**`resume-review.md`**  
"이력서를 많이 본 경력직 면접관" 페르소나로 이력서를 리뷰한다. `/apply-workflow` Phase 5에서 조건부 호출.

```
지적 대상:
- 읽기가 멈추는 문장 (설명 없는 코드명·약어)
- 수치 없는 강조어 ("획기적으로", "대폭")
- 과밀한 정보 블록
- 부정적 어감의 표현
- 코드 수준 세부사항 (이력서에 부적합)

심각도 분류: High (읽기 중단) / Medium (어색함) / Low (개선 권장)
```

---

**`doc-consistency.md`**  
회사별 문서 세트(career / resume / coverletter) 간 사실 충돌을 검증한다.

```
1. _output/{company}/ 내 모든 HTML 파일 로드
2. 비교 항목 추출: 수치, 트러블슈팅 서술, 기술 선택 이유, 재직 기간/직책
3. 충돌 분류:
   - Fact conflict   — 같은 사건인데 수치가 다름 → 반드시 수정
   - Expression variance — 같은 내용인데 표현이 다름 → 선택적 수정
4. 수정 목록 제시 후 승인 시 실행
```

---

### commands/ — 커스텀 명령어

**`apply-workflow.md`** (`/apply-workflow <slug>`)  
JD 공유부터 지원 완료까지의 단일 진입점. 중단 시 재개 가능.

```
Phase 0. Session Load + 재개 판단  — _workspace/CURRENT_TARGET.md 로드, 중단 기록 확인
Phase 1. Fit Check                 — JD 분석 + 적합도 표 제시 → Gate 1 (지원 여부)
Phase 2. 문서 범위 확인            — 필요 문서 목록 확인 → Gate 2
Phase 3. 카드 선별                 — 모듈 카드 매칭 + _context.md 생성 → Gate 3
Phase 4. 문서 빌드                 — 단일: resume-build 직접 실행 / 복수: Subagent 병렬
Phase 5. 품질 검증                 — resume-review / doc-consistency 조건부 실행
Phase 6. 마무리                    — 상태 업데이트, JD 아카이브 저장
```

---

**`module-card.md`** (`/module-card`)  
`_modules/project_modules.md`에 프로젝트 카드를 정석 구조로 추가·수정한다.

```
## N. 프로젝트명
**요약**       — 배경 → 판단 → 방향 (3-5문장)
**기술 스택**  — 선택 이유 포함 (단순 나열 금지)
**핵심 설계**  — what / why / how 구조로 각 결정 서술
**트러블슈팅** — 현상 → 원인 특정 과정 → 해결 → 결과 (선택)
**성과**       — before/after 수치 또는 구조적 변화
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

---

## 워크플로우

```
채용공고 입수
    │
    ▼
/apply-workflow <slug>
    Phase 0. Session Load + 재개 판단
             _workspace/CURRENT_TARGET.md 확인 → 중단 건 재개 여부 결정
    │
    ▼
    Phase 1. Fit Check
             JD 분석 + 적합도 표 제시 → Gate 1: 지원 여부 결정
    │
    ▼
    Phase 2. 문서 범위 확인
             필요 문서 목록 (이력서·경력기술서·자소서) → Gate 2: 범위 승인
    │
    ▼
    Phase 3. 카드 선별
             모듈 카드 매칭 + 근거 제시 + _context.md 생성 → Gate 3: 카드 승인
    │
    ▼
    Phase 4. 문서 빌드
             단일 문서: resume-build 직접 실행
             복수 문서: Subagent 병렬 실행
             → _output/{slug}/{slug}_{doc_type}.html
    │
    ▼
    Phase 5. 품질 검증 (조건부)
             이력서 포함 시 → resume-review (면접관 시점 High/Medium/Low)
             문서 2종 이상 → doc-consistency (다문서 사실 충돌 검증)
    │
    ▼
    Phase 6. 마무리
             _workspace/CURRENT_TARGET.md 상태 업데이트
             _workspace/past_jd_analyses/{slug}_{date}.md 아카이브
    │
    ▼
/git-commit
    변경 파일 논리 그룹핑 → 구조화 커밋
```

---

## 이 구조가 보여주는 것

**AI를 도구가 아닌 협업자로 설계**  
역할(skills), 규칙(rules), 절차(commands)를 모두 명문화하여 AI가 매 세션마다 일관되게 동작하도록 구성. "잘 물어보는" 수준이 아닌, 재현 가능한 시스템을 구축.

**Prompt Engineering이 아닌 System Design**  
일회성 프롬프트가 아닌 재사용 가능한 skill, 검증 가능한 rule, 논리적으로 분리된 command로 구조화. 워크플로우가 바뀌어도 각 컴포넌트는 독립적으로 수정 가능.

**소프트웨어 엔지니어링 원칙 적용**  
- SRP (단일 책임): skills · commands · rules 각자 하나의 역할만 담당  
- DRY (중복 제거): 프로젝트 내용은 `_modules/`에만 존재, 출력물은 파생  
- 검증 가능성: 일관성 검증(doc-consistency), 규칙 위반 검출(resume-review)을 단계로 내장  

---

*Powered by [Claude Code](https://claude.ai/code) — Anthropic*
