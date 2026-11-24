# AGENTS.md

Instructions for coding agents (Claude, Codex, Gemini, Cursor and others) working on Shainy Hsu's web tools. This repo is the hub: the home page plus the shared project state. Each tool lives in its own repo; see README.md.

## Read first

1. `.ai/00_INTERFACE_CONTRACT.md`: repos, function index with line numbers, data contract
2. `.ai/01_CURRENT_STATE.md`: the one task in progress and its done criteria
3. `.ai/02_DECISION_LOG.md`: decisions to keep and dead ends not to retry
4. `.ai/03_HANDOFF_PROMPT.md`: the same rules as a prompt for chat-based LLMs

## Rules (SSSM short-context protocol)

1. One atomic task per session: the one named in 01. Make the smallest change (a diff or the whole target function). Never rewrite a whole file. Keep the single-file pages, their structure and names.
2. Read selectively. Find a function with `grep -n`, then read at most 150 lines or 6,000 characters at a time. Never load a whole large file (class-hiking-calculator/index.html is about 6,100 lines).
3. Keep facts and ideas apart. Nothing in 01 section 3 is implemented. Mark any new algorithm or structure [Unverified] until Shainy has tested it.
4. Check the dead ends in 02 before proposing anything.
5. Modern Web Guidance 1.0. This project's Baseline target is "Baseline Widely available". Prefer native APIs (`<dialog>`, Popover API, CSS Grid/Subgrid, IndexedDB/OPFS) over polyfills and libraries.
6. End every reply with this status brief, 180 characters or fewer, instead of a recap:

```
【SSSM 狀態更新簡報】
- Applied（已改動）：
- Status（事實狀態）：Verified／Unverified；Done Criteria 達成與否
- Dead Ends（新增踩坑）：無
- Next Action（下一步唯一動作）：
```

## Repo conventions

- Talk to Shainy in Traditional Chinese (Taiwan usage). Page text and commit messages follow each repo.
- Each update of a page is one version: add a line to the README "Version history" and to CHANGELOG.md (`V<n> | YYYY-MM-DD HH:MM | one short line`, Taiwan time, UTC+8).
- Work on a branch and open a pull request; merge it only when Shainy asks.
- Line numbers drift between versions. Trust function names, and update the index in 00 when you change a page.
