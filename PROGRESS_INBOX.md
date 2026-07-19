# PROGRESS_INBOX — Claude Code 세션 공용 진행상황

> **규칙(→ CLAUDE.md)**: 대화 시작 시 이 파일을 먼저 읽어 → 새 항목을 맥락에 반영 → **반영한 항목 블록은 삭제하고 커밋**. 새 진행/결정은 여기에 append + 커밋.
> 형식: `## [YYYY-MM-DD · 세션] 요약` + 본문. 소비되면 그 블록 통째 삭제.

---

## [2026-07-20 · 데스크톱] 학종 탭 인앱 렌더러 도입 (#3)
- 학종 탭: 파일목록 → **인앱 마크다운 뷰어**. 노트 클릭 시 토큰으로 raw md fetch → marked 렌더 → `[[위키링크]]` 인앱 이동 → 백링크(전체 스캔 버튼). 프라이버시 유지(런타임 fetch, 공개 안 함).
- `marked@14` CDN 추가, `sw.js` CACHE `v1→v2`.
- 협업 체계를 Claude Code 기준으로 전환(#1): 이 `CLAUDE.md`·`PROGRESS_INBOX.md`·`.claude/settings.json` 신설. 옛 Cowork 지침은 이 repo엔 없었음.
- (확인·반영 후 이 블록 삭제)
