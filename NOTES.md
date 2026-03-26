# Implementation Notes

## CloudBees Actions Status


### Confirmed Available Actions
These actions are used in the workflow and are known to exist:

- ✅ `cloudbees-io-gha/publish-test-results@v2` - Publish test results
- ✅ `cloudbees-io-gha/register-build-artifact@v3` - Register build artifacts
- ✅ `cloudbees-io-gha/label-artifact-version@v1` - Add labels to artifacts
- ✅ `cloudbees-io-gha/publish-evidence-item@v2` - Publish evidence
- ✅ `cloudbees-io-gha/register-deployed-artifact` - Register deployments (no version tag)

### Deployment Registration Action

The workflow uses `cloudbees-io-gha/register-deployed-artifact` (no version tag) for registering deployments to enable DORA metrics.

#### Important Parameters
- `artifact-id`: The artifact ID from the build job output
- `target-environment`: Must match environment names in CloudBees Unify (not `environment`)
- `labels`: Optional labels for filtering/organization

#### Alternative: API-Based Approach
If no GitHub Action is available yet, you can register deployments via API:

```yaml
- name: Register Deployment via API
  run: |
    curl -X POST https://api.cloudbees.com/v1/deployments \
      -H "Authorization: Bearer ${{ secrets.CLOUDBEES_API_TOKEN }}" \
      -H "Content-Type: application/json" \
      -d '{
        "artifact_id": "${{ needs.build.outputs.artifact_id }}",
        "environment": "production",
        "version": "${{ needs.build.outputs.version }}",
        "timestamp": "${{ github.event.head_commit.timestamp }}",
        "commit_sha": "${{ github.sha }}",
        "workflow_run_id": "${{ github.run_id }}"
      }'
```

## Testing Without CloudBees Unify

If you don't have CloudBees Unify access yet, you can:

1. **Comment out CloudBees actions** in the workflow
2. **Test the core CI/CD flow**:
   - Tests will still run
   - CodeQL will still scan
   - Artifacts will be created
   - Deployments will be simulated

3. **Add CloudBees actions later** when access is available

## Scanner Options

The workflow uses CodeQL which is GitHub-native and free. Other supported scanners for CloudBees Unify:

### Available (No Self-Hosted Runner Required)
- ✅ **CodeQL** (GitHub native) - Currently implemented
- ✅ **Snyk** - `snyk/actions/node@master`
- ✅ **Trivy** - `aquasecurity/trivy-action@master`

### Requires Self-Hosted Runner
- **Black Duck** - `cloudbees-io-gha/black-duck-scan-publish@v2`
- **SonarQube** - Requires connection to SonarQube server

### To Add Additional Scanners
Add a new job in `.github/workflows/ci-cd.yml`:

```yaml
trivy-scan:
  name: Trivy Container Scan
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'
    - uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'
```

## Environment Configuration

### Current Setup
- Environments defined in `cloudbees-environments.yml`
- Environments created manually in CloudBees Unify (Phase 1)
- GitHub environments configured for deployment protection

### Future Phases (Roadmap)
- **Phase 2**: Automatic environment creation from pipelines
- **Phase 2**: Auto-detection of deployment events
- **Phase 2**: GHA deployment event webhook subscriptions

## DORA Metrics Implementation Notes

### What's Being Tracked
✅ **Deployment Frequency**: Every `register-deployed-artifact` call
✅ **Lead Time**: From commit timestamp to deployment timestamp
✅ **Change Failure Rate**: Workflow failures after deployment
✅ **MTTR**: Time between failure and successful recovery deployment

### What's NOT Tracked (Yet)
- ❌ Manual deployments outside GitHub Actions
- ❌ Deployments from other CI systems (Phase 1 will add this)
- ❌ Rollback events (may need custom implementation)

### To Simulate Different Scenarios

**Successful Deployment (Normal Flow)**
```bash
git commit -m "feat: Add new feature"
git push
# Deployment succeeds, metrics recorded
```

**Failed Deployment (High Change Failure Rate)**
```bash
# Add failing test or linter error
git commit -m "fix: Attempted fix"
git push
# Deployment fails, failure recorded
```

**Recovery Deployment (MTTR Tracking)**
```bash
# Fix the issue
git commit -m "fix: Correct the issue"
git push
# Deployment succeeds, MTTR calculated
```

## Jira Integration

To link deployments to Jira issues:

1. **Include Jira issue key in commit message**:
   ```bash
   git commit -m "PROJ-123: Add new feature"
   ```

2. **Configure Jira integration in CloudBees Unify**:
   - Settings > Integrations > Jira
   - Provide Jira URL and credentials
   - Map projects to components

3. **View linked work items**:
   - Deployments will show associated Jira issues
   - Jira issues will show deployment status

## Customization Checklist

- [ ] Update `APP_NAME` in workflow to match your component
- [ ] Modify deployment steps to target real infrastructure
- [ ] Configure actual smoke tests in staging deployment
- [ ] Set up notification channels (Slack, email, etc.)
- [ ] Add branch protection rules
- [ ] Configure CODEOWNERS file
- [ ] Set up monitoring and alerting
- [ ] Define SLOs for DORA metrics
- [ ] Add rollback procedures
- [ ] Document incident response process

## Performance Considerations

- **CodeQL scan**: First run ~5-10 minutes (builds cache), subsequent runs ~2-3 minutes
- **Test execution**: ~30 seconds for this demo app
- **Build time**: ~1 minute
- **Deployment**: Instant (simulated), adjust for real deployments

## Security Best Practices

- ✅ Use OIDC instead of long-lived tokens when possible
- ✅ Store secrets in GitHub Secrets, not in code
- ✅ Use environment protection rules for production
- ✅ Enable branch protection on main/develop
- ✅ Require pull request reviews
- ✅ Enable CodeQL scanning
- ✅ Keep dependencies up to date (Dependabot)

## Known Limitations

1. **Simulated Deployments**: Current workflow simulates deployments. Replace with actual deployment logic.
2. **No Rollback Logic**: Add rollback procedures for production failures.
3. **Limited Error Handling**: Enhance error handling for production use.
4. **Single Region**: Extend for multi-region deployments if needed.

## Next Steps After Initial Setup

1. **Validate metrics are flowing** to CloudBees Unify
2. **Run multiple deployments** to build up DORA metrics history
3. **Simulate failures** to test change failure rate tracking
4. **Set DORA targets** based on your organization's goals
5. **Share dashboard** with team and stakeholders
6. **Iterate on pipeline** based on metrics insights

## Support and Resources

- **CloudBees Unify Docs**: https://docs.cloudbees.com/docs/cloudbees-unify/latest/
- **CloudBees GHA Actions**: https://github.com/cloudbees-io-gha
- **DORA Research**: https://dora.dev/
- **This Repository Issues**: For demo-specific questions

---

Last Updated: March 2026



