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

### Step-by-Step: Creating an Azure AD App Registration

#### Part 1: Create the App Registration

1. **Open Azure Portal**
   - Navigate to [https://portal.azure.com](https://portal.azure.com)
   - Sign in with an account that has admin rights

2. **Navigate to App Registrations**
   - In the search bar at the top, type "App registrations"
   - Click on **App registrations** under Services
   - Or navigate manually: **Azure Active Directory** → **App registrations**

3. **Create New Registration**
   - Click **+ New registration** button at the top
   - Fill in the registration form:
     - **Name**: `PowerApps-CodeApp-Deploy`
     - **Supported account types**: Select **"Accounts in this organizational directory only (Single tenant)"**
     - **Redirect URI**: Leave empty (not needed)
   - Click **Register** button at the bottom

4. **Save Important Information**
   - You'll be redirected to the app's Overview page
   - **IMPORTANT**: Copy and save these values immediately:
     - **Application (client) ID** → This is your `CLIENT_ID`
       - Example format: `12345678-1234-1234-1234-123456789012`
     - **Directory (tenant) ID** → This is your `TENANT_ID`
       - Example format: `87654321-4321-4321-4321-210987654321`
   - Keep these values in a safe place (you'll need them for GitHub Secrets)

#### Part 2: Create Client Secret

1. **Navigate to Certificates & Secrets**
   - In the left sidebar, click **Certificates & secrets**
   - Click on the **Client secrets** tab

2. **Create New Secret**
   - Click **+ New client secret**
   - Fill in the form:
     - **Description**: `GitHub Actions Deploy`
     - **Expires**: Choose **24 months** (or as per your security policy)
   - Click **Add**

3. **Copy Secret Value**
   - **⚠️ CRITICAL**: The secret value is shown only ONCE!
   - In the table, you'll see two columns:
     - **Value**: This is your `CLIENT_SECRET` - COPY THIS NOW
     - **Secret ID**: You don't need this
   - **Copy the Value column** and save it securely
   - If you miss this step, you'll need to create a new secret

#### Part 3: Configure API Permissions

1. **Navigate to API Permissions**
   - In the left sidebar, click **API permissions**
   - You'll see **User.Read** permission already there (this is fine)

2. **Add Dynamics CRM Permission**
   - Click **+ Add a permission**
   - Scroll down and select **Dynamics CRM**
   - Select **Delegated permissions**
   - Check the box next to **user_impersonation**
   - Click **Add permissions** at the bottom

3. **Grant Admin Consent** (Required!)
   - Back on the API permissions page, you'll see a warning that admin consent is required
   - Click **Grant admin consent for [Your Organization]**
   - Confirm by clicking **Yes** in the popup
   - Wait for the status to change to green checkmarks

4. **Verify Permissions**
   - You should now see these permissions with green checkmarks:
     - Microsoft Graph: `User.Read` (Status: Granted)
     - Dynamics CRM: `user_impersonation` (Status: Granted)

#### Part 4: Get Power Platform Environment ID

1. **Open Power Platform Admin Center**
   - Navigate to [https://admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)
   - Sign in with your Power Platform admin account

2. **Find Your Environment**
   - Click **Environments** in the left sidebar
   - Find the environment where you want to deploy (e.g., "Omgeving van System Administrator")
   - Click on the environment name

3. **Copy Environment ID**
   - In the environment details, look for **Environment ID**
   - Copy this value → This is your `ENVIRONMENT_ID`
   - Example format: `ce7db999-716a-ec38-8300-2268acc2a749`

4. **Verify Code Apps are Enabled**
   - While you're here, verify Code Apps are enabled:
   - Click **Settings** at the top
   - Expand **Product** → Click **Features**
   - Scroll down to find **Power Apps code apps**
   - Ensure the toggle is **ON** (if not, enable it and save)

### Step-by-Step: Add Secrets to GitHub

Now that you have all four values, let's add them to GitHub:

1. **Navigate to Repository Settings**
   - Go to [https://github.com/MPParsley/hello-code-apps](https://github.com/MPParsley/hello-code-apps)
   - Click on the **Settings** tab (⚙️ icon, far right in the menu)
   - **Note**: If you don't see Settings, you may not have admin access to the repository

2. **Navigate to Secrets**
   - In the left sidebar, find **Security** section
   - Click **Secrets and variables**
   - Click **Actions**

3. **Add Each Secret**

   For each of the four secrets, repeat these steps:

   **Secret 1: TENANT_ID**
   - Click **New repository secret** (green button)
   - Name: `TENANT_ID`
   - Secret: Paste your Directory (tenant) ID from Azure
     - Example: `f77a4382-8389-4b40-9570-59524b2094b3`
   - Click **Add secret**

   **Secret 2: CLIENT_ID**
   - Click **New repository secret**
   - Name: `CLIENT_ID`
   - Secret: Paste your Application (client) ID from Azure
   - Click **Add secret**

   **Secret 3: CLIENT_SECRET**
   - Click **New repository secret**
   - Name: `CLIENT_SECRET`
   - Secret: Paste the secret Value you copied from Certificates & secrets
   - Click **Add secret**

   **Secret 4: ENVIRONMENT_ID**
   - Click **New repository secret**
   - Name: `ENVIRONMENT_ID`
   - Secret: Paste your Power Platform Environment ID
     - Example: `ce7db999-716a-ec38-8300-2268acc2a749`
   - Click **Add secret**

4. **Verify All Secrets**
   - After adding all four, you should see them listed:
     - `CLIENT_ID`
     - `CLIENT_SECRET`
     - `ENVIRONMENT_ID`
     - `TENANT_ID`
   - **Note**: You cannot view the secret values after creation (only update or delete)

### Testing the Setup

1. **Trigger a Manual Deployment**
   - Go to the **Actions** tab in your repository
   - Click on **Deploy Power Apps Code App** workflow
   - Click **Run workflow** dropdown (right side)
   - Select **Branch: main**
   - Click **Run workflow** button

2. **Monitor the Deployment**
   - Click on the running workflow to see details
   - Watch each step execute:
     - ✅ Checkout code
     - ✅ Setup Node.js
     - ✅ Install dependencies
     - ✅ Build application
     - ✅ Setup .NET
     - ✅ Install Power Platform CLI
     - ✅ Authenticate to Power Platform
     - ✅ Select Environment
     - ✅ Deploy Code App
     - ✅ Upload build artifacts

3. **Verify Success**
   - If all steps show green checkmarks ✅, deployment succeeded!
   - The workflow output will show the app URL
   - You can access your app at: `https://apps.powerapps.com/play/e/{environment-id}/app/{app-id}`

### Common Setup Issues

#### Issue: "Grant admin consent" button is grayed out
- **Cause**: You don't have admin rights
- **Solution**: Contact your Azure AD administrator to grant consent

#### Issue: Cannot see "Settings" tab in GitHub
- **Cause**: You don't have admin access to the repository
- **Solution**: Repository owner needs to grant you admin access

#### Issue: Deployment fails with authentication error
- **Cause**: Incorrect secret values
- **Solution**:
  1. Verify you copied the correct values from Azure
  2. Client secret: Make sure you copied the **Value** column, not the Secret ID
  3. IDs should be in GUID format (8-4-4-4-12 characters)
  4. Re-create secrets in GitHub if needed

#### Issue: "Environment does not allow this operation"
- **Cause**: Code Apps not enabled in environment
- **Solution**: See "Verify Code Apps are Enabled" section above

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
