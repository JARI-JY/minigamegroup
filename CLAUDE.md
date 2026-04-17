# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

순수 HTML/CSS/JS로 구성된 정적 웹앱. 빌드 도구·패키지 매니저·프레임워크 없이 파일을 브라우저에서 직접 열거나 GitHub Pages로 서빙한다.

- **라이브 URL**: https://jari-jy.github.io/minigamegroup/
- **저장소**: https://github.com/JARI-JY/minigamegroup

## 개발 방법

```bash
# 브라우저에서 직접 열기 (로컬 미리보기)
start index.html          # Windows
open index.html           # macOS

# 변경 후 배포 (훅이 자동 처리하지만 수동 시)
git add -A && git commit -m "메시지" && git push origin main
```

빌드·테스트·린트 도구 없음. 수정 즉시 브라우저에서 확인 가능.

## 파일 구조

```
index.html            ← 메인 진입점 (GitHub Pages 루트)
game.html             ← 브레이크아웃 단독 플레이 페이지
breakout-landing.html ← 브레이크아웃 소개 랜딩 페이지
.claude/settings.json ← PostToolUse 자동 커밋 훅
```

## 아키텍처: index.html SPA

5개 게임이 단일 HTML 파일 안에 모두 구현된 SPA.

### 화면 전환 패턴

```
#hub (게임 선택 화면)
  ↕ launchGame(name) / exitGame()
#game-view (게임 플레이 화면)
  └─ #breakout-container
  └─ #snake-container
  └─ #tetris-container
  └─ #2048-container
  └─ #minesweeper-container
```

`launchGame(name)` → `#hub` 숨김, 해당 컨테이너 `display:flex`, `activeGame = name`, `INITS[name]()` 호출  
`exitGame()` → `STOPS[name]()` 호출, `#game-view` 숨김, `#hub` 복원

### 게임별 구현 방식

| 게임 | 렌더링 | 루프 방식 | 핵심 변수 prefix |
|------|--------|-----------|-----------------|
| 브레이크아웃 | Canvas `#bo-canvas` | `requestAnimationFrame` | `bo*` |
| 스네이크 | Canvas `#sn-canvas` | `setInterval` | `sn*` |
| 테트리스 | Canvas `#tet-canvas` | `requestAnimationFrame` | `tet*` |
| 2048 | DOM `#g2-board` | 키 이벤트 기반 | `g2*` |
| 지뢰찾기 | DOM `#ms-board` | 클릭 이벤트 기반 | `ms*` |

### 키보드 이벤트 구조

모든 게임이 `document.addEventListener('keydown', ...)` 를 각각 등록.  
각 핸들러 첫 줄에서 `if (activeGame !== '게임명') return;` 으로 충돌 방지.  
**새 게임 추가 시 반드시 이 패턴 준수.**

### 게임 생명주기 함수 규칙

각 게임은 다음 두 함수를 구현:
- `xyzInit()` — 상태 초기화, 오버레이 표시, 첫 프레임 렌더
- `xyzStop()` — `cancelAnimationFrame` 또는 `clearInterval`, 타이머 정리

`INITS` / `STOPS` 객체에 등록해야 `launchGame` / `exitGame`이 정상 동작.

## 디자인 토큰 (CSS 변수)

```css
--primary: #00d4ff   /* 시안 — 주요 강조 */
--accent:  #ff00aa   /* 핑크 — 보조 강조 */
--bg:      #0a0a1a   /* 페이지 배경 */
--card:    #1a1a2e   /* 카드/패널 배경 */
--border:  #2a2a4a   /* 테두리 */
--dim:     #888      /* 흐린 텍스트 */
```

새 UI 요소에는 하드코딩 대신 위 변수 사용.

## Git / 배포 규칙

- `main` 브랜치에 push하면 GitHub Pages가 자동 재배포됨 (보통 1~2분 소요)
- `.claude/settings.json` PostToolUse 훅이 Write/Edit 후 자동으로 `git add -A && git commit && git push origin main` 실행
- 커밋 메시지 형식: `feat:` / `fix:` / `chore:` / `auto:` (훅 자동 생성)
