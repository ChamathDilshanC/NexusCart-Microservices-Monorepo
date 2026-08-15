# Multi-Vendor E-Commerce Platform: Complete Architecture & Guide

## 1. Overview
This document outlines the comprehensive architecture, technology stack, module breakdown, and deployment strategy for a scalable multi-vendor e-commerce platform. The system supports three primary user roles:
1. **Platform Admin:** Manages the overall platform, user base, and vendor application approvals.
2. **Business Vendor:** Manages individual storefronts, product catalogs, and vendor-specific orders.
3. **Customer:** Browses products, registers accounts, manages carts, and completes purchases.

## 2. Technical Stack
- **Frontend:** Next.js (App Router), Tailwind CSS, Lucide Icons, Zustand / React Context.
- **Backend:** Node.js (Express.js) + TypeScript.
- **Database:** MongoDB (Managed via Mongoose / Azure Cosmos DB).
- **Authentication:** NextAuth.js (Google OAuth) + Local Auth (JWT & bcrypt).
- **Email Service:** Nodemailer using Gmail SMTP for OTP verification.
- **Infrastructure & DevOps:** Terraform (Azure Provider), Azure App Service, Azure Static Web Apps, GitHub Actions (CI/CD).

## 3. Core Modules & Features

### A. Authentication & User Management
- **Dual Flow:**
  - **Local Auth:** Email and password registration requiring OTP-based email verification.
  - **Social Auth:** Seamless login via Google OAuth.
- **Email Verification Workflow:**
  1. User submits registration details.
  2. System generates a secure 6-digit one-time password (OTP).
  3. OTP is hashed and saved in the database with a 10-minute expiration.
  4. Node.js dispatches the verification code via Gmail SMTP.
  5. Upon validation, the account status updates to `isVerified: true`.

### B. Business Registry Module
- **Vendor Onboarding:** Businesses submit registration data (Business Name, Address, Registration Number, Contact Information).
- **Admin Approval Workflow:** Submitted vendor accounts remain pending until reviewed and approved by a Platform Admin.
- **Storefront Access:** Approved vendors gain exclusive access to a dedicated business dashboard.

### C. Admin Panel
- **User Management:** View, search, and manage platform users or restrict access.
- **Vendor Management:** Review and approve or reject business registry applications.
- **System Metrics:** Centralized dashboard tracking total orders, revenue, and active vendors.

## 4. Project Directory Structure & Git Submodules
This project uses Git Submodules to maintain the `frontend/` and `backend/` repositories as independent modules within this monorepo structure.

```text
e-commerce-platform/          <-- Main Repository
├── backend/                  <-- Git Submodule (Independent Repo)
├── frontend/                 <-- Git Submodule (Independent Repo)
├── infra/                    <-- Terraform Configuration
├── .github/                  <-- CI/CD Workflows
└── README.md


## Project Directory Structure
```text
e-commerce-platform/
├── backend/
│   ├── src/
│   │   ├── controllers/      # Auth, Business, Product, Admin controllers
│   │   ├── models/           # Mongoose schemas (User, Business, Product, Order, VerificationCode)
│   │   ├── routes/           # Express API route definitions
│   │   ├── middleware/       # RBAC, Auth guards, Admin checks
│   │   ├── utils/            # Email service, Token generators
│   │   └── app.js            # Express application entry point
│   ├── package.json
│   |── .env
|   |___README.md
├── frontend/
│   ├── src/
│   │   ├── app/              # Next.js App Router structure
│   │   │   ├── (auth)/       # Login, Register, Verify pages
│   │   │   ├── (admin)/      # Admin dashboard & vendor approvals
│   │   │   ├── (business)/   # Store setup & product management
│   │   │   └── shop/         # Public catalog and checkout
│   │   ├── components/       # Shared UI elements (Tailwind styled)
│   │   └── services/         # API client configurations (Axios/Fetch)
│   ├── tailwind.config.ts
│   └── package.json
|   |___README.md
├── infra/
│   ├── main.tf               # Terraform configuration for Azure resources
│   ├── variables.tf          # Terraform variables
│   └── outputs.tf            # Terraform outputs
├── docker-compose.yml        # Local development container orchestration
└── README.md
