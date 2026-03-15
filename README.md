# Skills

Personal agent skills for Claude Code and other coding agents.

## Install

```bash
# Install all skills
npx skills add oronb/skills

# Install a specific skill
npx skills add oronb/skills --skill rxswift-concurrency-bridge
npx skills add oronb/skills --skill gett-push
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [gett-push](skills/gett-push) | Git workflow for gtforge repos — branch naming, Jira-linked commits, single-commit-per-PR, and auto-PR creation |
| [rxswift-concurrency-bridge](skills/rxswift-concurrency-bridge) | Use RxSwift's built-in Swift Concurrency bridging APIs instead of manual `withCheckedThrowingContinuation` wrappers |