# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LaTeX Editor Pro** — 단일 HTML 파일로 구성된 LaTeX 텍스트 찾기/바꾸기 도구. 빌드 과정 없이 브라우저에서 직접 `index.html`을 열어 실행한다.

## Architecture

전체 앱이 `index.html` 하나에 CSS, HTML, JS가 모두 포함되어 있다.

### 핵심 데이터 구조

```js
data = {
  single: { activeId, tabs: [{id, name, f, r, reg}] },
  batch:  { activeId, tabs: [{id, name, list: [{f, r, reg}]}] }
}
```

`localStorage` 키 `latex_editor_persistent_v1`에 전체 상태를 JSON으로 저장. 구버전 키(`LEGACY_KEYS`)에서 마이그레이션 로직 포함.

### 주요 흐름

- **하이라이트**: `refreshHighlights()` — 에디터 내 모든 마크 초기화 후 재적용. 단일 세트는 `hl-0` 고정, 묶음 세트는 행 인덱스에 따라 `hl-0~17` 순환 (18색).
- **hangblock 전용 색상**: `\begin{hangblock}[..]{..}{..}` 와 `\end{hangblock}` 는 진한 주황(`#ff8c42`, `cm-hangblock-custom`), `\hblinebreak` 는 보라(`#c586c0`, `cm-hblinebreak-custom`). CSS specificity는 `.CodeMirror .CodeMirror-scroll .CodeMirror-code` 한정자로 `cm-keyword-custom` 보다 높게 유지해야 색이 덮이지 않음. 자식 토큰까지 색을 입히려면 셀렉터에 `*` 를 함께 둘 것.
- **치환 실행**: `runSingle()` / `runBatch()` — regex 모드 시 캡처 그룹 `\1` → `$1` 변환 처리.
- **엑셀 붙여넣기**: `handleExcelPaste()` — 탭/개행 구분 TSV 형식으로 여러 행을 한 번에 입력.
- **탭 관리**: `addTab`, `switchTab`, `deleteTab`, `renameTab` — 탭 전환 시 항상 `saveCurrentData()` 먼저 호출.

### 외부 의존성 (CDN)

- CodeMirror 5.65.13 (codemirror.min.js, stex mode, searchcursor addon)

## 개발 방법

별도 서버/빌드 불필요. `index.html`을 브라우저에서 직접 열면 된다.

로컬 스토리지 초기화가 필요한 경우 브라우저 개발자 도구 콘솔에서:
```js
localStorage.removeItem('latex_editor_persistent_v1')
```
