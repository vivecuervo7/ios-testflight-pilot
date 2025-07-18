# iOS TestFlight Pilot

A demonstration iOS application showcasing a multi-environment deployment to TestFlight using GitHub Actions.

## Overview

This project serves as a learning example for implementing a complete iOS CI/CD pipeline that:

- Runs automated tests
- Builds and deploys to multiple environments (Dev → Staging → Prod)
- Uses secure code signing with certificates and provisioning profiles
- Uploads builds to TestFlight via App Store Connect API
- Implements manual approval gates for production deployments

## Deployment Pipeline

### Workflow Overview

The deployment pipeline follows a sequential approach with manual approval gates:

```
Tests → Dev → Staging → Prod
         ↓       ↓        ↓
    Automatic Manual  Manual
```

### Environment Configuration

Each environment has its own:

- **Xcode Scheme**: `Pilot Dev`, `Pilot Staging`, `Pilot Prod`
- **Provisioning Profile**: Environment-specific profiles for code signing
- **GitHub Environment**: With optional protection rules for manual approvals

### Pipeline Steps

1. **Test Job** (`test`)

   - Runs on iOS Simulator
   - Executes unit tests using `Pilot Dev` scheme
   - Must pass before any deployments begin

2. **Dev Deployment** (`deploy-dev`)

   - Automatically deploys after tests pass
   - Uses development provisioning profile
   - Uploads to TestFlight with "Dev" tag

3. **Staging Deployment** (`deploy-staging`)

   - Requires Dev deployment to complete
   - Can be configured for manual approval
   - Uses staging provisioning profile

4. **Production Deployment** (`deploy-prod`)
   - Requires Staging deployment to complete
   - Should be configured for manual approval
   - Uses production provisioning profile

## GitHub Actions Architecture

### Composite Actions

The pipeline uses modular composite actions for better maintainability:

- **`setup-ios-build-environment`**: Sets up keychain, certificates, and Xcode
- **`build-and-upload-ios-app`**: Handles provisioning, build, export, and TestFlight upload
- **`cleanup-ios-build-environment`**: Cleans up temporary keychains
- **`deploy-ios-app-to-testflight`**: Orchestrates the complete deployment pipeline

## Required Secrets

Configure these secrets in your GitHub repository and environments:

### Repository Secrets

Configure these secrets at the repository level:

- `DISTRIBUTION_CERTIFICATE_BASE64`: Base64-encoded P12 distribution certificate
- `DISTRIBUTION_CERTIFICATE_PASSWORD`: Password for the P12 certificate
- `APP_STORE_CONNECT_API_KEY`: Content of the App Store Connect API key (.p8 file)
- `APP_STORE_CONNECT_API_KEY_ID`: API Key ID from App Store Connect
- `APP_STORE_CONNECT_ISSUER_ID`: Issuer ID from App Store Connect

### Environment Secrets

Configure these secrets within each GitHub environment (`dev`, `staging`, `prod`):

- `PROVISIONING_PROFILE_BASE64`: Base64-encoded provisioning profile specific to each environment

## Setup Instructions

### 1. Xcode Project Configuration

Ensure your Xcode project has:

- Three schemes configured: `Pilot Dev`, `Pilot Staging`, `Pilot Prod`
- Appropriate bundle identifiers for each environment
- Code signing configured for each target

### 2. App Store Connect Setup

1. Create an API key in App Store Connect
2. Note the Key ID and Issuer ID
3. Download the `.p8` key file

### 3. Certificate and Provisioning Profiles

1. Create distribution certificates in Apple Developer Portal
2. Create provisioning profiles for each environment
3. Export certificate as P12 format
4. Convert certificates and profiles to Base64:
   ```bash
   base64 -i certificate.p12 -o certificate.txt
   base64 -i profile.mobileprovision -o profile.txt
   ```

### 4. GitHub Configuration

1. **Configure GitHub Environments**: Create three environments (`dev`, `staging`, `prod`) in your repository settings
2. **Add Repository Secrets**: Configure certificate and App Store Connect API secrets at the repository level
3. **Add Environment Secrets**: For each environment, add the `PROVISIONING_PROFILE_BASE64` secret with the appropriate Base64-encoded provisioning profile for that environment:
   - **dev environment**: Add development provisioning profile
   - **staging environment**: Add staging provisioning profile
   - **prod environment**: Add production provisioning profile
4. **Configure Protection Rules**: Add manual approval requirements for staging and prod environments

## Key Learnings

This example demonstrates:

- **Sequential deployment patterns** for multi-environment releases
- **GitHub Actions composite actions** for code reuse and maintainability
- **iOS code signing automation** with certificates and provisioning profiles
- **Secure secret management** and temporary file handling
- **TestFlight automation** using App Store Connect API
- **Manual approval gates** for production deployments

## Usage

1. Push code to the `main` branch or manually trigger the workflow
2. Tests run automatically
3. Dev deployment happens automatically after tests pass
4. Staging and Prod deployments require manual approval (if configured)
5. Check TestFlight for uploaded builds
