# CLAUDE.md - GeekNews Theme Extension

## 프로젝트 개요

news.hada.io(GeekNews) CSS를 읽기 편한 매거진 스타일로 바꿔주는 Chrome Extension (Manifest V3).
원본 사이트의 작은 글자와 촘촘한 간격을 트렌디한 뉴스/매거진 스타일로 개선하는 것이 목표.

## 기술 스택

- Chrome Extension Manifest V3
- CSS Variables 기반 테마 시스템
- `chrome.storage.sync`로 테마 저장/동기화
- content script로 CSS + JS 주입

## 디자인 방향

- **레이아웃/타이포**: 매거진 스타일 (본문 17px, line-height 1.75, Pretendard 폰트)
- **라이트 테마**: 흰 배경(`#ffffff`), 진한 본문(`#1a1a1a`), 액센트 `#35a79c`(틸)
- **다크 테마**: GeekNews 기존 다크 팔레트 유지 (`#1d232c`, `#242b35`, 액센트 `#6666FF`)
- **적용 범위**: 리스트 + 상세 페이지 모두 통일

## 핵심 동작 방식

1. `content.js`가 `document_start`에 실행되어 `chrome.storage`에서 테마를 읽음
2. `<html data-gn-theme="light|dark">`를 설정
3. `content.css`의 CSS 변수가 해당 attribute selector로 활성화
4. 팝업에서 테마 변경 시 `chrome.storage.onChanged`로 실시간 반영

## 파일 구조

- `manifest.json` — 확장 설정 (permissions: storage, content_scripts 정의)
- `content.css` — CSS 변수 선언 + 전체 레이아웃/테마 (700줄+)
- `content.js` — 테마 적용 로직 (storage → data attribute)
- `popup.html` / `popup.js` — 라이트/다크 토글 UI

## 작업시 주의사항

### 원본 CSS 충돌 포인트

news.hada.io의 `main.css`는 아래 패턴들이 content.css와 충돌하므로 override시 항상 확인:

| 원본 규칙 | 문제 | 해결 방법 |
|---|---|---|
| `@media (prefers-color-scheme: dark) { textarea, input { background: #383B40 } }` | OS 다크모드시 input 색상 하드코딩 | `color-scheme: light/dark !important`로 강제 |
| `:root { color-scheme: dark light }` | 브라우저 UA 폼 렌더링에 영향 | html에서 테마별 color-scheme 덮어씀 |
| `.topic_row { grid-template-columns: 12px 24px 1fr }` | `grid-column-start: 3`이 암묵적 컬럼 생성 | grid 자체를 block으로 전환 |
| `.commentTD { overflow: hidden }` | 하위 요소 잘림 (pagination 버튼 등) | `.next { overflow: visible }` |
| `nav .geeknews { width: 4em }` | 폰트 키우면 텍스트 겹침 | `width: auto` |
| `.u { border-bottom: solid 1px }` | pagination 버튼에도 적용됨 | `.topicinfo .u` 등 스코프 제한 |
| `footer { display: table; background: #E5E5E5 }` | 테마 배경색 덮어씀 | `display: block; background: var()` |
| `article { max-width: 80% }` | 레이아웃 폭 제한 | `max-width: none` |
| `body { display: flex; flex-direction: column }` | sticky footer 패턴 | 유지하되 max-width/margin으로 중앙 정렬 |

### CSS 작성 원칙

- 원본 규칙과 충돌할 가능성이 있으면 `!important` 사용 (content script CSS는 페이지 CSS보다 먼저 로드됨)
- 새 셀렉터 추가 전에 반드시 원본 `main.css`에서 같은 셀렉터 확인
- 전역 태그 셀렉터(`button`, `a` 등)는 최소화 — 의도치 않은 요소에 영향
- `.u` 같은 범용 클래스는 부모 스코프로 한정

## 현재 진행 상태 (2026-04-16)

### 완료
- 메인 리스트 페이지 (.topic_row) — block 레이아웃
- 네비게이션 — flex 정렬, 검색창 고정폭
- Input/Textarea — color-scheme 강제, autofill 방지
- Footer — 테마 톤 통일
- 댓글 스레드 — 테마 컬러, 들여쓰기
- Pagination — overflow/border 이슈 해결

### 진행중
- 토픽 상세 페이지 (.topic) — vote+title 정렬 완료, 추가 다듬기 필요

### TODO
- 아이콘 제작
- 시스템 테마 자동 감지 옵션
- 폰트 크기 조절 슬라이더
- 서브 페이지 (write, login, profile 등) 스타일링
