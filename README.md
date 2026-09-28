# Repo Audit

A checklist artifact for verifying every git repo under `~/projects`
(plus the dotfiles repo) against a small best-practice standard: clean
working tree, on a named branch, branch is `main`, has a configured
origin. Rows that don't meet it get a **Fix** button with exact,
copyable git commands computed from that repo's own data; a
**Copy all fix commands** button assembles every gap across every repo
into one pasteable script.

Live: https://claude.ai/artifact/P1T51fEGN8Gm4FPQhxuhj9

`repo-audit.html` is the source. To update the live page, republish it
to that same artifact URL.

Built 2026-09-28. The repo data is a snapshot from a scan at that time —
rescan and regenerate the `mainRepos`/`flagged` arrays if repos have
moved, been added, or been removed since.
