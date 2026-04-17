# Writing Style Rules

> Apply these rules to all resume and career document output.

---

## 1. Narrative Structure

Every project entry follows this flow — no exceptions:

```
문제 (Problem) → 판단 (Judgment) → 결과 (Outcome)
```

- **문제**: 왜 이 작업이 필요했는가. 기존 상태의 무엇이 문제였는가.
- **판단**: 왜 이 방법을 선택했는가. 다른 선택지 대비 이유.
- **결과**: 구조적으로 무엇이 달라졌는가. 수치가 있으면 before/after.

기능 나열로 끝나는 서술은 작성 실패로 간주한다.

---

## 2. Figures & Metrics

- 수치가 있으면 반드시 before/after 형식으로 표기
  - 좋은 예: `9시간 → 1시간 (약 9배 개선)`
  - 나쁜 예: `빌드 시간 단축`, `성능 개선`
- 개발 기간은 성과가 아님
  - 나쁜 예: `2주 만에 완성`, `1인 개발 5일`
  - 기간을 쓰려면 반드시 맥락과 결과가 함께 있어야 함

---

## 3. Tone & Form

**문체**
- 명사체 사용 — 동사형 종결 금지
  - 좋은 예: `원인 특정 및 수정`, `구조 설계 및 적용`
  - 나쁜 예: `원인을 찾아 수정했습니다`, `구조를 설계했습니다`

**금지 표현**
- `완벽한`, `최고의`, `무조건`, `획기적인` 등 과장어
- `개발했습니다`, `구현했습니다`, `담당했습니다` 등 합니다체
- 기간 단독 나열 (`2주`, `5일`, `2인 8개월` 등)

---

## 4. Troubleshooting Entries

트러블슈팅 항목은 반드시 아래 세 요소를 포함:

1. **현상**: 어떤 문제가 발생했는가
2. **원인 특정 과정**: 어떻게 원인을 찾았는가 (측정, 로그, 재현 등)
3. **해결**: 무엇을 바꿨고 결과가 어떻게 달라졌는가

차별화 맥락이 있으면 명시:
- `팀 전체 미발견 상태에서 단독 원인 특정`
- `기존 접근 방식과 달리 데이터 기반으로 원인 격리`

---

## 5. Technology Choices

기술 스택을 언급할 때는 선택 이유를 함께 서술:
- 좋은 예: `Addressables 도입 — 번들 단위 로딩으로 초기 메모리 점유 감소`
- 나쁜 예: `Addressables, UniTask, DOTween 사용`

기술 용어는 한글/영문 혼용 그대로 유지 (Addressables, Perforce, PyQt5 등)

---

## 6. HTML Output Rules

**태그 사용**
- 트러블슈팅 항목: `<span class="tag-ts">Troubleshooting</span>`
- 세부 보충 설명: `<span class="note">내용</span>` (들여쓰기 효과)
- Bold(`<strong>`) 남발 금지 — 진짜 강조할 것만

**디자인 기준 (`_base/career_basic.html` 스타일 유지)**
- 폰트: Noto Serif KR (제목) + Noto Sans KR (본문) + DM Mono (메타/코드)
- 레이아웃: 흰색 배경, 좌우 분할 (메타 컬럼 200px + 콘텐츠 컬럼)
- `_base/` 파일을 직접 수정하지 않음 — `_output/`에 복사 후 작업

**PDF 출력**
Chrome → 파일 열기 → `Ctrl+P` → 프린터: PDF로 저장 → 여백: 없음 → 저장

**내용 넘침 대응**
각 `.page`는 A4 기준으로 새 페이지에서 시작. 내용이 넘치면 자동으로 다음 페이지로 이어짐
(`@media print`에 `page-break-before: always`, `height: auto` 적용 완료).

우선순위 순서로 대응:
1. 항목 자체를 줄이는 것이 근본 해결 — 덜 중요한 항목 제거 후 HTML 반영
2. 특정 항목이 페이지 중간에서 잘리는 경우 해당 `<li>`에 추가:
   ```html
   <li style="page-break-inside: avoid">...</li>
   ```