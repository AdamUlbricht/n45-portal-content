---
parent: n45-portal
visibility: public
---

# N45.dev Portal - Deployment and Operations Manual

This document provides technical details for deploying, maintaining, and troubleshooting the live `n45.io` portal. It covers hosting, CI/CD, and disaster recovery procedures.

---

## 1. System Architecture & Services

This section provides a high-level overview of the interconnected services that run the portal.

### 1.1. Key Service Reference

| Service | Purpose | Key Configurations & Secrets |
| :--- | :--- | :--- |
| **Vercel** | Hosting & Frontend | Env Vars: `SUPABASE_URL`, `GOOGLE_ID`, `AUTH_SECRET` |
| **Supabase** | Database & Whitelist | Tables: `posts`, `approved_users` |
| **GitHub** | Code & Automation | Secrets: `CRON_SECRET` |
| **Google Cloud** | Auth & YouTube API | OAuth Consent Screen, Credentials |
| **ImprovMX** | Email Forwarding | DNS Records (MX, TXT) |

### 1.2. Database Schema (`posts` table)

```sql
id (uuid)
external_link (text, unique)
source (text)
content (text)
timestamp (date)
metadata (jsonb)
```

### 1.3. Automation Schedule

-   **Trigger:** GitHub Action (`.github/workflows/daily-sync.yml`)
-   **Time:** 09:00 UTC (Daily)
-   **Action:** Pings `https://n45.io/api/sync` with a secure Bearer Token (`CRON_SECRET`).

---

## 2. Email Infrastructure

-   **Address:** `contact@n45.io`
-   **Provider:** ImprovMX (Free Tier)
-   **Mechanism:** Forwards all incoming mail to a personal address.
-   **DNS Records:**
    -   MX: `mx1.improvmx.com` (Priority 10)
    -   MX: `mx2.improvmx.com` (Priority 20)
    -   TXT: `v=spf1 include:spf.improvmx.com ~all`
-   **Management:** Log in to the [ImprovMX Dashboard](https://improvmx.com) to manage the forwarding destination.

---

## 3. Troubleshooting

### 3.1. Code Deployment Failure

-   **Symptom:** You pushed a code change (e.g., to a React component), but the live site is unchanged or broken.
-   **Fix:**
    1.  Open the [Vercel Dashboard](https://vercel.com/).
    2.  Navigate to the `n45-portal-web` project.
    3.  Review the latest deployment in the "Deployments" tab.
    4.  If it failed, click on the deployment and inspect the "Build Logs" for a specific error message.

### 3.2. Content Update Failure

-   **Symptom:** You updated a markdown file in the content repository, but the page is showing old content or returns a 404 error.
-   **Fix:** This is often a caching issue with the git submodule on Vercel. Trigger a "Redeploy" from the Vercel dashboard, ensuring the "Use existing build cache" option is disabled.

### 3.3. Social Feed Sync Failure

-   **Symptom:** The "Daily Content Sync" GitHub Action has failed.
-   **Fix:**
    1.  Check the [GitHub Actions Logs](https://github.com/YOUR_USERNAME/n45-portal-web/actions) for the `n45-portal-web` repository.
    2.  If the error is `401 Unauthorized`, the `CRON_SECRET` environment variable in Vercel does not match the `CRON_SECRET` repository secret in GitHub. Ensure they are identical.
    3.  If the error is a `YouTube API Error`, the API Quota may be exceeded for the day, or the API key managed in Google Cloud Console has expired.

---

## 4. Disaster Recovery

This procedure outlines how to rebuild the project from scratch in a catastrophic failure scenario.

1.  **Clone Code Repository:**
    ```bash
    git clone https://github.com/[your-username]/n45-portal-web.git
    ```
2.  **Clone Content Repository:**
    ```bash
    git clone https://github.com/[your-username]/n45-portal-content.git
    ```
3.  **Link Content as Submodule:**
    -   Navigate into the `n45-portal-web` directory.
    -   Run the following command to link the content folder:
        ```bash
        git submodule add ../n45-portal-content content
        ```
4.  **Set Up Environment Variables:**
    -   Create a new file named `.env.local` in the root of the `n45-portal-web` project.
    -   Log in to the Vercel project settings and copy all environment variables into this file.
5.  **Install Dependencies:**
    ```bash
    npm install
    ```
6.  **Run Development Server:**
    ```bash
    npm run dev
    ```