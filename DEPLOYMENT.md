# Deployment Guide

This repository contains a Power Apps Code App with automatic deployment via GitHub Actions.

## Prerequisites

- Power Platform environment with Code Apps enabled
- Azure AD App Registration for authentication
- GitHub repository secrets configured

## Setup GitHub Secrets

To enable automatic deployment, you need to configure the following secrets in your GitHub repository:

### Required Secrets

1. **TENANT_ID**: Your Microsoft 365 Tenant ID
2. **CLIENT_ID**: Application (client) ID from Azure AD App Registration
3. **CLIENT_SECRET**: Client secret from Azure AD App Registration
4. **ENVIRONMENT_ID**: Power Platform Environment ID (e.g., `ce7db999-716a-ec38-8300-2268acc2a749`)

### Creating an Azure AD App Registration

1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to **Azure Active Directory > App registrations > New registration**
3. Name: `PowerApps-CodeApp-Deploy`
4. Supported account types: **Accounts in this organizational directory only**
5. Click **Register**

### Configure App Registration

1. **Client Secret**:
   - Go to **Certificates & secrets**
   - Click **New client secret**
   - Add description: `GitHub Actions`
   - Choose expiration period
   - Copy the **Value** (this is your `CLIENT_SECRET`)

2. **API Permissions**:
   - Go to **API permissions**
   - Click **Add a permission**
   - Select **Dynamics CRM**
   - Check **user_impersonation**
   - Click **Grant admin consent**

3. **Copy IDs**:
   - From the Overview page, copy:
     - **Application (client) ID** → `CLIENT_ID`
     - **Directory (tenant) ID** → `TENANT_ID`

### Add Secrets to GitHub

1. Go to your GitHub repository
2. Navigate to **Settings > Secrets and variables > Actions**
3. Click **New repository secret**
4. Add each of the four secrets:
   - `TENANT_ID`
   - `CLIENT_ID`
   - `CLIENT_SECRET`
   - `ENVIRONMENT_ID`

## Manual Deployment

To deploy manually from your local machine:

```bash
# Install dependencies
npm install

# Build the app
npm run build

# Authenticate (if not already authenticated)
pac auth create

# Select environment
pac env select --environment YOUR_ENVIRONMENT_ID

# Deploy
pac code push
```

## Automatic Deployment

Once secrets are configured, the app will automatically deploy on:
- Every push to the `main` branch
- Manual trigger via **Actions** tab → **Deploy Power Apps Code App** → **Run workflow**

## Current Environment

- **App ID**: `9d1cc3ce-f6e5-42b6-be80-81a071098f98`
- **Display Name**: Hello World Code App
- **Environment**: Omgeving van System Administrator
- **Environment ID**: `ce7db999-716a-ec38-8300-2268acc2a749`

## Local Development

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Access locally at:
# http://localhost:5173/

# Test in Power Apps:
# https://apps.powerapps.com/play/e/YOUR_ENV_ID/a/local?_localAppUrl=http://localhost:5173/
```

## Troubleshooting

### Code Apps Not Enabled

If deployment fails with "environment does not allow this operation", enable Code Apps:

1. Go to [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/)
2. Select your environment
3. Go to **Settings > Product > Features**
4. Enable **"Power Apps code apps"**
5. Save

### Authentication Failed

- Verify all secrets are correctly configured
- Ensure the App Registration has the correct API permissions
- Check that admin consent has been granted

### Build Errors

- Ensure Node.js version matches requirements (22.x)
- Delete `node_modules` and `package-lock.json`, then run `npm install` again
