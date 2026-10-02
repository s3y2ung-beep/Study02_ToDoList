# Study02_ToDoList

매일 10~20개의 할 일을 관리하는 개인용 할 일 관리 앱입니다.
설치나 빌드 없이 브라우저에서 바로 실행되는 순수 HTML/CSS/JavaScript 앱입니다.

> **현재 상태:** 구현 완료 — [PRD](PRD.md) 기준 v1

## 주요 기능

- 할 일 추가, 수정(인라인 편집), 삭제
- 완료 체크 — 완료한 항목은 목록 아래로 이동
- 카테고리 분류: 업무 / 개인 / 공부
- 카테고리 필터 탭
- 진행률 보기: 전체 + 카테고리별
- 완료 항목 일괄 정리
- 새로고침해도 유지되는 데이터 (`localStorage`)

## 기술 스택

- HTML / CSS / JavaScript (프레임워크·외부 라이브러리 없음)
- `index.html` 단일 파일 구성
- 브라우저 `localStorage`에 저장

## 실행 방법

`index.html` 파일을 브라우저에서 열면 바로 실행됩니다. 서버나 설치가 필요 없습니다.

- 지원 브라우저: 최신 Chrome, Edge, Firefox, Safari
- 데이터는 사용하는 브라우저에만 저장되며, 기기 간 동기화는 지원하지 않습니다.
- 자체 테스트: `index.html#test`로 열고 개발자 도구 콘솔에서 `[self-test] N passed, 0 failed`를 확인합니다.

## 문서

- [PRD.md](PRD.md) — 제품 요구사항 및 설계 문서
- [PROMPTS.md](PROMPTS.md) — PRD를 Claude Code로 구현하기 위한 5단계 프롬프트

## README 이전 버전

- [README-v1.md](README-v1.md) — 기획 단계 (PRD 작성 직후)
- [README-v2.md](README-v2.md) — 구현 프롬프트 추가 시점
