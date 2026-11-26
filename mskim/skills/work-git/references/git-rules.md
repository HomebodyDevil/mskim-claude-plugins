# Git Rules

> AI-readable Git policy derived from the provided **Git 규칙** document.
> This document normalizes the original rules into explicit `MUST`, `SHOULD`, and `MAY` statements so that AI coding agents can apply them consistently.
> No additional policy has been introduced beyond the source document.

---

## 1. Purpose

This document defines the repository permission model, branch strategy, development workflow, commit-message convention, Pull Request requirements, reviewer policy, and merge rules.

AI agents working in this repository MUST treat this document as the authoritative Git workflow policy unless a higher-priority repository instruction explicitly overrides it.

---

## 2. Normative Keywords

The following keywords are used in this document:

- **MUST**: mandatory requirement.
- **MUST NOT**: prohibited behavior.
- **SHOULD**: recommended behavior unless there is a justified exception.
- **MAY**: optional behavior.

---

# 3. Repository Permission Model

## 3.1 Permission Groups

| Group | Role | Permission Scope | Rules |
|---|---|---|---|
| `Leader` | Part leader / Team leader | Entire repository | Administrator privileges; MAY approve branches and PRs |
| `Member` | General member | Entire repository | Read access by default; write access is granted only after request/approval |
| `Maintenance` | Operations / Maintenance | Entire repository | Read and write access |
| `COM_Prj{number}_Leader` | Project leader | Corresponding project repository | Repository administrator; manages branches and permissions |
| `COM_Prj{number}_Member` | Project member | Corresponding project repository | Read/write access; MAY create `feature/*` branches and create PRs to `dev` |

Examples:

```text
COM_Prj1_Leader
COM_Prj1_Member
```

A company/project-specific prefix MAY be used as shown in the source example:

```text
Bithumb_Prj1_Leader
Bithumb_Prj1_Member
```

## 3.2 Permission Rules

1. Global repository permissions and project-specific permissions MUST be treated separately.
2. `Member` MUST receive read permission by default.
3. `Member` write permission MUST be granted only after request/approval.
4. Project permission group names SHOULD follow this pattern:

```text
COM_Prj{project-number}_{Role}
```

5. Project numbers are used to distinguish enhancement/affiliate projects.
6. A new project MUST receive a new project number.

Examples:

```text
Prj2
Prj3
```

---

# 4. Branch Strategy

## 4.1 Branch Types

| Branch | Purpose |
|---|---|
| `main` | Stable version whose production deployment has been completed |
| `dev` | Development integration branch before production deployment |
| `feature/*` | Individual or feature-level development branch derived from `dev` |

## 4.2 Branch Hierarchy

```text
feature/* -> dev -> main
```

Example:

```text
feature/login   ┐
                ├──> dev ───> main
feature/payment ┘
```

## 4.3 Branch Rules

1. A new feature branch MUST be created from `dev`.
2. Feature branches MUST follow the naming pattern:

```text
feature/<feature-name>
```

Examples:

```text
feature/login
feature/payment
```

3. Completed feature work MUST be submitted to `dev` through a Pull Request.
4. A feature PR MUST be approved by a part leader or project leader before merging into `dev`.
5. Production changes MUST flow from `dev` to `main`.
6. Changes MUST NOT be merged directly to `main` while bypassing `dev`.

---

# 5. Development Workflow

AI agents MUST follow this sequence for normal feature development:

```text
1. Start from dev
2. Create feature/<feature-name>
3. Implement the feature
4. Commit changes
5. Create a PR: feature/* -> dev
6. Obtain required approval
7. Merge into dev
8. When deploying to production, merge dev -> main
```

Equivalent flow:

```text
dev
 └─ feature/<feature-name>
       └─ Pull Request
            └─ dev
                 └─ production release
                      └─ main
```

---

# 6. Commit Message Rules

## 6.1 Commit Message Format

Commit messages MUST use the following structure:

```text
<type>(<scope>): <subject>

<body>
```

`scope` is optional.

Therefore, both forms are valid:

```text
<type>(<scope>): <subject>
```

```text
<type>: <subject>
```

## 6.2 Example

```text
docs(README): README 추가

- 프로젝트 시작 관련 내용 추가
- 의존성 패키지 설치
- 개발 모드 실행
- 프로젝트 빌드 실행
```

---

# 7. Commit Types

AI agents MUST choose the most appropriate type from the following list.

| Type | Meaning |
|---|---|
| `feat` | Add a new feature |
| `fix` | Fix a bug |
| `docs` | Add, modify, or delete documentation such as README |
| `style` | Change code style without changing business logic |
| `refactor` | Improve code structure without changing behavior |
| `test` | Add or modify test code |
| `chore` | Build, package, configuration, or maintenance work |
| `migrate` | Migrate from the source/original system to a business project |
| `noop` | No-operation / trivial change with no logical impact, such as typo or UI color change |

## 7.1 Type Selection Guidance

Use:

```text
feat
```

when the change introduces new behavior.

Use:

```text
fix
```

when the change corrects erroneous behavior.

Use:

```text
docs
```

when only documentation is changed.

Use:

```text
style
```

when code formatting/style changes but business logic does not.

Use:

```text
refactor
```

when code structure changes but functionality remains unchanged.

Use:

```text
test
```

for test-related changes.

Use:

```text
chore
```

for build, dependency/package, configuration, or maintenance tasks.

Use:

```text
migrate
```

for source/original-system to business-project migration work.

Use:

```text
noop
```

for simple changes with no logical impact.

---

# 8. Commit Scope Rules

## 8.1 Purpose

`scope` identifies the code area or functional module affected by the commit.

## 8.2 Allowed Scope Forms

The scope SHOULD be specific.

Examples include:

- class name
- function name
- module name
- domain name

Example:

```text
feat(AuthService): 로그인 토큰 갱신 기능 추가
```

## 8.3 Multiple Scopes

Multiple scopes MUST be separated by commas.

Example:

```text
refactor(AuthService,TokenProvider): 인증 구조 개선
```

## 8.4 Optional Scope

The scope MAY be omitted.

Example:

```text
fix: 로그인 오류 수정
```

---

# 9. Commit Subject Rules

The commit subject MUST satisfy all of the following:

1. It MUST summarize the change concisely.
2. It MUST be 50 characters or fewer.
3. It MUST use imperative present tense.
4. It MUST NOT end with a period (`.`).

Example:

```text
feat(AuthService): 로그인 토큰 갱신 기능 추가
```

---

# 10. Commit Body Rules

The commit body is optional, but SHOULD be written when additional context helps reviewers.

The body SHOULD describe:

- why the change was made
- how behavior differs from the previous implementation

Rules:

1. A blank line MUST exist between the subject and body.
2. Body text SHOULD be written in imperative present tense.
3. Body lines SHOULD be 75 characters or fewer.

Example:

```text
fix(LoginService): 세션 만료 처리 수정

- 만료된 세션이 재사용되지 않도록 검증 추가
- 기존 로그인 흐름과의 차이점 정리
```

---

# 11. Commit Granularity

1. Commits SHOULD be small.
2. One commit SHOULD represent one meaningful change.
3. The subject SHOULD make the purpose of the change understandable without reading the diff.
4. The body SHOULD include the reason for the change when that context helps the reviewer.

Preferred model:

```text
1 commit = 1 meaningful change
```

---

# 12. Pull Request Rules

## 12.1 PR Title

A PR title MUST use the same format as a commit message:

```text
<type>(<scope>): <subject>
```

Example:

```text
feat(AuthService): 로그인 토큰 갱신 기능 추가
```

---

## 12.2 PR Description

A PR description MUST include:

1. Change summary
2. Major changed files and affected scope
3. Test method or execution method

Recommended template:

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

---

# 13. Reviewer Rules

## 13.1 Reviewer Is Mandatory

Every PR MUST have at least one reviewer assigned.

Minimum requirement:

```text
reviewer count >= 1
```

## 13.2 Reviewer Priority

Reviewer selection SHOULD prioritize:

1. Person responsible for the affected feature/module
2. Part leader

## 13.3 Reviewer Selection Rules

- If the PR affects a specific feature/module, assign the responsible owner.
- If the impact scope is broad, include the team leader.
- For an urgent fix, assign all relevant stakeholders.

AI agents MUST NOT create or prepare a PR under the assumption that reviewer assignment can be omitted.

---

# 14. Merge Rules

1. A PR MUST receive at least one approval before merge.
2. The author MUST NOT self-merge their own PR.
3. `main` MUST receive changes through `dev`.
4. Direct feature-to-main flow is prohibited by the documented workflow.

Required flow:

```text
feature/* -> dev -> main
```

Prohibited normal flow:

```text
feature/* -> main
```

---

# 15. AI Agent Operational Rules

When an AI coding agent performs Git-related work, it MUST apply the following decision process.

## 15.1 Before Starting Work

1. Identify the target feature or change.
2. Confirm that the base branch is `dev`.
3. Choose an appropriate feature branch name:

```text
feature/<feature-name>
```

4. Keep the planned work limited to one meaningful change where practical.

## 15.2 Before Committing

The AI MUST verify:

```text
[ ] Commit type is valid
[ ] Scope is specific or intentionally omitted
[ ] Subject is concise
[ ] Subject is <= 50 characters
[ ] Subject does not end with "."
[ ] Subject uses imperative present tense
[ ] Body is separated by one blank line when present
[ ] Body explains reason/difference when useful
[ ] Commit contains one meaningful change
```

## 15.3 Before Creating a PR

The AI MUST verify:

```text
[ ] PR target branch is dev for feature work
[ ] PR title follows commit-message format
[ ] Change summary is included
[ ] Major changed files / impact scope are included
[ ] Test or execution method is included
[ ] At least one reviewer is assigned
```

## 15.4 Before Merge

The AI MUST verify:

```text
[ ] At least one approval exists
[ ] PR author is not self-merging
[ ] Feature work merges into dev
[ ] Production merge follows dev -> main
```

If any mandatory condition is not satisfied, the AI MUST stop and report the unmet rule instead of proceeding with the merge.

---

# 16. Machine-Readable Summary

```yaml
git_policy:
  permissions:
    Leader:
      repository_scope: all
      role: administrator
      can_approve_pr: true
    Member:
      repository_scope: all
      default_access: read
      write_access: approval_required
    Maintenance:
      repository_scope: all
      access:
        - read
        - write
    project_groups:
      leader_pattern: "COM_Prj{number}_Leader"
      member_pattern: "COM_Prj{number}_Member"
      project_member:
        can_create_feature_branch: true
        can_create_pr_to_dev: true

  branches:
    main:
      purpose: production_stable
    dev:
      purpose: development_integration
    feature:
      pattern: "feature/*"
      base_branch: dev

  workflow:
    normal_flow:
      - feature/*
      - dev
      - main
    feature_pr_target: dev
    production_merge:
      from: dev
      to: main

  commit:
    format: "<type>(<scope>): <subject>"
    scope_optional: true
    subject:
      max_length: 50
      imperative_present: true
      trailing_period: false
    body:
      optional: true
      recommended: true
      blank_line_after_subject: true
      max_line_length: 75
      describe:
        - reason
        - behavioral_difference
    atomic_commit_recommended: true

  commit_types:
    - feat
    - fix
    - docs
    - style
    - refactor
    - test
    - chore
    - migrate
    - noop

  pull_request:
    title_format: "<type>(<scope>): <subject>"
    required_description:
      - change_summary
      - major_changed_files_and_impact
      - test_or_execution_method
    reviewer:
      required: true
      minimum: 1
    merge:
      minimum_approvals: 1
      self_merge_allowed: false
      main_must_merge_from_dev: true
```

---

# 17. Quick Reference

```text
Branch:
  feature/* -> dev -> main

Feature branch:
  feature/<feature-name>

Commit:
  <type>(<scope>): <subject>

Types:
  feat | fix | docs | style | refactor | test | chore | migrate | noop

Subject:
  <= 50 chars
  imperative present tense
  no trailing period

Body:
  optional but recommended
  blank line after subject
  reason / previous behavior difference
  <= 75 chars per line

PR:
  same title format as commit
  include summary
  include changed files / impact
  include test or execution method
  assign >= 1 reviewer

Merge:
  >= 1 approval
  no self-merge
  main only through dev
```
