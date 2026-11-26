---
name: work-git
description: Commits, pushes, opens pull requests, and checks merges under the work Git policy — feature/<name> branches cut from dev, `<type>(<scope>): <subject>` messages (feat, fix, docs, style, refactor, test, chore, migrate, noop; subject 50 chars or fewer, no trailing period), feature/* → dev → main, PRs with a summary, changed files and impact, test method, and at least one reviewer, and no self-merge. Explicit invocation only.
argument-hint: "[commit | push | pr | merge] [what was changed / PR number / extra instructions]"
disable-model-invocation: true
---

# Work Git: commit, push, and PR under the work policy

Input: $ARGUMENTS

The policy is `references/git-rules.md` (the normative source; read it when a case below does not settle a question). This file turns it into the steps to follow. Write everything you report to the user in Korean.

A repository's own instruction (its CLAUDE.md, CONTRIBUTING, or a rule the user states now) overrides this policy where they conflict, as the policy itself allows. When that happens, follow the repository and say which rule you set aside.

## 0. Decide what was asked

- `commit` — commit only. `push` — commit what is pending, then push. `pr` — commit and push if needed, then open a PR. `merge` — check the merge conditions of a PR (§6).
- Nothing given, or only a description of the change: commit and push. Open a PR only when the user asks for one.
- The rest of the input is context: what changed, the feature name, reviewers, the PR number.

## 1. Look before acting

Run these and read the output before any change:

```bash
git status --short --branch
git branch -a --list
git log --oneline -10
git diff --stat; git diff --cached --stat
git remote -v
```

Stop and ask when:

- the repository has no `dev` branch (local or `origin/dev`): the whole flow assumes one, so ask which branch is the integration branch;
- the working tree holds files that look like secrets (`.env`, keys, credentials, `*.pem`) or large generated output: name them and leave them out unless the user says otherwise;
- a rebase, merge, or cherry-pick is in progress.

## 2. Get onto a feature branch

Commits for normal work go on `feature/<feature-name>`, cut from `dev`. The name is short, lowercase, hyphenated English that says what the feature is (`feature/login`, `feature/payment-retry`), taken from the change or from the user's words.

| Current branch | What to do |
|---|---|
| `feature/*` | Stay. If it was not cut from `dev` (`git merge-base --is-ancestor origin/dev HEAD` fails and the log shows another base), tell the user. |
| `dev` | `git switch -c feature/<name>` — uncommitted changes come along. |
| `main`, or anything else | `git fetch origin dev` then `git switch -c feature/<name> origin/dev`. Uncommitted changes come along when they apply cleanly; if git refuses, stop and ask — do not stash or reset on your own. If the branch already has commits that are not on `dev`, ask before moving them. |
| another prefix (`fix/x`, `hotfix/x`, a personal name) | The policy only defines `feature/*`. Ask whether to rename it (`git branch -m feature/<name>`) or keep it. |

Never commit directly on `main` or `dev`.

## 3. Split into commits

One commit is one meaningful change (§11 of the policy). If the working tree mixes changes — a bug fix and an unrelated refactor, a feature and a dependency bump — make one commit for each, in an order where every commit makes sense on its own.

- Stage by path: `git add <paths>`. `git add -p` is interactive and does not work here; to split hunks inside one file, write the hunks to a patch and `git apply --cached <patch>`.
- Before each commit, check what is staged with `git diff --cached --stat`.

## 4. Write the message

```text
<type>(<scope>): <subject>

<body>
```

**Type** — pick the one that fits the change best:

| Type | When |
|---|---|
| `feat` | adds new behavior |
| `fix` | corrects wrong behavior |
| `docs` | only documentation (README and the like) |
| `style` | formatting/style, no logic change |
| `refactor` | structure changes, behavior does not |
| `test` | adds or changes tests |
| `chore` | build, packages, configuration, maintenance |
| `migrate` | moves a source/original system into the business project |
| `noop` | no logical impact: a typo, a UI color |

When a commit contains both a feature and its tests, it is `feat` (the tests serve the feature). A commit that is only tests is `test`.

**Scope** — the class, function, module, or domain the change touches, as specific as is true: `feat(AuthService): ...`. Several scopes are comma-separated with no spaces: `refactor(AuthService,TokenProvider): ...`. Leave it out when nothing specific fits (`fix: 로그인 오류 수정`) rather than writing a vague one like `(code)` or `(all)`.

**Subject**

- Summarizes the change so a reader understands the purpose without the diff.
- 50 characters or fewer, counting the whole first line (type and scope included); one Korean character counts as one. Count it, do not estimate: `python -c "import sys; print(len(sys.argv[1]))" "<line>"` (an argument, not a pipe: piped stdin on Windows may decode Korean in the wrong code page and miscount).
- Imperative present tense, no trailing period. In Korean that is the noun-ending form the policy's examples use — `추가`, `수정`, `개선`, `삭제`, `변경` — not `추가함`, `추가했습니다`, `추가하였음`. In English, `add`, `fix` — not `added`, `fixes`.
- Language: follow the repository's recent commits; if the history is empty or does not follow this format, write Korean, as the policy's examples do.

**Body** — optional, but write it when the reason or the behavior change is not obvious from the subject:

- one blank line after the subject;
- why the change was made and how behavior differs from before, in imperative present tense, as `- ` bullets like the policy's examples;
- only reasons you know — from the user, the diff, an issue, or the code. Do not invent a motive ("느린 요청이 끊기지 않도록") the user never gave; when the reason is unknown, describe what changed and how behavior differs, or leave the body out;
- every line 75 characters or fewer.

Commit with a message file so the line breaks survive: write it to a temporary file, `git commit -F <file>`, then delete the file. If this session asks for attribution trailers on commits, add them after the body; they are not part of the body's content rules.

**Check before committing** — go through this and fix anything that fails:

```text
[ ] type is one of the nine
[ ] scope is specific, or deliberately left out
[ ] subject <= 50 characters (counted), no trailing ".", imperative
[ ] blank line between subject and body; body lines <= 75 characters
[ ] the body gives the reason / difference when it helps a reviewer
[ ] the commit holds one meaningful change
```

After committing, read it back with `git log -1 --format=%B`.

**Examples**

```text
docs(README): README 추가

- 프로젝트 시작 관련 내용 추가
- 의존성 패키지 설치
- 개발 모드 실행
- 프로젝트 빌드 실행
```

```text
fix(LoginService): 세션 만료 처리 수정

- 만료된 세션이 재사용되지 않도록 검증 추가
- 기존 로그인 흐름과의 차이점 정리
```

## 5. Push

- Push only the feature branch: `git push -u origin feature/<name>`.
- Do not push to `main` or `dev`: changes reach them through PRs. If the user asks for a direct push to either, say that the policy forbids it and offer the PR instead; do it only if they confirm a repository rule that allows it.
- Do not force-push unless the user asks, and never to `main` or `dev`. When asked, use `--force-with-lease`.
- If the push is rejected for permission (403, "permission denied"), report it: under the policy a `Member` has read access until write access is approved. Do not retry with other credentials.

## 6. Pull request and merge

**Opening a PR** (when asked):

- Base: `dev` for feature work. `main` only for a production release PR from `dev` (`dev → main`). A PR from `feature/*` to `main` is not allowed — stop and say so.
- Title: the commit-message format. With one commit, its subject; with several, one subject covering them all.
- Body: use this template, filled from the actual diff (`git diff origin/dev...HEAD --stat` and the commits):

```markdown
## 변경 요약

- 변경 사항 요약

## 주요 변경 파일 및 영향 범위

- `path/to/file`
- 영향 모듈 또는 기능

## 테스트 방법 / 실행 방법

1. 실행 절차
2. 검증 절차
3. 기대 결과
```

  Write only test steps you ran or that can actually be run; if nothing was tested, say so in that section instead of inventing steps.
- Reviewer: at least one, always. Use the reviewer the user named. If none was named, suggest one from `CODEOWNERS` or the recent authors of the touched files (`git log --format='%an' -- <paths>`), and ask the user to confirm. Priority: the owner of the affected module, then the part leader; add the team leader when the impact is broad, and all relevant people for an urgent fix. Do not create the PR without a reviewer.
- Create it with `gh pr create --base dev --head feature/<name> --title "<title>" --body-file <file> --reviewer <login>` and report the URL. If `gh` is missing or not logged in, give the user the title, body, and reviewer to paste instead.

**Merging** — you do not merge a PR the user wrote. You act with the user's credentials, so a merge you run on their PR is the author merging their own PR, which the policy forbids. When asked to merge, check and report:

```bash
gh pr view <n> --json author,baseRefName,headRefName,reviewDecision,reviews
gh api user --jq .login
```

```text
[ ] at least one approval
[ ] the PR author is not the person merging
[ ] feature/* goes into dev; only dev goes into main
```

Merge only when every box holds (for example, the user is the approving reviewer of someone else's PR). If any fails, stop and name the rule that is not met.

## 7. Report

End with a short Korean summary:

- branch (and whether it was created, from what);
- each commit: short hash and the full first line;
- push result, PR URL and reviewer, or merge check result;
- any rule you could not satisfy, any file you left out, and any repository rule that overrode the policy.
