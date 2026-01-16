---
parent: n45-portal
visibility: public
---

# N45.dev Portal - Developer Onboarding Guide

---

This guide provides a comprehensive, step-by-step process for setting up a complete local development environment for the n45.dev portal. Following these instructions will allow a new developer to go from a clean machine to a fully functional local server.

## 1. Prerequisites

Before you begin, ensure you have the following software installed on your machine:

-   **[Git](https://git-scm.com/):** For version control and managing the repositories.
-   **[Node.js](https://nodejs.org/) (LTS version):** The JavaScript runtime environment.
-   **[pnpm](https://pnpm.io/):** The package manager used for this project. Install it with `npm install -g pnpm`.
-   **[Visual Studio Code](https://code.visualstudio.com/):** The recommended code editor.

## 2. Repository Setup

The project is split into two repositories: one for the web application code and one for the content. You must clone both and link them.

1.  **Clone the Main Application Repository:**
    ```bash
    git clone https://github.com/n45-io/n45-portal-web.git
    cd n45-portal-web
    ```

2.  **Initialize and Clone the Content Submodule:**
    The content is managed in a separate repository and linked as a Git submodule.
    ```bash
    git submodule init
    git submodule update
    ```
    This will create a `public/projects` folder and populate it with the content from the `n45-portal-content` repository.

## 3. Environment Variable Setup

The application requires a set of environment variables to connect to external services like Supabase and Google.

1.  **Create the Environment File:**
    In the root of the `n45-portal-web` directory, create a new file named `.env.local`.

2.  **Populate the `.env.local` File:**
    Copy the following template and fill in the values. See the sections below for instructions on how to get each value.

    ```env
    # Supabase Credentials
    # Get these from your Supabase project dashboard: Settings > API
    NEXT_PUBLIC_SUPABASE_URL="YOUR_SUPABASE_URL"
    NEXT_PUBLIC_SUPABASE_ANON_KEY="YOUR_SUPABASE_ANON_KEY"

    # Google OAuth Credentials
    # Get these from the Google Cloud Console: APIs & Services > Credentials
    GOOGLE_CLIENT_ID="YOUR_GOOGLE_CLIENT_ID"
    GOOGLE_CLIENT_SECRET="YOUR_GOOGLE_CLIENT_SECRET"

    # NextAuth Configuration
    # Generate a secret using: openssl rand -base64 32
    NEXTAUTH_SECRET="YOUR_GENERATED_NEXTAUTH_SECRET"
    NEXTAUTH_URL="http://localhost:3000" # Use this for local development

    # YouTube API Key (for the daily sync)
    # Get this from the Google Cloud Console: APIs & Services > Credentials
    YOUTUBE_API_KEY="YOUR_YOUTUBE_API_KEY"

    # Sync Endpoint Secret (for authenticating the cron job)
    # Generate a secure random string (e.g., using a password manager)
    SYNC_SECRET="YOUR_SECURE_RANDOM_STRING"
    ```

### How to Get Credentials:

-   **Supabase:**
    1.  Create a new project on [Supabase](https://supabase.com/).
    2.  Navigate to **Settings > API**.
    3.  Copy the **Project URL** and the **`anon` public key**.

-   **Google OAuth:**
    1.  Go to the [Google Cloud Console](https://console.cloud.google.com/).
    2.  Create a new project.
    3.  Go to **APIs & Services > Credentials**.
    4.  Create an **OAuth 2.0 Client ID**.
    5.  Set the **Authorized JavaScript origins** to `http://localhost:3000`.
    6.  Set the **Authorized redirect URIs** to `http://localhost:3000/api/auth/callback/google`.
    7.  Copy the **Client ID** and **Client Secret**.

-   **NEXTAUTH_SECRET:**
    Run the following command in your terminal to generate a secure secret:
    ```bash
    openssl rand -base64 32
    ```

## 4. Database Setup

You need to replicate the database schema in your local Supabase instance.

1.  **Go to the Supabase SQL Editor** in your project dashboard.
2.  **Create the `approved_users` table:** This table acts as a whitelist for the secure area.
    ```sql
    CREATE TABLE approved_users (
      id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
      email TEXT NOT NULL UNIQUE
    );
    -- Add your own email to the whitelist for testing
    INSERT INTO approved_users (email) VALUES ('your.email@example.com');
    ```
3.  **Create the `posts` table:** This table stores the aggregated social feed data.
    ```sql
    CREATE TABLE posts (
      id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
      source TEXT NOT NULL,
      content TEXT,
      url TEXT,
      timestamp TIMESTAMPTZ NOT NULL,
      metadata JSONB
    );
    ```

## 5. Installation and Running the Server

1.  **Install Dependencies:**
    From the root of the `n45-portal-web` directory, run:
    ```bash
    pnpm install
    ```

2.  **Run the Development Server:**
    ```bash
    pnpm run dev
    ```

3.  **Access the Application:**
    Open your web browser and navigate to [http://localhost:3000](http://localhost:3000).

You should now have a fully functional local version of the n45.dev portal running, connected to your own Supabase instance.