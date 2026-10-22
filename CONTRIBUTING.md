# 협업 가이드

상세 규칙은 [조직 README](https://github.com/YC-CapstoneDesign-5)의 10~11장을 기준으로 합니다.

## 작업 흐름
1. 이슈 생성 (기능 요청 / 버그 리포트 / 작업 템플릿 사용)
2. `develop`에서 브랜치 분기: `<prefix>/<작업명>` (소문자, 하이픈)
   - prefix: `feat` `fix` `docs` `refactor` `test` `chore`
3. Conventional Commits 형식으로 커밋 (`<type>: 요약` + `## 변경 이유` / `## 변경 내용`)
4. `develop` 대상으로 PR 생성 — 제목 `[<Type>] 요약`, 본문은 템플릿이 자동으로 채워집니다.
5. 리뷰어 1인 이상 승인 + CI 통과 후 **Squash and Merge**, 머지된 브랜치는 삭제

## 커밋 예시
```
feat: 4대보험 고지 자료 다차원 귀속 매칭 로직 구현

## 변경 이유
- 주민번호 단독 매칭 시 발생하는 동명이인 문제 방지

## 변경 내용
- InsuranceMatcher 서비스 신설

Resolves: #42
```

## 주의
- 개인정보(주민등록번호 등)는 커밋, 이슈, PR, 스크린샷에 절대 원본으로 올리지 않습니다.
