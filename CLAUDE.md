# CLAUDE.md — 할 일 관리 앱

요구사항은 [PRD.md](PRD.md), 단계별 구현 지시는 [PROMPTS.md](PROMPTS.md)를 기준으로 한다.

## 공통 규칙

- 산출물은 `index.html` 하나. `<style>` 1개, `<script>` 1개 인라인. 외부 라이브러리·CDN·웹폰트 금지.
- `file://`로 더블클릭해 열어도 동작해야 함. `type="module"`, `fetch`, `crypto.randomUUID()` 사용 금지.
- 사용자 입력 문자열은 `textContent` / `value`로만 출력. `innerHTML`에 넣지 않음.
- 스크립트 영역 순서와 주석 머리말: 상수 → 저장소 → 순수 로직 → 상태 → 상태 변경 → 렌더링 → 이벤트 → 자체 테스트 → 초기화
- 모든 상태 변경은 "state 수정 → (todos가 바뀌면) 저장 → `render()`" 순서. DOM에서 데이터를 읽지 않음.
- localStorage 키는 `todoApp.v1`, 값은 `{"version":1,"todos":[...]}`. 읽고 쓰는 곳은 저장소 영역 함수뿐.
- 화면 문구는 PRD.md에 적힌 문자열을 그대로 사용.
- 커밋 메시지는 한글로, 무엇을 왜 바꿨는지 적는다. 단계마다 커밋한다.

## 테스트

- 자체 테스트: 브라우저에서 `index.html#test`를 열고 콘솔의 `[self-test] N passed, 0 failed` 확인 (현재 24개).
- 화면 동작: PRD 9.2 인수 테스트 1~15번을 브라우저에서 직접 확인.
- 한글 입력 후 바로 Enter → 1개만 추가되는지는 사람이 직접 확인해야 함 (자동 재현 불가).

## 주요 결정 (이유는 PRD 11장)

- 진행률은 필터와 무관하게 항상 전체 기준 + 카테고리별 표시.
- 완료 항목은 목록 아래로, 각 그룹은 추가한 순서.
- 전체 다시 그리기 방식이므로 `render()`가 키보드 포커스와 편집 칸 커서 위치를 보존한다.
- 깨진 저장 데이터는 백업 키에 보관한 뒤 원래 키를 초기화한다.

## 배포

- GitHub Pages, `main` 브랜치 루트: https://s3y2ung-beep.github.io/Study02_ToDoList/

## 다음 할 일 (강의 4.3~)

- 4.3: 데스크톱 넓은 화면용 2단 레이아웃, 현재 버전은 `mobile_version/`으로 보존.
- 4.4: Pages 주소에서 개발자 도구로 `JSON.parse(localStorage.getItem("todoApp.v1"))` 확인.
