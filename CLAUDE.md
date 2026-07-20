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
  - `loadHakjong()`=repo 트리 / `openHjNote()`=노트 raw md / `hjMd()`=마크다운 렌더(marked)+위키링크 인앱 이동 / `hjSanitize()`=원시 HTML 무력화 / `hjBacklinks()`=백링크(전체 스캔, 6동시) / `hjRenderTree()`=폴더 트리.
  - ⚠️ **프라이버시 절대 규칙**: 학종 내용은 **절대 빌드에 굽지 않는다.** 항상 토큰 런타임 fetch, 공개 Pages엔 앱 껍데기만. **Quartz식 SSG 금지**(내용이 공개돼버림).

## 🔒 보안 불변조건 (2026-07-20 감사 — 깨지 말 것)

> 공개 Pages에서 도는 앱이 **비공개 볼트 토큰**을 들고 있다. 아래 4개는 옵션이 아니라 불변조건이다. 새 기능 추가 시 매번 확인.

1. **토큰은 기기 밖으로 나가지 않는다.** `syncPush()`는 `sync`뿐 아니라 **`hakjong.token`도 반드시 비워서** 보낸다. `syncPull()`/`importJSON()`은 반대로 **로컬 토큰을 보존**한다(안 그러면 학종 연결이 끊김). 새 비밀값을 `DB`에 추가하면 이 4곳을 같이 고쳐라.
2. **`exportJSON()`은 비밀값을 안 쓴다** — `hakjong.token`·`sync`·`settings.notePw*` 제외. 백업 파일은 카톡·메일로 돌아다닌다는 전제.
3. **외부 스크립트는 버전 고정 + SRI 필수.** marked는 `marked@14.1.4` + `integrity`로 박혀 있다. **버전 올리면 해시도 같이 갱신**해야 하고, 안 하면 스크립트가 아예 안 뜬다(그게 정상 동작).
4. **볼트 노트 HTML은 항상 `hjSanitize()`를 거친다.** 노트의 원시 HTML이 그대로 렌더되면 노트 한 줄로 토큰을 털 수 있다. `marked.parse()` 결과를 직접 innerHTML에 넣지 말 것.
- 잠금 비번은 **SHA-256 해시로만** 저장(`settings.notePwH`). 평문 `notePw`는 레거시 승격용이며 새로 쓰지 않는다.

## 작업 규칙 (Claude Code)
- `index.html` 편집 후 **반드시 JS 문법 검사**: 인라인 스크립트 추출 → `node --check`. (한글 多, 템플릿리터럴 미닫힘 주의.)
- 문법 통과 ≠ 동작. **실제로 띄워서 확인**: `python -m http.server 8099` → 브라우저에서 `viewHTML()` 전 탭 + `openSettings()` 호출해 예외 0 확인. (file:// 로는 JS가 안 돈다.)
- 배포 파일 변경 시 `sw.js`의 `const CACHE` 버전 bump. **현재 값은 `sw.js`에서 직접 확인할 것** — 여기 숫자를 적어두면 반드시 낡는다(실제로 v2라 적힌 채 v6까지 갔었다).
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
