---
date: 2026-01-15
slug: n45-portal
subtitle: A self-healing, automated personal data aggregator.
tags:
- Next.js
- Supabase
- Python
- Automation
title: Showcase - N45.io
visibility: public
weight: 1
---

# Overview
This website is not just a portfolio; it is a living "Digital Garden" designed to aggregate my professional and personal data streams into a single, cohesive interface.

The goal was to eliminate manual maintenance. Instead of "updating my website," I simply update my Obsidian notes or upload a YouTube video, and the system automatically propagates those changes to the web.

# Architecture

## 1. The Data Pipe (Obsidian to Web)
My local knowledge base is managed in **Obsidian**. A custom **Python** script acts as the bridge:
* It parses my local Markdown files.
* It processes Frontmatter metadata.
* It sanitizes internal links and pushes the content to a headless Git repository.
* **Vercel** triggers a rebuild, rendering the static content via **Next.js**.

## 2. The Aggregator (Social Feed)
The "Social" tab is powered by a **Supabase** (PostgreSQL) database.
* **Ingestion:** A GitHub Action runs a Cron Job every 24 hours.
* **Logic:** It queries the **YouTube Data API** for my latest upload.
* **Storage:** New content is "Upserted" into Supabase to prevent duplicates.
* **Frontend:** The UI fetches this data in real-time, creating a "zero-touch" social feed.

## 3. Security (RBAC)
While the portfolio is public, the Social Feed utilizes **Role-Based Access Control**:
* **Authentication:** Google OAuth 2.0 (via **NextAuth.js**).
* **Authorization:** A strict whitelist system managed in Supabase.
* **Auto-Capture:** Unauthorized login attempts are captured in a "Waitlist" state, allowing for one-click approval by the admin.

# Tech Stack
* **Frontend:** Next.js 14 (App Router), Tailwind CSS, Framer Motion.
* **Backend:** Supabase, Vercel Serverless Functions.
* **Automation:** Python, GitHub Actions.