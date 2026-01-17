---
date modified: 2026-01-17
parent: n45-portal
slug: content-management-guide
status: Live
subtitle: The "Content." Instructions for adding and managing projects, profile information,
  and the social feed.
tags: null
title: Showcase - N45.io - Content Management Guide
type: documentation
visibility: public
---

# N45.dev Portal - Content Management Guide

This guide provides instructions for managing all user-facing content on the `n45.io` portal, including the project showcase, professional profile, social feed, and user access.

---

## 1. Professional Profile & Resume

The content for your main profile/resume page is sourced directly from a single note in your Obsidian vault.

1.  **Locate and Edit:** Open the `[[Professional_Profile]]` note in Obsidian.
2.  **Make Changes:** Update the content as needed.
3.  **Deploy:** Run the `deploy.bat` script located in the project's root directory. This will push the content changes to the live server.
4.  **Verification:** The website will reflect the changes within approximately 2 minutes.

---

## 2. Project Showcase

Adding a new project to the `/showcase` page involves creating a new markdown file with specific frontmatter.

1.  **Create a New Note:** Create a new markdown file in your Obsidian vault.
2.  **Filename Convention:** The filename **must** use `kebab-case` (e.g., `my-awesome-project.md`). This directly translates to the URL slug.
3.  **Add Frontmatter:** Add the following YAML block to the very top of the note. This metadata is crucial for how the project is displayed.

    ```yaml
    ---
    title: "Project Title"
    subtitle: "A brief, catchy subtitle"
    image: "/images/projects/your-cover-image.png" # Must be in the /public/images/projects folder
    slug: "project-title" # Must match the filename without .md
    date: YYYY-MM-DD
    status: "Active"  # Options: Active, Finished, Archived
    visibility: "public" # Or "private" to hide it
    ---
    ```
4.  **Add Content:** Write the body of your project description below the frontmatter.
5.  **Deploy:** Run the `deploy.bat` script.
6.  **Result:** A new project card will appear on the `/showcase` page, and a detailed page will be available at `/showcase/project-title`.

---

## 3. Social Media Feed

The social feed on the homepage is populated both automatically and manually.

### 3.1. Automatic Posting (YouTube)

-   **Action:** Simply upload a new video to your configured YouTube channel.
-   **Result:** The system runs a daily sync at 09:00 UTC and will automatically detect and post the new video to the feed.

### 3.2. Manual Posting

For other content sources (like LinkedIn, blog posts, etc.), you must add an entry directly to the database.

1.  **Navigate to Supabase:** Open the [Supabase Dashboard](https://supabase.com/).
2.  **Select the Table:** In the **Table Editor**, select the `posts` table.
3.  **Insert Row:** Click the **"Insert new row"** button.
4.  **Fill Required Fields:**
    -   `content`: The main text for the post.
    -   `source`: A lowercase identifier (e.g., 'manual', 'linkedin', 'github').
    -   `external_link`: The full URL to the original content.
    -   `timestamp`: The time the post should be associated with.
5.  **Save:** Click **"Save"**. The new post will appear on the website immediately.

---

## 4. User Access Control

The `/secure` area of the portal is protected by a whitelist.

### Granting Access to a New User

1.  **User Action:** Instruct the user to visit `n45.io/secure` and attempt to sign in with their Google account. Their attempt will be logged, but they will be denied access.
2.  **Navigate to Supabase:** Open the [Supabase Dashboard](https://supabase.com/).
3.  **Select the Table:** In the **Table Editor**, select the `approved_users` table.
4.  **Find User:** You will see the user's email in the table with the `is_active` column set to `false`.
5.  **Approve User:** Click to edit the row and change `is_active` to `true`.
6.  **Save:** Click **"Save"**.
7.  **Notify User:** Inform the user they can now sign in successfully.