# GitHub Actions Deployment Setup Guide

This guide will help you configure automated deployments to Cloudflare Workers using GitHub Actions.

## Prerequisites

1. A Cloudflare account with:
   - Workers plan ($5/month minimum for Durable Objects)
   - **OR** Workers for Platforms enabled (for full container features)

2. A Cloudflare API Token with the following permissions:
   - Account: Workers Scripts (Edit)
   - Account: Workers KV Storage (Edit)
   - Account: Workers R2 Storage (Edit)
   - Account: D1 (Edit)
   - Account: Durable Objects (Edit)
   - Account: AI Gateway (Edit) - Optional
   - Account: Workers for Platforms (Edit) - Optional, if using containers

## Step 1: Create Cloudflare API Token

1. Go to https://dash.cloudflare.com/profile/api-tokens
2. Click "Create Token"
3. Use the "Edit Cloudflare Workers" template or create a custom token with the permissions listed above
4. Copy the token - you'll need it in the next step

## Step 2: Get Your Cloudflare Account ID

1. Go to https://dash.cloudflare.com
2. Select any website (or go to Workers & Pages)
3. Scroll down on the right sidebar to find your "Account ID"
4. Copy the Account ID

## Step 3: Configure GitHub Secrets

Go to your GitHub repository settings: `Settings > Secrets and variables > Actions`

### Required Secrets (Click "New repository secret" for each)

| Secret Name | Description | How to Get |
|-------------|-------------|------------|
| `CLOUDFLARE_API_TOKEN` | Your Cloudflare API token | From Step 1 |
| `CLOUDFLARE_ACCOUNT_ID` | Your Cloudflare account ID | From Step 2 |
| `JWT_SECRET` | Random string for session management | Generate: `openssl rand -base64 32` |
| `WEBHOOK_SECRET` | Random string for webhooks | Generate: `openssl rand -base64 32` |

### Optional Secrets (for AI providers)

| Secret Name | Description | Required For |
|-------------|-------------|--------------|
| `ANTHROPIC_API_KEY` | Anthropic Claude API key | Using Claude models |
| `OPENAI_API_KEY` | OpenAI API key | Using GPT models |
| `GOOGLE_AI_STUDIO_API_KEY` | Google AI Studio API key | Using Gemini models |
| `OPENROUTER_API_KEY` | OpenRouter API key | Using OpenRouter |
| `GROQ_API_KEY` | Groq API key | Using Groq models |

### Optional Secrets (for OAuth)

| Secret Name | Description | Required For |
|-------------|-------------|--------------|
| `GOOGLE_CLIENT_SECRET` | Google OAuth client secret | Google login |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID | Google login |
| `GITHUB_CLIENT_ID` | GitHub OAuth app client ID | GitHub login |
| `GITHUB_CLIENT_SECRET` | GitHub OAuth app client secret | GitHub login |

### Optional Secrets (for AI Gateway)

| Secret Name | Description |
|-------------|-------------|
| `CLOUDFLARE_AI_GATEWAY_TOKEN` | AI Gateway authentication token |
| `CLOUDFLARE_AI_GATEWAY_URL` | Custom AI Gateway URL (if not using default) |

## Step 4: Configure GitHub Variables

Go to: `Settings > Secrets and variables > Actions > Variables` tab

| Variable Name | Default Value | Description |
|---------------|---------------|-------------|
| `CUSTOM_DOMAIN` | (empty) | Your custom domain (e.g., `build.cloudflare.dev`) |
| `CUSTOM_PREVIEW_DOMAIN` | (empty) | Preview domain for user apps (optional) |
| `TEMPLATES_REPOSITORY` | `https://github.com/cloudflare/vibesdk-templates` | Templates repository URL |
| `CLOUDFLARE_AI_GATEWAY` | `vibesdk-gateway` | AI Gateway name |
| `MAX_SANDBOX_INSTANCES` | `10` | Maximum container instances (if using Workers for Platforms) |
| `SANDBOX_INSTANCE_TYPE` | `standard` | Container type: `standard` or `enhanced` |
| `DISPATCH_NAMESPACE` | `vibesdk-default-namespace` | Workers for Platforms namespace name |

## Step 5: Trigger Deployment

### Automatic Deployment

The workflow will automatically deploy when:
- You push to the `main` branch
- A pull request is merged to `main`

### Manual Deployment

1. Go to `Actions` tab in your repository
2. Select "Deploy to Cloudflare Workers" workflow
3. Click "Run workflow"
4. Select environment (production/staging)
5. Click "Run workflow"

## Step 6: Verify Deployment

After the workflow completes:

1. Check the Actions run for any errors
2. Visit your custom domain (if configured) or the workers.dev URL shown in deployment logs
3. Verify the application loads correctly

## Troubleshooting

### Deployment Fails with "Dispatch namespaces not available"

This is **normal** if you don't have Workers for Platforms enabled. The deployment script will automatically:
- Skip dispatch namespace setup
- Comment out dispatch_namespaces in wrangler.jsonc
- Continue with deployment

The app will deploy but container features won't work. See `FINDINGS_REPORT.md` for details.

### Deployment Fails with "Containers not supported"

Containers require **Workers for Platforms** (enterprise feature). Options:

1. **Contact Cloudflare sales** to enable Workers for Platforms
2. **Modify the code** to remove container dependencies (see `FINDINGS_REPORT.md` Section 5)
3. **Use external services** for code execution (see `FINDINGS_REPORT.md` Section 8.2)

### Database Migration Errors

If you see migration errors:
1. Ensure the D1 database exists in your Cloudflare account
2. Check the database ID in `wrangler.jsonc` matches your D1 database
3. Run migrations manually: `npm run db:migrate:remote`

### Secret Upload Errors

If secrets fail to upload:
1. Verify your API token has "Workers Scripts" edit permission
2. Check that all required secrets are set in GitHub
3. Ensure secret values don't contain invalid characters

## Advanced Configuration

### Custom Domain Setup

1. Add your domain to Cloudflare
2. Set the `CUSTOM_DOMAIN` variable to your domain
3. The deployment script will automatically:
   - Configure routes in wrangler.jsonc
   - Detect the zone ID
   - Set up custom domain routing

### Multiple Environments

To deploy to staging/production:

1. Create environment-specific secrets:
   - `STAGING_CLOUDFLARE_API_TOKEN`
   - `PRODUCTION_CLOUDFLARE_API_TOKEN`

2. Modify the workflow to use different secrets based on the environment input

3. Use workflow dispatch to select the environment

### Container Configuration

If you have Workers for Platforms:

- `MAX_SANDBOX_INSTANCES`: Number of container instances (default: 10)
- `SANDBOX_INSTANCE_TYPE`:
  - `standard`: 4 vCPU, 4GB RAM
  - `enhanced`: 8 vCPU, 8GB RAM

The deployment script will automatically update container configuration in wrangler.jsonc.

## Monitoring Deployments

### GitHub Actions Dashboard

- View all deployments: `Actions` tab
- Each run shows:
  - Build logs
  - Migration status
  - Deployment success/failure
  - Configuration summary

### Cloudflare Dashboard

- View deployed worker: https://dash.cloudflare.com
- Navigate to: Workers & Pages > Overview
- Click on your worker to see:
  - Invocation metrics
  - Error logs
  - Resource usage

## Security Best Practices

1. **Never commit secrets** to the repository
2. **Rotate API tokens** regularly (every 90 days)
3. **Use least-privilege permissions** for API tokens
4. **Enable 2FA** on your Cloudflare account
5. **Review deployment logs** for sensitive data exposure

## Getting Help

- **Deployment Issues**: Check `FINDINGS_REPORT.md` for compatibility analysis
- **GitHub Actions**: Review workflow logs in Actions tab
- **Cloudflare Errors**: Check Cloudflare dashboard > Workers > Logs
- **Container Issues**: Verify Workers for Platforms is enabled

## Next Steps

After successful deployment:

1. ✅ Test the deployed application
2. ✅ Configure custom domain (if not done)
3. ✅ Set up monitoring and alerts
4. ✅ Review `FINDINGS_REPORT.md` for optimization opportunities
5. ✅ Consider enabling Workers for Platforms for full functionality

---

**Need Help?** See `FINDINGS_REPORT.md` for detailed analysis and recommendations.
