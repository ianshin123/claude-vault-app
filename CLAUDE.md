# CLAUDE.md — 플래너 앱 (Claude Vault Planner)

> 이 repo를 다루는 **Claude Code** 세션(데스크톱·모바일)의 단일 행동 지침. (옛 Cowork 협업 방식은 폐기 — 이제 Claude Code 기준.)

## 프로젝트
- **단일 파일 PWA**: 전부 `index.html`(HTML+CSS+JS 인라인). 빌드 체인 없음.
- **배포**: GitHub Pages (`.nojekyll`). `index.html` push → 바로 반영. `sw.js`는 HTML network-first라 즉시 최신, 정적자산 cache-first.
- **탭 3개**: 오늘(today)·달력(cal)·학종(hakjong). 하단 도크.

## 데이터 모델 (localStorage `planner_v2` = `DB`)
- `DB.projects[]`(과목=프로젝트: `exams[]`·`milestones[]`), `tasks[]`, `repeats[]`, `days{}`, `memos[]`, `settings`, `sync`, `hakjong`.
- **플래너 데이터 sync**: `DB.sync{url,key}` → 커스텀 엔드포인트 POST/GET(`X-Sync-Key`). 기기 간 동기화.
- **학종 뷰어**: `DB.hakjong{repo,branch,token}` → GitHub API로 **private brain-vault**를 런타임 fetch.
  - `loadHakjong()`=repo 트리 / `openHjNote()`=노트 raw md / `hjMd()`=마크다운 렌더(marked)+위키링크 인앱 이동 / `hjBacklinks()`=백링크(전체 스캔) / `hjRenderTree()`=폴더 트리.
  - ⚠️ **프라이버시 절대 규칙**: 학종 내용은 **절대 빌드에 굽지 않는다.** 항상 토큰 런타임 fetch, 공개 Pages엔 앱 껍데기만. **Quartz식 SSG 금지**(내용이 공개돼버림).

## 작업 규칙 (Claude Code)
- `index.html` 편집 후 **반드시 JS 문법 검사**: 인라인 스크립트 추출 → `node --check`. (한글 多, 템플릿리터럴 미닫힘 주의.)
- 배포 파일 변경 시 `sw.js`의 `const CACHE` 버전 bump(현재 `cv-planner-v2`).
- 커밋은 작은 단위. 커밋 후 진행상황을 `PROGRESS_INBOX.md`에 남긴다(아래 프로토콜).

## 🔄 세션 협업 프로토콜 (여러 CC 세션 공용 — #2)
**모든 대화 시작 시 기본 동작:**
1. `PROGRESS_INBOX.md`를 **먼저 읽는다**.
2. 새 항목을 **내 맥락에 반영**.
3. 반영한 항목은 **파일에서 삭제 + 커밋**(중복 방지).
4. 내가 작업하며 다른 세션이 알아야 할 진행/결정은 `PROGRESS_INBOX.md`에 **append + 커밋**.
→ 사용자가 복붙 안 해도 세션끼리 진행을 이어받음. `.claude/settings.json`의 UserPromptSubmit 훅이 매 메시지 인박스를 자동 노출한다.

## 톤
- 소유자 = 신이안. 비판적·직설적, 막연한 칭찬 X, 실질 피드백.
