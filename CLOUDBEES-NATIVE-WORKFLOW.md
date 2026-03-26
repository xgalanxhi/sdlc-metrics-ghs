# CloudBees Unify Native Workflow

This document explains the CloudBees Unify native workflow (`.cloudbees/workflows/ci-cd.yaml`) and how it differs from the GitHub Actions version (`.github/workflows/ci-cd.yml`).

## Overview

The repository now contains **two equivalent workflows**:

1. **GitHub Actions** (`.github/workflows/ci-cd.yml`) - Runs on GitHub Actions with CloudBees GHA integration
2. **CloudBees Native** (`.cloudbees/workflows/ci-cd.yaml`) - Runs natively on CloudBees Unify platform

Both workflows achieve the same goals but use different syntax and execution environments.

## Key Differences

### 1. Workflow Structure

| Aspect | GitHub Actions | CloudBees Native |
|--------|----------------|------------------|
| **File format** | YAML | YAML |
| **Header** | None | `apiVersion` + `kind: workflow` |
| **Action namespace** | `uses: actions/*` or `cloudbees-io-gha/*` | `uses: cloudbees-io/*` |
| **Runner** | `runs-on: ubuntu-latest` | Implicit (CloudBees platform) |
| **Permissions** | Explicit OIDC permissions | Handled by platform |

### 2. Action Names

| Purpose | GitHub Actions | CloudBees Native |
|---------|----------------|------------------|
| Checkout | `actions/checkout@v4` | `cloudbees-io/checkout@v1` |
| Setup Node | `actions/setup-node@v4` | `cloudbees-io/configure-node@v1` |
| Test Results | `cloudbees-io-gha/publish-test-results@v2` | `cloudbees-io/publish-test-results@v1` |
| Register Artifact | `cloudbees-io-gha/register-build-artifact@v3` | `cloudbees-io/publish-artifact@v1` |
| Evidence | `cloudbees-io-gha/publish-evidence-item@v2` | `cloudbees-io/publish-evidence@v1` |
| Deployment | `cloudbees-io-gha/register-deployed-artifact` | `cloudbees-io/register-deployment@v1` |

### 3. Context Variables

| Purpose | GitHub Actions | CloudBees Native |
|---------|----------------|------------------|
| Branch name | `${{ github.ref_name }}` | `${{ cloudbees.scm.branch }}` |
| Commit SHA | `${{ github.sha }}` | `${{ cloudbees.scm.sha }}` |
| Actor | `${{ github.actor }}` | `${{ cloudbees.scm.actor }}` |
| Run ID | `${{ github.run_id }}` | `${{ cloudbees.run_id }}` |
| Repository | `${{ github.repository }}` | `${{ cloudbees.scm.repository }}` |
| Timestamp | `${{ github.event.head_commit.timestamp }}` | `${{ cloudbees.timestamp }}` |

### 4. Step Outputs

| GitHub Actions | CloudBees Native |
|----------------|------------------|
| `echo "key=value" >> $GITHUB_OUTPUT` | `echo "key=value" >> $CLOUDBEES_OUTPUTS/key` |
| `${{ steps.stepid.outputs.key }}` | `${{ steps.stepid.key }}` |

### 5. Build Duration Tracking

**GitHub Actions:**
```yaml
- name: Register Build Artifact
  uses: cloudbees-io-gha/register-build-artifact@v3
  # The entire job is marked as "build"
```

**CloudBees Native:**
```yaml
- name: Build application
  kind: build  # ← Explicitly marks this step as the build
  run: npm run build
```

The `kind: build` annotation in CloudBees native workflows provides **more accurate** build duration metrics because it marks only the actual build step, not the entire job.

### 6. Deployment Tracking

**GitHub Actions:**
```yaml
- name: Register Deployed Artifact
  uses: cloudbees-io-gha/register-deployed-artifact@latest
  with:
    artifact-id: ${{ needs.build.outputs.artifact_id }}
    target-environment: production
```

**CloudBees Native:**
```yaml
- name: Deploy to Production
  kind: deploy  # ← Marks this as a deployment step
  run: ./deploy.sh

- name: Register Deployment
  uses: cloudbees-io/register-deployment@v1
  with:
    artifact-id: ${{ needs.build.outputs.artifact_id }}
    environment: production
```

### 7. Container Execution

**GitHub Actions:**
- Uses `runs-on: ubuntu-latest` for the entire job
- Steps run directly or use actions

**CloudBees Native:**
- Can specify container per step: `uses: docker://node:18`
- Provides more flexibility for different tool versions per step

### 8. Conditionals

**GitHub Actions:**
```yaml
if: github.ref == 'refs/heads/main'
```

**CloudBees Native:**
```yaml
if: cloudbees.scm.branch == 'main'
```

## Advantages of Each Approach

### GitHub Actions with CloudBees Integration

**Pros:**
- ✅ Runs on GitHub's infrastructure (no additional setup)
- ✅ Familiar syntax for GitHub users
- ✅ Free GitHub Actions minutes for public repos
- ✅ Integrated with GitHub Security/Dependabot
- ✅ Native CodeQL scanning support
- ✅ Works with existing GitHub marketplace actions

**Cons:**
- ❌ Less precise build duration metrics (entire job vs. single step)
- ❌ Requires OIDC setup for CloudBees integration
- ❌ Two-platform architecture (GitHub + CloudBees)

### CloudBees Native Workflow

**Pros:**
- ✅ More accurate build/deploy duration metrics (`kind:` annotations)
- ✅ Single platform (everything in CloudBees)
- ✅ Unified authentication and permissions
- ✅ Better integration with CloudBees features
- ✅ Consistent syntax across CloudBees CI/CD tools
- ✅ More control over execution environment (per-step containers)

**Cons:**
- ❌ Requires CloudBees platform access
- ❌ Different syntax from GitHub Actions
- ❌ May require migration effort for existing GitHub workflows
- ❌ Less GitHub-specific integrations

## When to Use Each

### Use GitHub Actions (.github/workflows/)

- Already using GitHub for source control
- Want to leverage GitHub's free CI/CD minutes
- Need GitHub-specific features (Dependabot, CodeQL, etc.)
- Team is familiar with GitHub Actions syntax
- Want to start quickly without CloudBees platform setup

### Use CloudBees Native (.cloudbees/workflows/)

- Already have CloudBees platform infrastructure
- Need precise DORA metrics (accurate build/deploy times)
- Want unified platform for all CI/CD
- Enterprise requirements (compliance, audit, governance)
- Need advanced CloudBees features (feature flags, progressive delivery, etc.)

## Migration Path

If you want to migrate from GitHub Actions to CloudBees Native:

1. **Start with both workflows** (side-by-side comparison)
2. **Validate metrics** in CloudBees Unify from both sources
3. **Compare performance and accuracy** (especially build duration)
4. **Train team** on CloudBees native syntax
5. **Gradually migrate** one job at a time
6. **Decommission GitHub workflow** when confident

## Current Implementation

This repository includes **both workflows** to:
- Demonstrate equivalent implementations
- Allow side-by-side comparison
- Support migration scenarios
- Provide reference examples

**Choose the workflow that best fits your infrastructure and team needs.**

## Running the CloudBees Native Workflow

To use the CloudBees native workflow:

1. **Push to CloudBees Unify platform** (not GitHub)
2. CloudBees will automatically discover `.cloudbees/workflows/ci-cd.yaml`
3. Workflow will appear in CloudBees Unify UI
4. Metrics will be automatically collected with `kind:` annotations

The workflow file is included here for reference and can be used when:
- You have access to CloudBees Unify platform
- You want to run workflows natively on CloudBees infrastructure
- You need the most accurate DORA metrics possible

---

**Note**: The `.cloudbees/workflows/` directory is for CloudBees native workflows. GitHub Actions will ignore it, and CloudBees native platform will ignore `.github/workflows/`.
