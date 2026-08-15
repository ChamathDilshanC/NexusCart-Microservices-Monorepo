# Deployment Guide: Multi-Vendor E-Commerce Platform

This document provides a step-by-step guide to provisioning infrastructure and deploying the application to Microsoft Azure.

## 1. Prerequisites
Before deploying, ensure you have the following installed and configured:
- **Azure CLI:** Installed and authenticated (`az login`).
- **Terraform:** Installed (v1.5.0 or later recommended).
- **GitHub Account:** For repository hosting and CI/CD automation.
- **Environment Variables:** Prepared `.env` files for local testing (Do not commit these).

## 2. Infrastructure Provisioning (Terraform)
We use Terraform to automate the creation of Azure resources.

1. **Navigate to the infra directory:**
   ```bash
   cd infra

```

2. **Initialize Terraform:**
This downloads the required providers.
```bash
terraform init

```


3. **Plan the Deployment:**
Review the changes that will be applied to your Azure subscription.
```bash
terraform plan

```


4. **Apply Configuration:**
Create the resources (App Service, Cosmos DB, etc.) in Azure.
```bash
terraform apply
# Type 'yes' when prompted

```



## 3. Configuring Application Environment Variables

Once the infrastructure is live, you must inject your application secrets into the Azure App Service.

1. **Go to Azure Portal:** Navigate to your newly created App Service.
2. **Configuration:** Click on "Configuration" in the left sidebar.
3. **Application Settings:** Add the required environment variables:
* `MONGODB_URI`
* `GOOGLE_CLIENT_ID`
* `GOOGLE_CLIENT_SECRET`
* `NEXTAUTH_SECRET`
* `EMAIL_USER`
* `EMAIL_PASS`


4. **Save:** Click "Save" to restart the service with the new configuration.

## 4. Automating Deployment (CI/CD with GitHub Actions)

To enable automated deployments on every `git push`, create a file at `.github/workflows/deploy.yml`:

```yaml
name: Deploy E-Commerce App

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      - name: Install and Build
        run: |
          cd frontend && npm install && npm run build
      - name: Deploy to Azure
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'your-app-name-here'
          publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE }}
          package: ./frontend

```

## 5. Post-Deployment & Maintenance

* **Domain Mapping:** Link your custom domain via the Azure App Service "Custom Domains" section.
* **Monitoring:** Use Azure Monitor and Application Insights to track system health and errors.

## 6. Important: Cleanup (Free Tier Safety)

If you are using the Azure Free Tier for development and testing, **always destroy the infrastructure** when not in use to prevent unexpected billing.

```bash
# Navigate to the infra directory
cd infra
# Run the destroy command
terraform destroy
# Type 'yes' to confirm
