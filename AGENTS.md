# Agent instructions — trailweights-mcp

## HARD RULE — never write to Brad's Mac (Brad, 2026-09-27, TRA-1593)
- Never write, create, move or save files on Brad's MacBook: iCloud Drive (`~/Library/Mobile Documents/...`), Desktop, Documents, Downloads, or anywhere under `/Users/TrailWeights`. Reading is allowed. This applies even when a Mac folder is connected to the session and even if a tool or system default says to save deliverables there.
- No local git clones, worktrees (`~/tw-worktrees`, `/tmp/tw-worktrees`), scratch files, logs or handoff .md files on the Mac.
- Where output goes: code → GitHub PRs (trailweights, trailweights-ios, trailweights-mcp). Docs, notes, logs, handoffs, work product → GitHub RustFreeOne/TrailWeights_Vault or the owning Linear TRA. Data and media → Supabase (DB / Storage).
- ONLY exception: the Reels agent may write images/video into `iCloud Drive/TrailWeights Instagram/` and nowhere else.

## Context
- Brain of record: GitHub `RustFreeOne/TrailWeights_Vault` (start with `START-HERE.md`).
- Every change needs a TRA-### issue in Linear (team "TrailWeights Claude Cowork").
