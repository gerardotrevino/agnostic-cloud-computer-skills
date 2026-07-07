# Git Conventions Reference

## Commit Message Format

Use Conventional Commits format:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | When to Use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only changes |
| `style` | Formatting, missing semicolons (no logic change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Build system or external dependencies |
| `ci` | CI configuration changes |
| `chore` | Maintenance tasks |

### Rules

- Subject line: imperative mood, lowercase, no period, max 72 characters
- Body: explain what and why (not how), wrap at 72 characters
- Footer: reference issues (`Closes #123`), breaking changes (`BREAKING CHANGE:`)

## Branching Strategy

| Branch | Purpose | Merges Into |
|--------|---------|-------------|
| `main` | Production-ready code | — |
| `develop` | Integration branch (optional) | `main` |
| `feat/{name}` | Feature development | `develop` or `main` |
| `fix/{name}` | Bug fixes | `develop` or `main` |
| `hotfix/{name}` | Production emergency fixes | `main` directly |

For small teams or solo projects, trunk-based development (commit directly to `main`) is acceptable with CI gates.

## Pull Request Workflow

1. Create feature branch from `main` (or `develop`)
2. Make atomic commits with clear messages
3. Push and open PR with description template
4. CI pipeline runs automatically (must pass)
5. Code review (human or automated)
6. Squash merge into target branch
7. Delete feature branch

### PR Description Template

```markdown
## What
Brief description of the change.

## Why
Context and motivation.

## How
Implementation approach.

## Testing
How this was tested.

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
- [ ] Security implications considered
```

## Release Process

| Step | Action |
|------|--------|
| 1. Tag | Create semantic version tag (`v1.2.3`) |
| 2. Changelog | Generate from conventional commits |
| 3. Build | CI builds release artifacts |
| 4. Deploy | Automated deployment to staging → production |
| 5. Verify | Health checks and smoke tests |
| 6. Announce | Update changelog, notify stakeholders |

## Semantic Versioning

`MAJOR.MINOR.PATCH` — MAJOR for breaking changes, MINOR for new features, PATCH for bug fixes. Pre-release: `1.0.0-beta.1`. Build metadata: `1.0.0+build.123`.
