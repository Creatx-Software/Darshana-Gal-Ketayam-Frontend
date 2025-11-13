# GitHub Actions Deployment Workflow

This repository uses GitHub Actions to automatically build and deploy the Next.js application to production.

## Setup Instructions

### 1. Configure GitHub Secrets

You need to add the following secrets to your GitHub repository:

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add each of the following:

| Secret Name | Description | Example Value |
|------------|-------------|---------------|
| `SFTP_HOST` | Your SFTP server hostname or IP | `darshanagalketayam.lk` |
| `SFTP_USERNAME` | SFTP username for authentication | `your-sftp-user` |
| `SFTP_PASSWORD` | SFTP password for authentication | `your-secure-password` |

### 2. Workflow Triggers

The deployment workflow runs automatically when:

- **Push to main branch**: Any commit pushed to the `main` branch triggers deployment
- **Manual trigger**: You can manually trigger the workflow from the Actions tab

### 3. Workflow Steps

The workflow performs the following steps:

1. **Checkout code** - Retrieves the latest code from the repository
2. **Setup Node.js** - Installs Node.js 20.x with npm caching
3. **Install dependencies** - Runs `npm ci` for clean, reproducible installs
4. **Type check** - Validates TypeScript types (`npm run type-check`)
5. **Build** - Creates optimized production build (`npm run build`) - This also catches most code issues
7. **Create deployment package** - Packages necessary files:
   - `.next/` - Built Next.js application
   - `public/` - Static assets
   - `node_modules/` - Production dependencies
   - `package.json` & `package-lock.json`
   - `next.config.js`
   - `.env.production` - Production environment variables
8. **Deploy to SFTP** - Uploads files to production server
9. **Verify** - Confirms successful deployment

### 4. Deployment Location

Files are deployed to:
```
/var/www/vhosts/darshanagalketayam.lk/httpdocs/
```

### 5. Environment Variables

The following environment variables are used during build:

- `NEXT_PUBLIC_API_URL`: https://api.darshanagalketayam.lk
- `NEXT_PUBLIC_SITE_URL`: https://darshanagalketayam.lk
- `NODE_ENV`: production

### 6. Running the Application on Server

After deployment, you need to ensure your server is running the Next.js application. You have two options:

#### Option A: Using PM2 (Recommended)

SSH into your server and run:

```bash
cd /var/www/vhosts/darshanagalketayam.lk/httpdocs
npm install -g pm2
pm2 start npm --name "darshana-gal-ketayam" -- start
pm2 save
pm2 startup
```

#### Option B: Using systemd service

Create a systemd service file for automatic startup and management.

### 7. Monitoring Deployments

- View deployment status in the **Actions** tab of your GitHub repository
- Each deployment creates a detailed log of all steps
- Failed deployments will show error messages for debugging

### 8. Troubleshooting

**Deployment fails at type-check or build:**
- Fix the TypeScript errors or build issues in your code
- Push the fixes to trigger a new deployment

**SFTP connection fails:**
- Verify your SFTP credentials in GitHub Secrets
- Ensure the SFTP server is accessible
- Check firewall rules allow GitHub Actions IPs

**Build succeeds but site doesn't update:**
- Check if the Next.js server process is running on your host
- Restart the application using PM2 or your process manager
- Verify the remote path is correct

### 9. Security Best Practices

✅ Secrets are stored securely in GitHub Secrets
✅ Never commit `.env` files with real credentials
✅ Use SSH keys instead of passwords when possible
✅ Limit deployment permissions to necessary users

### 10. Future Improvements

Consider adding:
- [ ] Deploy to staging environment first
- [ ] Run automated tests before deployment
- [ ] Add deployment notifications (Slack, Discord, email)
- [ ] Implement blue-green deployment strategy
- [ ] Add rollback capabilities
- [ ] Use SSH keys instead of password authentication
