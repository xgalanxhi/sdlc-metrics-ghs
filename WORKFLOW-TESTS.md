# Workflow Test Plan for CloudBees Unify Bug

These test workflows help isolate what's preventing ci-cd.yml from showing in CloudBees Unify's control plane.

## Test Workflows Created

### 1. `simple-test.yml` - Baseline
**Tests:** Minimal workflow with single job
- Single job, no dependencies
- Simple steps only
- No environments, outputs, or complex features

**Expected:** Should show in Unify (if this doesn't show, integration is broken)

---

### 2. `test-multi-job.yml` - Job Dependencies
**Tests:** Multiple jobs with `needs:` dependencies
- 2 jobs with dependency chain
- No environments or outputs
- 

**Expected:** If this fails but simple-test works, issue is with job dependencies

---

### 3. `test-with-env.yml` - GitHub Environments
**Tests:** Workflow using GitHub environment feature
- Single job with `environment:` block
- Includes environment URL

**Expected:** If this fails but simple-test works, issue is with environment configuration

---

### 4. `test-with-outputs.yml` - Job Outputs
**Tests:** Jobs passing data via outputs
- Job with outputs defined
- Second job consuming those outputs

**Expected:** If this fails but simple-test works, issue is with job outputs

---

### 5. `test-cloudbees-actions.yml` - CloudBees Actions
**Tests:** Using CloudBees GitHub Actions
- Uses `cloudbees-io-gha/publish-test-results@v2`
- Includes OIDC permissions

**Expected:** If this fails but simple-test works, issue is with CloudBees actions themselves

---

## How to Test

1. **Commit and push these workflows:**
   ```bash
   git add .github/workflows/
   git commit -m "Add test workflows for Unify debugging"
   git push
   ```

2. **Trigger each workflow manually** in GitHub Actions (workflow_dispatch)

3. **Check CloudBees Unify UI** after each run:
   - Look for the workflow name in the CI Workflows list
   - Note which ones appear and which don't

4. **Report results:**
   - ✅ = Shows in Unify
   - ❌ = Doesn't show in Unify

   | Workflow | Shows in Unify? | Notes |
   |----------|----------------|--------|
   | ant.yaml | ✅ | Working baseline |
   | simple-test.yml | ? | |
   | test-multi-job.yml | ? | |
   | test-with-env.yml | ? | |
   | test-with-outputs.yml | ? | |
   | test-cloudbees-actions.yml | ? | |
   | ci-cd.yml | ❌ | Original broken workflow |

---

## Analysis Guide

**If only simple-test.yml works:**
- Issue is with workflow complexity
- Narrow down which specific feature breaks it

**If all tests work except ci-cd.yml:**
- Issue is with combination of features in ci-cd.yml
- Try removing features one by one from ci-cd.yml

**If none work (including simple-test.yml):**
- GitHub integration is broken/not configured
- Check Unify connection settings

**If all work including ci-cd.yml:**
- Cache issue - Unify might need refresh
- Check workflow filters in Unify UI

---

## Next Steps

Based on results, we can:
1. Identify the exact feature causing the bug
2. Create minimal reproduction for CloudBees support
3. Find workaround by modifying ci-cd.yml
4. Report bug to CloudBees with specific reproduction steps
