---
date modified: 2026-01-17
parent: n45-portal
slug: security-and-access-control
status: Live
subtitle: The "Security." Details on authentication, user whitelisting, and secrets
  management.
tags: null
title: Showcase - N45.io - Security and Access Control
visibility: public
---

# N45.dev Portal - Security and Access Control

---

This document outlines the security architecture of the n45.dev portal, focusing on authentication, authorization, and secrets management.

## 1. Authentication

Authentication is the process of verifying a user's identity. The portal uses **Google OAuth 2.0** as its sole authentication provider, managed via the NextAuth.js library.

### Authentication Flow

1.  **User Action:** The user clicks the "Login" button on the frontend.
2.  **Redirect to Google:** The Next.js application redirects the user to Google's OAuth consent screen.
3.  **Google Authentication:** The user authenticates with their Google account and grants the application permission to view their email address and basic profile information.
4.  **Callback to Application:** Google redirects the user back to the application's callback URL (`/api/auth/callback/google`) with an authorization code.
5.  **Session Creation:** NextAuth.js securely exchanges the authorization code for an access token and user profile information. It then creates a session for the user, storing it in a secure, HTTP-only cookie.

This flow ensures that the application never handles user passwords directly, significantly reducing security risks.

## 2. Authorization

Authorization is the process of determining whether an authenticated user has permission to access a specific resource. The portal operates on a **strict whitelist model**.

### Whitelist Implementation

-   **Database Table:** A dedicated table in the Supabase PostgreSQL database named `approved_users` stores the email addresses of all individuals authorized to access the secure area.
-   **Access Check:**
    1.  When a user successfully authenticates via Google, the application takes their session email address.
    2.  It then performs a lookup against the `approved_users` table.
    3.  **If a matching email is found,** the user is granted access to the `/secure` routes.
    4.  **If no match is found,** the user is denied access, even though they are successfully authenticated with Google.

This model ensures that only pre-approved individuals can access sensitive or non-public content, providing a robust security boundary.

## 3. Secrets Management

Secrets are sensitive pieces of information like API keys, database credentials, and OAuth client secrets. These must be managed securely to prevent unauthorized access.

### Environment Variables

-   All secrets are managed as **environment variables**. They are never hard-coded into the source code.
-   A local `.env.local` file is used for development, which is explicitly excluded from version control via `.gitignore`.
-   This ensures that no sensitive credentials are ever committed to the GitHub repository.

### Production Secrets

-   **Vercel:** All production environment variables for the Next.js application (Supabase keys, Google OAuth secrets, etc.) are securely stored in the **Vercel Project Settings**. Vercel encrypts these variables and only exposes them to the build and runtime environments.
-   **GitHub Actions:** The `SYNC_SECRET` and `YOUTUBE_API_KEY`, which are used by the daily cron job, are stored as **Encrypted Secrets** in the GitHub repository settings. These secrets are only accessible to the GitHub Actions runner during a workflow run.

### List of Secrets

| Secret Name                   | Used In           | Purpose                                              |
| ----------------------------- | ----------------- | ---------------------------------------------------- |
| `NEXT_PUBLIC_SUPABASE_URL`    | Vercel / Local    | Public URL for the Supabase project.                 |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Vercel / Local    | Public (anonymous) key for the Supabase API.         |
| `GOOGLE_CLIENT_ID`            | Vercel / Local    | Client ID for Google OAuth 2.0.                      |
| `GOOGLE_CLIENT_SECRET`        | Vercel / Local    | Client Secret for Google OAuth 2.0.                  |
| `NEXTAUTH_SECRET`             | Vercel / Local    | A secret key to encrypt the NextAuth.js session JWT. |
| `NEXTAUTH_URL`                | Vercel / Local    | The canonical URL of the application.                |
| `YOUTUBE_API_KEY`             | Vercel / Local    | API key for accessing the YouTube Data API.          |
| `SYNC_SECRET`                 | Vercel / GitHub   | A shared secret to authenticate the cron job request.|