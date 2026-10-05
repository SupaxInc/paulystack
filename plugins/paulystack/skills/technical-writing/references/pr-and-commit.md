# PR descriptions and commit messages

## Example

Before (typical default):

```
feat: Add comprehensive technical writing skill 🚀

- Added SKILL.md
- Added references/rfc.md
- Added evals
```

After:

```
Add technical-writing skill to paulystack

Writes and reviews RFCs, PR descriptions, commits, Jira tickets and
READMEs. Each doc type loads its own rules file. The eval suite
compares runs with and without the skill.
```

## Commit messages

- Subject: 72 characters or fewer, imperative ("Add", "Fix", "Remove"), naming what changed. No trailing period.
- Match the repo's convention when `git log --oneline -10` shows one (Conventional Commits prefixes, ticket keys, scopes). The repo's style beats these defaults.
- Body only when the why isn't obvious from the subject: why the change was needed, or what it deliberately doesn't do. Wrap at about 72 columns.
- No file lists, no emoji, no "This commit ...".
- Don't commit. Give the message; the user commits.

## PR descriptions

A PR description is a briefing a reviewer reads in under a minute before opening the diff.

- Describe the whole branch against its base, as it is now. Don't narrate review history or earlier approaches ("after feedback, I changed ...").
- Use the smallest structure that fits:
  - one-line fix: one or two sentences;
  - a feature or behavior change: what it does now, then why, in a few short paragraphs;
  - a risky or wide change: add where to review first and what could break.
- Don't add default `Summary`, `Changes`, or `Test Plan` headings. Add a heading only when the PR has two things a reviewer must find separately.
- No file lists (the diff shows them), no pasted logs or command output (link them), no restating the title.
- Say how it was verified only when that isn't obvious from the diff (the tests in the diff run in CI; a manual check against staging does not).
- Link issues and tickets only when you've seen the number in the branch, commits, or conversation.
