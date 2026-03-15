# Skills

Personal agent skills for Claude Code and other coding agents.

## Install

**Quick install:** Use the [Skill Scraper](https://github.com/oronbz/skill-scraper) Chrome extension to browse and install skills directly from this page with one click.

**CLI:**

```bash
# Install all skills
npx skills add oronbz/skills

# Install a specific skill
npx skills add oronbz/skills --skill rxswift-concurrency-bridge
npx skills add oronbz/skills --skill gett-push
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [gett-push](skills/gett-push) | Git workflow for gtforge repos — branch naming, Jira-linked commits, single-commit-per-PR, and auto-PR creation |
| [rxswift-concurrency-bridge](skills/rxswift-concurrency-bridge) | Use RxSwift's built-in Swift Concurrency bridging APIs instead of manual `withCheckedThrowingContinuation` wrappers |