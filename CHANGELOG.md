# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- Agent skill configuration (`AGENTS.md`, `docs/agents/`) for issue tracking, triage labels, and domain docs — sets up GitHub issues, the default triage label vocabulary, and single-context domain doc rules for engineering skills to read
- Added this repo (repo-audit itself) to the checklist's own data — it didn't exist at the time of the original scan
- `repo-audit.html`: a checklist artifact auditing every repo under
  `~/projects` plus the dotfiles repo against a best-practice standard
  (clean tree, named branch, `main`, configured origin), with per-repo
  and copy-all fix-command generation.
