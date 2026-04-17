Run `git status` and `git diff --stat HEAD`. If no changes at all, stop.

## 그룹핑 분석

변경 파일 전체를 아래 기준으로 논리적 커밋 그룹으로 분류한다:

- **rename/remove**: 파일 이름 변경·삭제는 항상 단독 그룹
- **feat**: 신규 파일 추가 (기능 단위로 묶음)
- **refactor**: 기존 파일 구조 변경 (함께 변경된 연관 파일 포함)
- **fix**: 버그·오류 수정
- **docs/chore**: 문서, 설정 변경

같은 목적을 위한 변경(예: 내용 분리 + 분리를 참조하는 파일 수정)은 같은 그룹으로 묶는다.
성격이 다른 변경은 반드시 다른 그룹으로 분리한다.

## 제시 및 확인

그룹 목록을 아래 형식으로 제시하고 사용자 확인을 받는다:

```
[커밋 1] [type] 메시지
  - 파일 A
  - 파일 B

[커밋 2] [type] 메시지
  - 파일 C
```

## 순차 커밋

확인 후 그룹 순서대로 커밋. 파일은 명시적으로 지정 (`git add -A` 금지):

```
git add <파일 목록>
git commit -m "[category] 메시지

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

커밋 메시지: 한국어, 명사체, 한 줄.
