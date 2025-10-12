# Prompt Operations Library

## Overview
This repository contains a set of operational playbooks and prompts for running Codex CLI workflows. Each Markdown file captures the objective, guardrails, and expected outputs for a specific task you may encounter while assisting engineers, release managers, or product stakeholders.

## Prompt Catalogue
| File | Focus | Key Responsibilities |
| --- | --- | --- |
| `git-pr.md` | Automated PR creation | From unstaged changes, infer a branch name, produce AI-authored commit/PR messaging, push, and open the PR via `gh`. |
| `push-publish.md` | Release and publish workflow | Decide the next semantic version, update changelog and version metadata, tag and push, create a release, merge the PR, and publish if required. |
| `run-prompt.md` | Prompt execution logging | Log prompts to `.prompts/log.md` with metadata before running user instructions. |
| `wiki-changelog.md` | Wiki changelog automation | Summarise merged PRs per day in plain English, update `Changelog.md` in the GitHub Wiki, and push the changes idempotently. |

## Working With These Prompts
- Treat each file as an authoritative SOP: follow the sequence, guardrails, and reporting expectations it outlines.
- Many prompts assume access to standard tooling such as Git, the GitHub CLI (`gh`), and project-specific scripts; ensure you meet the prerequisites before execution.
- Maintain professionalism and accuracy—most instructions target high-scrutiny workflows like release management or executive reporting.

## Extending The Library
When adding new prompts:
1. Keep instructions concise, specific, and actionable.
2. Note any tooling assumptions, environment variables, or safety checks.
3. Describe the expected deliverables so operators can validate their work quickly.

By following these guidelines, contributors can continue to grow a reliable reference set for complex operational tasks.
