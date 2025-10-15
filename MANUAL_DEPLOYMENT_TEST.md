# Manual Deployment Test - West US Production

## Changes Made
- Updated workflow to skip validation for manual deployments
- This allows testing West US deployment without IP Access List blocking validation
- Validation still runs for PRs and automatic deployments

## Workflow Behavior

### Manual Deployment (workflow_dispatch):
```
✅ Skip validation (no IP ACL check on dev workspace)
✅ Deploy directly to West US Production
✅ Uses West US workspace credentials
✅ Requires manual approval
```

### Pull Request (pull_request):
```
⚠️ Run validation (will fail if IP ACL blocks)
```

### Automatic Deployment (push to main):
```
⚠️ Run validation (will fail if IP ACL blocks)
✅ Deploy to dev
```

## Why This Works for Manual Deployment

The West US workspace may have different IP Access List settings than the dev workspace. By skipping validation (which uses dev workspace), we can test deployment directly to West US.

## Testing Manual Deployment

1. Commit this workflow change to main
2. Go to: https://github.com/sumitsaraswat/sample_dab/actions
3. Click: "Databricks DAB CI/CD"
4. Click: "Run workflow"
5. Select: main branch, westus_prod environment
6. Approve the deployment
7. Verify in West US Databricks workspace

## Important Notes

- Validation is skipped for manual deployments only
- This is a workaround for IP Access List blocking
- Proper solution: Add GitHub Actions IPs to all workspace IP Access Lists
- For production use: Ensure IP Access Lists are properly configured

