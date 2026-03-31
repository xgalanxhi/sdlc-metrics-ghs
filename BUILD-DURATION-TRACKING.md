# Build Duration Tracking in CloudBees Unify

## Native Unify Workflows vs GitHub Actions

### Unify Native Workflows
In native CloudBees Unify workflows, you specify `kind: build` on a specific step:


```yaml
steps:
  - name: Build Application
    kind: build  # ← Marks this specific step as the build
    run: ./build.sh
```

Unify then measures **only that step's duration** for build time metrics.

### GitHub Actions Integration

In GitHub Actions, the equivalent is **using the `register-build-artifact` action**:

```yaml
jobs:
  build:  # ← The entire job is considered the "build"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Application
        run: npm run build

      - name: Register Build Artifact
        uses: cloudbees-io-gha/register-build-artifact@v3  # ← Marks this job as a build
        with:
          name: my-app
          version: 1.0.0
          type: npm-package
```

## Key Differences

| Aspect | Native Unify Workflow | GitHub Actions |
|--------|----------------------|----------------|
| **Granularity** | Step-level (single step) | Job-level (entire job) |
| **How to mark** | `kind: build` on step | Use `register-build-artifact` action |
| **Duration measured** | Individual step | Entire job (checkout → register) |
| **Precision** | High (exact build step) | Lower (includes setup/checkout) |

## Build Duration Calculation

### What's Included in GitHub Actions Build Time

When CloudBees Unify calculates build duration for GitHub Actions, it measures the entire job containing `register-build-artifact`:

```yaml
jobs:
  build:
    steps:
      - Checkout code          # ← Included in build time
      - Setup Node.js          # ← Included in build time
      - Install dependencies   # ← Included in build time
      - Build application      # ← Included in build time ✓ (actual build)
      - Create package         # ← Included in build time
      - Register artifact      # ← Included in build time
```

### To Get More Accurate Build-Only Timing

If you need to track **only** the actual build step duration (excluding setup), you have a few options:

#### Option 1: Separate Jobs
Split setup and build into separate jobs:

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      # Upload node_modules as artifact

  build:  # ← This job duration = build time
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build  # Only the build command
      - uses: cloudbees-io-gha/register-build-artifact@v3
```

**Pros**: Most accurate build time
**Cons**: More complex, slower overall (separate jobs)

#### Option 2: Custom Timing with Evidence
Track build timing manually and include in evidence:

```yaml
- name: Build application
  id: build
  run: |
    BUILD_START=$(date -u +%s)
    npm run build
    BUILD_END=$(date -u +%s)
    BUILD_DURATION=$((BUILD_END - BUILD_START))
    echo "build_duration=$BUILD_DURATION" >> $GITHUB_OUTPUT
    echo "Build took $BUILD_DURATION seconds"

- name: Register Build Artifact
  uses: cloudbees-io-gha/register-build-artifact@v3
  with:
    name: my-app
    version: 1.0.0

- name: Publish Build Duration Evidence
  uses: cloudbees-io-gha/publish-evidence-item@v2
  with:
    content: |
      # Build Metrics
      **Actual Build Duration**: ${{ steps.build.outputs.build_duration }}s
      **Total Job Duration**: Job-level timing
```

**Pros**: No workflow restructuring
**Cons**: Build duration not in main dashboard, only in evidence

#### Option 3: Accept Job-Level Timing
For most cases, **job-level timing is sufficient** because:
- Setup/checkout time is relatively consistent
- Trends over time still show build performance changes
- Simpler workflow structure

## Current Implementation

This repository uses **Option 3** (job-level timing) with build timing captured for reference:

- The `build` job contains `register-build-artifact@v3`
- CloudBees Unify measures the entire job duration
- Build step timing is captured in outputs but not sent to Unify
- This is the **recommended approach** for GitHub Actions

## Recommendations

1. **For most projects**: Use job-level timing (current implementation)
2. **For large projects with long setup**: Consider Option 1 (separate jobs)
3. **For precise metrics**: Use Option 2 (custom evidence) in addition to job-level

## Viewing Build Duration in Unify

In CloudBees Unify dashboard:
1. Navigate to **Analytics** > **Build Performance**
2. View the "Successful build duration" chart
3. The duration shown is the entire `build` job from the workflow
4. Filter by component to see trends over time

## Note on Optimization

If build times are consistently high, check what's included:
- Is checkout/setup slow? (Consider caching)
- Is npm ci taking long? (Use `cache: 'npm'` in setup-node)
- Is the actual build slow? (Optimize build process)

Run times for current workflow:
- Checkout: ~5s
- Setup Node: ~10s (with cache)
- Install deps: ~20s (with cache)
- **Actual build**: ~30s ← Core build time
- Create package: ~5s
- Register artifact: ~2s
- **Total job**: ~72s ← What Unify measures

---

**Summary**: For GitHub Actions, `register-build-artifact` is the equivalent of `kind: build`, but it marks the entire job, not a single step.
