# GitHub Actions Workflows

## Current Workflows

### `ci.yml` - Main CI Pipeline
**Runs on:** Push to `main`, PRs to `main` and `develop`

**Jobs:**
1. **Quick Checks** - Linting and type checking (fails fast)
2. **Build & Test** - Full build and unit tests
3. **CI Summary** - Aggregates results

**Optimizations:**
- Concurrency control: Cancels previous runs when new commits are pushed
- PNPM caching: Speeds up dependency installation
- Fail-fast strategy: Quick checks run first to catch errors early
- Minimal external dependencies: No expensive third-party services

## Removed Workflows

The following workflows were removed to reduce GitHub Actions quota usage:

### Removed: `test.yml` and `test-simple.yml`
**Reason:** Duplicate workflows doing the same thing
- Both ran lint, type check, and build
- Consolidated into single `ci.yml` workflow

### Removed: `contextforge-*.yml` (export, quality, sync)
**Reason:** Referenced non-existent external CLI tool
- Referenced `@contextforge/cli` which doesn't exist in this project
- Would fail immediately on every run
- Functionality can be re-implemented as needed within the app itself

### Removed Jobs from Original Workflows:
- **Integration tests** - Using placeholder skip script
- **E2E tests** - Using placeholder skip script
- **Performance tests** - Using placeholder skip script
- **Security tests** - Using placeholder skip script
- **Accessibility tests** - Using placeholder skip script
- **Docker build tests** - Not needed for Vercel deployment
- **Snyk scanning** - Expensive third-party service
- **SonarCloud** - Expensive third-party service

## Cost Savings

### Before Optimization:
- **5 workflow files** running on push and PR
- **10+ jobs** including expensive services (Docker, Playwright, Redis)
- **~45-60 minutes** of CI time per push/PR
- **2x execution** (runs on both push and PR to same branch)

### After Optimization:
- **1 workflow file** with concurrency control
- **3 jobs** (quick-checks, build-and-test, summary)
- **~5-8 minutes** of CI time per run
- **Cancels old runs** when new commits pushed
- **~85-90% reduction in CI quota usage**

## Optional Workflows (Not Included)

If needed in the future, you can add these as separate workflow files:

### E2E Testing (Optional)
```yaml
name: E2E Tests

on:
  workflow_dispatch:  # Manual trigger only
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      # ... Playwright setup and tests
```

### Security Audit (Optional)
```yaml
name: Security Audit

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm audit --audit-level=moderate
```

## Recommendations

1. **Keep it lean**: Only add workflows when absolutely necessary
2. **Use workflow_dispatch**: For optional/expensive jobs, use manual triggers
3. **Schedule wisely**: Use cron for non-critical checks (weekly/monthly)
4. **Monitor quota**: Check Actions usage in repository Settings > Actions
5. **Use caching**: Always cache dependencies (pnpm, npm, etc.)
6. **Concurrency control**: Cancel old runs to save quota

## Testing Changes

Before pushing workflow changes:

```bash
# Validate workflow syntax locally
gh workflow view ci.yml

# Test workflow with act (local GitHub Actions runner)
act pull_request -W .github/workflows/ci.yml
```

## Troubleshooting

### Workflow not triggering?
- Check branch protection rules
- Verify workflow file syntax with `yamllint`
- Check Actions tab for error messages

### Running out of quota?
- Review Actions usage in Settings > Actions
- Reduce frequency of scheduled workflows
- Use `workflow_dispatch` for manual-only workflows
- Consider self-hosted runners for heavy workloads
