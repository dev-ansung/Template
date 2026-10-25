# Contributing Guidelines

## 1. Branching Strategy and Worktree Isolation

| Branch Type | Naming Format | PR Permitted | Requirements |
| --- | --- | --- | --- |
| Feature / Bugfix | `feat/<issue-id>-<slug>` or `fix/<issue-id>-<slug>` | Yes | Adheres to all quality gates and rebase checks. |
| Experimental | `exp/<identifier>/<slug>` | No | Bypasses commit and lint rules. Direct PRs to main are blocked. |

### Branch Rules
* All feature branches must branch directly from upstream `main`.
* A pull request must be rebased onto the latest `main` with all conflicts resolved prior to review. Merge commits are prohibited.
* One pull request is permitted per branch.
* Experimental branches (`exp/*`) exist purely for prototyping and spikes. They must never be merged directly into `main`. Once an experimental solution is validated, recreate the minimal, verified changes in a clean feature branch originating from `main`.

### Worktree Isolation Pattern
To support concurrent development without cross-branch contamination, lockfile conflicts, or uncommitted file overwrites, all active development should run inside isolated Git worktrees rather than switching branches in the root checkout.

#### Setup a Worktree
Create a dedicated sibling directory for the branch off latest `main`:
```bash
git fetch origin main
git worktree add -b <branch-name> ../worktrees/<sanitized-branch-name> origin/main
cd ../worktrees/<sanitized-branch-name>

```

#### Worktree Invariants

* The primary root repository directory should remain untouched on `main`.
* Build outputs, local dependency directories, and runtime caches must remain scoped to the individual worktree.
* When working concurrently, configure dynamic environment variables (such as local ports) to avoid runtime collisions across active worktrees.

#### Teardown a Worktree

Once a branch is merged, pushed, or abandoned, return to the main repository to remove the worktree and prune metadata:

```bash
cd <repo-root>
git worktree remove ../worktrees/<sanitized-branch-name>
git worktree prune

```

---

## 2. Issue, PR, and Commit Standardization

### Issue Titles

Use the format:

```text
[<TYPE>-<SCOPE>] <Short descriptive title>

```

* Allowed types: `FEAT`, `FIX`, `REFACTOR`, `PERF`, `DEBT`, `SPIKE`
* Example: `[FIX-AUTH] Handle session timeout during token refresh`

### Pull Request Titles

Use the format:

```text
<type>(<scope>): <summary> (resolves #<issue-id>)

```

* Example: `fix(auth): handle session timeout during token refresh (resolves #142)`

### Commit Messages

Commits must follow the Conventional Commits specification:

```text
<type>(<scope>): <short summary in imperative mood>

[optional body: explain what and why]
[optional footer: issue references]

```

* Allowed types: `feat`, `fix`, `refactor`, `perf`, `test`, `build`, `ci`, `docs`, `chore`
* A feature branch may contain multiple commits during development.

---

## 3. Scope Boundaries and Discovered Issues

* Pull requests must remain strictly limited to the scope of the assigned issue.
* If bugs, technical debt, or architectural issues are discovered outside the immediate task:
1. Do not modify the out-of-scope code within the current branch or worktree.
2. Open a separate issue using the standardized issue format.
3. Reference the newly opened issue in the pull request description under Discovered Issues.



---

## 4. Local Verification and Diff Justification

### Pre-Commit Gates

Before committing code locally inside the worktree, the following commands must execute cleanly:

* Project linter and code formatter
* Project build / compile target
* Relevant test suites

### Line-of-Code (LOC) and Architecture Justification

Every pull request must justify net additions to keep the codebase decoupled and clean:

* Summarize total lines added and removed.
* Detail why introduced abstractions, helper utilities, or third-party dependencies are strictly necessary instead of reusing existing repository functionality.

---

## 5. Pull Request Template

Every pull request must include the following structure:

```markdown
### Summary
<!-- High-level description of changes -->

### LOC and Architectural Justification
- Net Lines: +X / -Y
- Architectural Rationale: <!-- Explain why new code/abstractions were introduced rather than reusing existing utilities -->

### Scope and Discovered Issues
- Resolves: #<issue-id>
- Discovered Issues Logged:
  - [ ] #<issue-id> - <Issue title>

### Pre-Submission Checklist
- [ ] Developed and verified inside an isolated worktree.
- [ ] Rebased onto latest origin/main with conflicts resolved.
- [ ] Linter, build, and tests pass locally prior to commit.
- [ ] PR title adheres to standardized format.
- [ ] Branch contains only one logical feature or fix.

```

---

## 6. Continuous Integration

Pull requests require clean status passes before approval and merge:

* Automated linting and formatting check
* Clean compilation and build check
* Complete automated test suite execution
* Rebase check confirming linear history against `main`