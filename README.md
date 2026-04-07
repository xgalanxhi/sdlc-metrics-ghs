# CloudBees Unify SDLC Metrics - GitHub Actions Demo

This repository demonstrates CloudBees Unify's metrics and analytics capabilities when integrated with GitHub Actions workflows. It showcases how to collect and visualize DORA metrics, deployment analytics, and software delivery insights across your CI/CD pipeline.
 
## Overview
 
This demo application validates the CloudBees Unify metrics functionality, specifically focusing on:

- **DORA Metrics Dashboard**: Deployment frequency, lead time, change failure rate, and MTTR
- **Deployment Analytics**: Component summary, deployment overview, and activity tracking
- **Artifact Traceability**: Version history with deployment details across environments
- **Environment Inventory**: Track artifact versions promoted across dev, staging, and production
- **Test Results Publishing**: Unit test results and coverage metrics
- **Security Scanning**: CodeQL for code vulnerabilities + Gitleaks for secret detection
- **Secret Management**: Automated test secrets workflow for validation

## Architecture

### Application
- **Language**: Node.js 18
- **Framework**: Express.js
- **Testing**: Jest with coverage reporting
- **Linting**: ESLint
- **Security**: GitHub CodeQL scanning + Gitleaks secret detection
  

### CI/CD Pipeline
The workflow (`.github/workflows/ci-cd.yml`) includes:

1. **Test Job**: Runs unit tests and publishes results to CloudBees Unify
2. **Lint Job**: Code quality checks with ESLint
3. **CodeQL Scan**: GitHub-native security scanning (no self-hosted runner required)
4. **Gitleaks Scan**: Secret detection scanning to identify exposed credentials
5. **Build-Test Job**: Creates artifacts and registers them in CloudBees Unify
6. **Deploy Jobs**: Simulates deployments to dev/staging/production environments

### Environments
- **Development**: Auto-deploys from `develop` branch
- **Staging**: Auto-deploys from `main` branch (with smoke tests)
- **Production**: Manual approval required, deploys from `main` after staging

## Prerequisites

Before using this repository, ensure you have:

1. **CloudBees Unify Account**: Access to a CloudBees Unify instance
2. **GitHub Repository**: This repository pushed to GitHub
3. **Jira Board** (Optional): For linking work items to deployments
4. **GitHub Actions**: Enabled on your repository

## Setup Instructions

### 1. CloudBees Unify Configuration

#### Create Environments in Unify
Before running workflows, create the following environments in CloudBees Unify:

```yaml
- development (non-production)
- staging (non-production)
- production (production)
```

You can use the `cloudbees-environments.yml` file as a reference for environment configuration.

### 2. GitHub Repository Configuration

#### Configure GitHub Environments
1. Go to **Settings** > **Environments**
2. Create three environments:
   - `development`: No protection rules
   - `staging`: No protection rules (or add required reviewers if desired)
   - `production`: Add required reviewers and enable "Wait timer" if desired

### 3. Initial Workflow Run

1. Push this code to your GitHub repository
2. Create a `develop` branch: `git checkout -b develop && git push -u origin develop`
3. Make a commit to `develop` to trigger the workflow
4. The workflow will:
   - Run tests and publish results to CloudBees Unify
   - Perform linting checks
   - Run CodeQL security scan
   - Run Gitleaks secret detection scan
   - Build and register the artifact
   - Deploy to development environment

## Validating Metrics in CloudBees Unify

### 1. Test Results
Navigate to **Analytics** > **Test Results** in CloudBees Unify:
- View test execution trends
- See test coverage metrics
- Identify flaky tests
- Track test duration over time

### 2. Build Artifacts
Go to **Components** > **Your Component**:
- See all registered artifact versions
- View build evidence for each version
- Check artifact labels and metadata
- Trace artifacts to source commits

### 3. Deployment Tracking
View **Environment Inventory**:
- See which versions are deployed to each environment
- Track deployment history
- View artifact progression through environments

### 4. DORA Metrics Dashboard
Navigate to **Analytics** > **DORA Metrics**:

**Deployment Frequency**
- Track how often deployments occur to each environment
- View trends over time (daily, weekly, monthly)
- Compare frequency across different components

**Lead Time for Changes**
- Measure time from commit to production deployment
- Identify bottlenecks in your delivery pipeline
- Track improvement over time

**Change Failure Rate**
- See percentage of deployments that cause failures
- Monitor quality trends
- Compare rates across environments

**Mean Time to Recovery (MTTR)**
- Track how quickly issues are resolved
- View recovery time trends
- Set targets and monitor progress

### 5. Deployment Analytics
Access additional deployment insights:
- **Component Summary**: Overview of deployment activity
- **Software Delivery Activity**: Team velocity and throughput
- **Code Progression**: Successful deployments tracking
- **Average Deployment Time**: Performance metrics

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── ci-cd.yml              # Main CI/CD pipeline
│       └── manage-test-secrets.yml # Automated test secrets management
├── .cloudbees/
│   └── workflows/
│       └── ci-cd.yaml             # CloudBees native workflow (equivalent)
├── src/
│   ├── index.js                   # Express application
│   └── utils.js                   # Utility functions
├── tests/
│   ├── api.test.js               # API endpoint tests
│   └── utils.test.js             # Utility function tests
├── cloudbees-environments.yml    # Environment configuration reference
├── package.json                  # Node.js dependencies and scripts
├── .eslintrc.js                  # ESLint configuration
├── .env.example                  # Environment variables template
├── BUILD-DURATION-TRACKING.md   # Build metrics documentation
├── CLOUDBEES-NATIVE-WORKFLOW.md # Native workflow documentation
└── README.md                     # This file
```

## Workflow Jobs Explained

### Test Job
- Runs Jest unit tests with coverage
- Generates JUnit XML report
- Publishes test results to CloudBees Unify using `cloudbees-io-gha/publish-test-results@v2`

### Lint Job
- Runs ESLint on source code
- Enforces code quality standards
- Continues on error to not block pipeline

### CodeQL Scan Job
- GitHub-native security scanning
- Analyzes JavaScript code for vulnerabilities
- No self-hosted runner required
- Results visible in GitHub Security tab

### Gitleaks Scan Job
- Secret detection scanning using Gitleaks
- Identifies exposed credentials in code and history
- Scans entire git history with `fetch-depth: 0`
- Publishes findings to CloudBees Unify

### Build-Test Job
- Generates semantic version from git history
- Creates production build
- Packages artifact as tar.gz
- Registers artifact in CloudBees Unify using `cloudbees-io-gha/register-build-artifact@v3`
- Adds labels for filtering and organization
- Publishes build evidence

### Deploy Jobs
- Simulates deployment to respective environments (customize for real deployments)
- **Note**: Deployment registration is currently disabled (action not yet released)
- Includes smoke tests for staging environment
- Production requires manual approval via GitHub Environments

## Key CloudBees Actions Used

### `cloudbees-io-gha/publish-test-results@v2`
Publishes test results to CloudBees Unify for test analytics.

```yaml
- uses: cloudbees-io-gha/publish-test-results@v2
  with:
    test-type: junit
    results-path: junit.xml
```

### `cloudbees-io-gha/register-build-artifact@v3`
Registers build artifacts for traceability.

```yaml
- uses: cloudbees-io-gha/register-build-artifact@v3
  with:
    name: ${{ github.repository }}
    url: https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}
    version: ${{ steps.version.outputs.version }}
    type: npm-package
```


### `cloudbees-io-gha/gitleaks-scan-publish@v1`
Scans repository for exposed secrets and credentials.

```yaml
- uses: cloudbees-io-gha/gitleaks-scan-publish@v1
  # Optional: with cloudbees-pat for publishing to Unify
```

### `cloudbees-io-gha/label-artifact-version@v1`
Adds labels to artifacts for filtering and organization.

```yaml
- uses: cloudbees-io-gha/label-artifact-version@v1
  with:
    artifact-id: ${{ steps.register-artifact.outputs.cbp_artifact_id }}
    labels: "main,nodejs,express,automated"
```

### `cloudbees-io-gha/publish-evidence-item@v2`
Publishes build and deployment evidence.

```yaml
- uses: cloudbees-io-gha/publish-evidence-item@v2
  with:
    content: |-
      # Build Evidence Report
      ...
```

**Note**: `register-deployed-artifact` action is currently disabled in the workflow as it's not yet released. Deployment tracking will be enabled when the action becomes available.

## Additional Workflows

### Manage Test Secrets Workflow
The repository includes a utility workflow (`manage-test-secrets.yml`) for testing secret detection:

**Purpose**: Creates or deletes a test file (`env.test`) containing fake credentials for Gitleaks validation

**Usage**:
1. Go to **Actions** tab in GitHub
2. Select **"Manage Test Secrets File"** workflow
3. Click **"Run workflow"**
4. Choose action: **create** or **delete**

**Features**:
- Fake credentials are base64 encoded in the workflow to avoid detection
- Decodes and writes plaintext secrets to `env.test` file for Gitleaks to detect
- Automatically commits and pushes changes
- Useful for validating secret scanning integration

## Customization

### Add More Environments
1. Add environment definition in `cloudbees-environments.yml`
2. Create environment in CloudBees Unify
3. Add new deploy job in `.github/workflows/ci-cd.yml`
4. Configure environment in GitHub repository settings

### Integrate with Jira
Add Jira issue keys to commits to link deployments to work items:

```bash
git commit -m "PROJ-123: Add new feature"
```

CloudBees Unify will automatically associate deployments with Jira issues.

### Use Different CI Providers
The same CloudBees actions are available for:
- Jenkins/CBCI (pipeline steps)
- Other CI systems (API-based integration)

## Resources

- [CloudBees Unify Documentation](https://docs.cloudbees.com/docs/cloudbees-unify/latest/)
- [DORA Metrics Guide](https://dora.dev/guides/dora-metrics-four-keys/)
- [CloudBees GitHub Actions](https://github.com/cloudbees-io-gha)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Support

For issues related to:
- **This demo repository**: Open an issue in this repo
- **CloudBees Unify**: Contact CloudBees Support
- **GitHub Actions**: Refer to GitHub documentation

## License

MIT License - See LICENSE file for details


---


**Note**: This is a demonstration repository for testing CloudBees Unify metrics capabilities. Customize the deployment steps to match your actual deployment targets and processes.
