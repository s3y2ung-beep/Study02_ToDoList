# Claude Code 단계별 프롬프트 — 할 일 관리 앱

[PRD.md](PRD.md)를 Claude Code에서 순서대로 실행할 프롬프트 5개로 나눈 문서입니다.

## 사용 방법

- 단계마다 아래 코드 블록 하나를 그대로 복사해 Claude Code에 붙여넣습니다.
- **한 단계가 끝나고 결과를 확인한 뒤** 다음 단계로 넘어갑니다.
- 각 프롬프트는 새 세션에서 시작해도 동작하도록, 필요한 맥락을 `PRD.md`와 `CLAUDE.md`에서 읽게 되어 있습니다.
- 1단계에서 `CLAUDE.md`에 공통 규칙을 적어 두므로, 2~5단계는 그 규칙을 따릅니다.

| 단계 | 내용 | PRD 범위 |
|---|---|---|
| 1 | 뼈대 + 핵심 로직 + 자체 테스트 | 4장, 6.1, 9.1 |
| 2 | 저장소 + 경고 배너 | FR-8, 7장 |
| 3 | 목록·진행률·필터 + 추가·체크·삭제·정리 | FR-1, FR-3~7, FR-9 |
| 4 | 인라인 편집 | FR-2 |
| 5 | 반응형·접근성 마무리 + 전체 검증 | 5장, 8장, 9.2 |

---

## 1단계 — 뼈대, 핵심 로직, 자체 테스트

```text
PRD.md를 끝까지 읽고, 할 일 관리 앱의 1단계를 진행해줘.

## 목표
index.html 단일 파일의 뼈대를 만들고, 화면과 무관한 핵심 로직을 테스트와 함께 구현한다.

## 할 일
1. git 브랜치 feat/todo-app을 만들고 그 위에서 작업해.
2. CLAUDE.md를 만들고 아래 "공통 규칙"을 그대로 기록해.
   - 산출물은 index.html 하나. <style> 1개, <script> 1개 인라인. 외부 라이브러리·CDN·웹폰트 금지.
   - file://로 더블클릭해 열어도 동작해야 함. type="module", fetch 사용 금지.
   - 사용자 입력 문자열은 textContent / value로만 출력. innerHTML에 넣지 않음.
   - 스크립트 영역 순서와 주석 머리말: 상수 → 저장소 → 순수 로직 → 상태 → 상태 변경 → 렌더링 → 이벤트 → 자체 테스트 → 초기화
   - 모든 상태 변경은 "state 수정 → (todos가 바뀌면) 저장 → render()" 순서. DOM에서 데이터를 읽지 않음.
   - 화면 문구는 PRD.md에 적힌 문자열을 그대로 사용.
   - 테스트 실행: 브라우저에서 index.html#test를 열고 콘솔의 "[self-test]" 줄을 확인.
3. index.html 뼈대를 만들어. 영역 주석, 상수(STORAGE_KEY="todo-app.v1", MAX_TITLE=100, CATEGORIES는 PRD 4.3 그대로),
   그리고 화면 영역 요소: #banner, #progress, #add-form(#new-title, #new-category, 추가 버튼), #add-error, #tabs, #list, #clear-done.
4. 자체 테스트 도구를 먼저 만들어: test(name, fn), assertEqual(actual, expected), runSelfTests().
   URL 해시가 #test일 때만 실행하고, 실패는 "[self-test] FAIL 이름: 내용", 마지막 줄은 "[self-test] N passed, M failed"로 콘솔에 출력.
5. 아래 함수의 테스트를 먼저 작성하고, 실패하는 것을 확인한 다음 구현해 (TDD).
   - validateTitle(raw) → {ok:true, title} | {ok:false}  : trim 후 1~100자. 공백만/빈 문자열/문자열 아님/101자 → 실패
   - createTodo(title, category, now) → {id, title, category, done:false, createdAt:now}, id는 매번 다름
   - sortTodos(todos) → 새 배열. 미완료 먼저, 각 그룹은 createdAt 오름차순. 원본 배열은 바꾸지 않음
   - filterTodos(todos, filter) → "all" 또는 카테고리 키로 거르기
   - calcProgress(todos) → {all, work, personal, study}, 각각 {done, total, percent}. percent는 반올림 정수, total 0이면 0
   - parseStored(raw) → {todos, status:"empty"|"ok"|"corrupt", skipped}
     null → empty / JSON 아님·version≠1·todos가 배열 아님 → corrupt /
     잘못된 항목(알 수 없는 카테고리, 제목 검증 실패, done이 boolean 아님, createdAt이 숫자 아님, 중복 id)은 건너뛰고 개수를 skipped에 / 제목은 trim해서 저장

## 하지 말 것
- 화면 그리기, 이벤트 처리, localStorage 접근은 아직 만들지 마. (2~3단계 범위)

## 완료 조건
- index.html#test 콘솔 마지막 줄이 "[self-test] N passed, 0 failed"이고 FAIL 줄이 없음.

## 마지막에
- index.html과 CLAUDE.md를 커밋하고, 테스트 결과 출력과 만든 함수 목록을 보고해줘.
```

---

## 2단계 — 저장소와 경고 배너

```text
PRD.md와 CLAUDE.md를 읽고, 할 일 관리 앱의 2단계를 진행해줘. (1단계는 완료됨)

## 목표
localStorage 저장·불러오기를 만들고, 저장이 안 되거나 데이터가 깨졌을 때 사용자에게 알린다. (PRD FR-8, 7장)

## 할 일
1. 아래 함수의 테스트를 먼저 작성하고 실패를 확인한 뒤 구현해.
   테스트에는 진짜 localStorage 대신 가짜 저장소 객체(getItem/setItem)와 항상 예외를 던지는 저장소를 써.
   - getStorage() → localStorage를 쓸 수 있으면 그 객체, 접근 시 예외가 나면 null
   - loadTodos(storage, now) → {todos, warning: null | "unavailable" | "corrupt"}
     · storage가 null → "unavailable"
     · 데이터가 깨졌으면 원본 문자열을 "todo-app.v1.backup-<now>" 키에 백업하고 "corrupt"
     · 건너뛴 항목이 있으면 console.warn으로 개수 기록
   - saveTodos(storage, todos) → 성공 true / storage가 null이거나 저장 중 예외면 false
     저장 형식: {"version":1,"todos":[...]}
2. 상태 객체를 만들어:
   state = { todos: [], filter: "all", editingId: null, editDraft: null, editError: false, lastCategory: "work", warning: null }
3. render()와 renderBanner()를 만들어. 이번 단계의 render()는 배너만 그림.
   경고 문구:
   - unavailable: "데이터를 저장할 수 없습니다. 새로고침하면 사라집니다."
   - corrupt: "저장된 데이터를 읽지 못해 백업 후 초기화했습니다."
4. 초기화: 저장소 준비 → loadTodos → state 반영 → render().
5. 다른 탭에서 데이터가 바뀌면(window의 storage 이벤트, 키가 todo-app.v1일 때) 최신 목록으로 다시 그려.
   편집 중이던 항목이 사라졌으면 편집 상태도 해제해.

## 하지 말 것
- 목록, 진행률, 입력 처리는 아직 만들지 마. (3단계 범위)

## 완료 조건
- index.html#test 전부 통과.
- 콘솔에서 localStorage.setItem("todo-app.v1", "x") 후 새로고침하면 corrupt 배너가 보이고, backup 키가 생김. 확인 후 테스트용 키는 지워.

## 마지막에
- 커밋하고, 테스트 결과와 배너 확인 결과를 보고해줘.
```

---

## 3단계 — 목록·진행률·필터와 추가·체크·삭제·정리

```text
PRD.md와 CLAUDE.md를 읽고, 할 일 관리 앱의 3단계를 진행해줘. (1~2단계는 완료됨)

## 목표
화면의 핵심을 완성한다: 할 일 추가, 완료 체크, 삭제, 완료 항목 정리, 카테고리 필터, 진행률. (PRD FR-1, FR-3~FR-7, FR-9)

## 할 일
1. commit(todos) 헬퍼: state.todos 교체 → 저장(실패하면 state.warning="unavailable") → render().
2. 상태 변경 함수: addTodo(rawTitle, category) → boolean, toggleTodo(id), deleteTodo(id), clearDone(), setFilter(filter).
3. render()가 배너 다음에 진행률 → 필터 탭 → 목록 → 정리 버튼 순서로 그리게 해.
4. 세부 동작은 PRD를 그대로 따르고, 특히 아래를 지켜:
   - 추가: Enter 또는 [추가]. 공백만이면 추가하지 않고 "할 일을 입력하세요" 표시(다시 입력하면 숨김).
     추가 후 입력칸 비우고 포커스 유지, 카테고리 선택은 유지.
   - 한글 입력: 입력칸 keydown에서 Enter이면서 e.isComposing이면 preventDefault (조합 중 Enter로 두 번 추가되는 문제 방지).
   - 카테고리 기본값: 필터 탭이 카테고리면 그 값, "전체"면 사용자가 마지막으로 직접 고른 값(state.lastCategory).
   - 목록: 미완료 위, 완료 아래(취소선 + 흐리게). 항목마다 체크박스, 제목, 카테고리 색 배지, ✎, 🗑.
     체크박스와 버튼에 aria-label(예: "보고서 작성 삭제"). 클릭 처리는 #list 하나에 이벤트 위임(data-id, data-action).
   - 필터 탭: [전체 n] [업무 n] [개인 n] [공부 n], 선택된 탭 강조 + aria-pressed.
   - 진행률: "전체 진행률", "완료 / 전체 (퍼센트%)" + 막대. 카테고리별 "업무 3/5" + 작은 막대.
     필터와 관계없이 항상 전체 데이터 기준. 0개일 때 "할 일을 추가해 보세요", 0개 카테고리는 흐리게.
   - 빈 목록: "할 일이 없습니다" / 필터 결과가 비면 "이 카테고리에 할 일이 없습니다".
   - 정리 버튼: 완료 항목이 있을 때만 "완료 항목 정리 (n)". confirm("완료한 할 일 n개를 삭제할까요?") 후 필터와 무관하게 모든 완료 항목 삭제.
5. 위 화면이 보기 좋을 정도의 기본 스타일(최대 너비 600px 가운데 정렬, 배지 색, 진행률 막대).

## 하지 말 것
- ✎ 버튼(인라인 편집)은 아직 연결하지 마. (4단계 범위)

## 완료 조건
- index.html#test 전부 통과 (기존 테스트 회귀 없음).
- PRD 9.2 인수 테스트 1, 2, 3, 4, 7, 8, 9, 10, 12번을 실제로 실행해서 통과.

## 마지막에
- 커밋하고, 인수 테스트별 결과를 보고해줘.
- 한글 입력 후 바로 Enter를 눌렀을 때 1개만 추가되는지는 직접 확인이 필요하다고 알려줘.
```

---

## 4단계 — 인라인 편집

```text
PRD.md와 CLAUDE.md를 읽고, 할 일 관리 앱의 4단계를 진행해줘. (1~3단계는 완료됨)

## 목표
할 일을 목록 안에서 바로 수정하는 인라인 편집을 만든다. (PRD FR-2)

## 할 일
1. 함수: startEdit(id), cancelEdit(), saveEdit() → boolean.
2. 편집 진입: ✎ 버튼 클릭 또는 제목 더블클릭.
   편집 줄 구성: [제목 입력칸(maxlength 100)] [카테고리 선택] [저장] [취소], 진입 시 제목 입력칸에 포커스 + 전체 선택.
3. 저장: Enter 또는 [저장]. 제목이 공백뿐이면 편집을 유지하고 "할 일을 입력하세요" 표시.
   취소: Esc 또는 [취소]. 포커스가 빠져나가도(blur) 자동 저장하지 않음.
   한글 조합 중 Enter(e.isComposing)는 무시.
4. 한 번에 한 항목만 편집. 편집 중 다른 항목의 ✎를 누르면 기존 편집은 취소하고 새 항목을 편집.
5. 편집 중 입력값은 state.editDraft에 계속 저장해서, 다른 항목을 체크하는 등 화면이 다시 그려져도 입력 내용과 편집 상태가 유지되게 해.
   다시 그릴 때 포커스를 강제로 옮기지 마 (포커스는 편집 진입 시에만).
6. 편집 중인 항목을 삭제하면 편집 상태도 해제.

## 완료 조건
- index.html#test 전부 통과.
- PRD 9.2 인수 테스트 5, 6번 통과.
- 추가 확인: 편집 중 다른 항목 체크 → 입력 중이던 내용 유지 / 공백만 남기고 저장 → 편집 유지 + 안내 문구 /
  편집 칸에 150자 입력 → 100자에서 잘림 / 편집 중인 항목 삭제 → 편집 해제.

## 마지막에
- 커밋하고, 확인 결과를 항목별로 보고해줘.
```

---

## 5단계 — 반응형·접근성 마무리와 전체 검증

```text
PRD.md와 CLAUDE.md를 읽고, 할 일 관리 앱의 마지막 5단계를 진행해줘. (1~4단계는 완료됨)

## 목표
모바일 화면과 키보드 사용을 다듬고, PRD의 모든 요구사항을 처음부터 끝까지 검증한 뒤 README를 갱신한다. (PRD 5장, 8장, 9.2)

## 할 일
1. 반응형: 너비 360px에서 가로 스크롤이 없어야 함. 입력 줄·편집 줄은 줄바꿈 허용, 긴 제목은 줄바꿈.
2. 접근성: 버튼·체크박스·탭에 :focus-visible 외곽선, 진행률 막대에 role="progressbar"와 aria-valuenow/min/max,
   경고 배너에 role="alert". 키보드(Tab, Enter, Space, Esc)만으로 모든 기능 사용 가능.
3. 전체 검증:
   - index.html#test 전부 통과.
   - PRD 9.2 인수 테스트 1~14번 전부 실행. 13번은 너비 360px에서 확인.
   - 할 일 100개를 넣고 체크·삭제·탭 전환이 느려지지 않는지 확인 후 테스트 데이터 삭제.
   - 같은 파일을 탭 두 개로 열고, 한쪽에서 추가한 항목이 다른 탭에도 나타나는지 확인.
4. 실패한 항목이 있으면 고친 뒤 해당 테스트를 다시 실행해서 통과를 확인해.
5. README.md 갱신: "현재 상태"를 구현 완료로 바꾸고, "(계획)", "(구현 후)" 표시를 제거하고,
   자체 테스트 실행 방법(index.html#test + 콘솔) 한 줄을 추가.

## 완료 조건
- 위 검증 항목이 모두 통과하고, 그 근거(실행한 확인과 결과)를 보고에 포함.

## 마지막에
- 커밋하고, PRD 요구사항별 충족 여부를 표로 정리해줘.
- 한글 입력(IME) 확인처럼 사람이 직접 봐야 하는 항목은 따로 목록으로 알려줘.
- GitHub에 push할지는 나에게 먼저 물어봐.
```
