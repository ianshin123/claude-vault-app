# PROGRESS_INBOX — Claude Code 세션 공용 진행상황

> **규칙(→ CLAUDE.md)**: 대화 시작 시 이 파일을 먼저 읽어 → 새 항목을 맥락에 반영 → **반영한 항목 블록은 삭제하고 커밋**. 새 진행/결정은 여기에 append + 커밋.
> 형식: `## [YYYY-MM-DD · 세션] 요약` + 본문. 소비되면 그 블록 통째 삭제.

---

## [2026-07-20 · 데스크톱] 보안 감사 + 레거시 정리 (#4)

**배경**: 볼트·앱 전수 감사에서 토큰 유출 경로 3개 발견. 공개 Pages 앱이 비공개 볼트 PAT을 들고 있다는 게 위험의 핵심.

**고친 것**
- 🔴 `syncPush()`가 `delete data.sync`만 하고 **`hakjong.token`을 그대로 서버로 POST**하던 것 → 토큰 제거 후 전송. `syncPull()`·`importJSON()`은 로컬 토큰 보존하도록 대응 수정(안 그러면 pull마다 학종 연결 끊김).
- 🔴 `exportJSON()`이 토큰·동기화 비밀키·잠금 비번을 **평문 백업 파일에 기록**하던 것 → 전부 제외.
- 🟠 marked CDN에 SRI 없음 → `marked@14.1.4` 고정 + `integrity` sha384 + `crossorigin`. **버전 올릴 때 해시 같이 갱신 필수.**
- 🟠 볼트 노트의 **원시 HTML이 그대로 렌더**되던 것 → `hjSanitize()` 신설(`<template>` inert 파싱, script/iframe/form/svg 등 제거, `on*`·`javascript:` 제거). 위키링크는 `data-wl`(URI 인코딩)로 넘겨 따옴표 탈출 차단.
- 🟡 잠금 비번 평문 저장 → **SHA-256**(`settings.notePwH`). 옛 평문은 첫 해제 시 자동 승격.
- 🟡 `pullGist()` **미정의 함수 호출** 제거(부팅 최상위라 조건 걸리면 스크립트 사망).
- 🟡 죽은 레거시 47줄 제거: `planExam`·`planProj`·`editSubject`/`saveSubject`/`delSubject`/`toggleSubj` + `subjById`/`subjectExams`/`subjProgress`/`subjSortKey`. (`DB.subjects`/`exams`는 `load()`의 `_unify` 마이그레이션에서만 쓰이므로 **남겨둠 — 지우지 말 것**.)
- 🟡 `hjScanAll()` 순차 fetch(노트 수만큼 왕복) → 6개 동시.
- `sw.js` CACHE v6→v7. CLAUDE.md에 **보안 불변조건 4개** 섹션 신설 + 캐시 버전 하드코딩 제거(v2라 적힌 채 v6까지 갔던 문제).

**검증**: `node --check` 통과 + localhost로 실제 구동 → XSS 6종(onerror/script/javascript:/iframe/svg onload/form) 전부 무력화 확인, 토큰 push·export 미포함 확인, 전 탭 렌더·설정시트 예외 0.

**다음 세션이 알아야 할 것**
- 새 비밀값을 `DB`에 추가하면 **4곳(syncPush·syncPull·exportJSON·importJSON)을 세트로** 고쳐야 한다.
- 이 인박스 프로토콜이 #3 블록에서 안 지켜졌었다(그래프 탭 추가→제거 커밋 4개가 기록 안 됨). 커밋했으면 여기 남길 것.
