---
date modified: 2026-01-17
slug: n45-portal
status: Live
subtitle: A self-healing, automated personal data aggregator and digital garden.
tags:
- Next.js
- Supabase
- Python
- Automation
- Vercel
title: Showcase - N45.io
visibility: public
weight: 1
---

# Project Summary
The N45.io Portal is a living "Digital Garden" designed to aggregate and serve a lifetime of professional and personal data streams through a single, cohesive interface. The primary design goal was to achieve a zero-maintenance content pipeline; the portal updates itself automatically in response to external events, such as changes in a local knowledge base or new content posted to social media platforms.

This project demonstrates a robust, enterprise-level approach to personal knowledge management, web development, and security.

# Key Features
- **Automated Content Sync:** A custom Python script bridges a local Obsidian vault with the web frontend. It parses Markdown files, processes metadata, and pushes content to a Git repository, triggering an automated, static rebuild on Vercel.
- **Social Feed Aggregation:** The "Social" feed is powered by a Supabase (PostgreSQL) database. A recurring GitHub Actions cron job queries external services like the YouTube Data API, ingests new content, and upserts it into the database, creating a "zero-touch" activity feed.
- **Whitelist-Based Security Model:** While the main portfolio is public, sensitive areas are protected by a robust Role-Based Access Control (RBAC) system. Authentication is handled via Google OAuth 2.0, and authorization is managed through a strict user whitelist in the Supabase backend.

# System Architecture
The portal is built on a modern, decoupled architecture designed for scalability and maintainability.

- **Frontend (Presentation Layer):** A Next.js 14 application hosted on Vercel, utilizing the App Router for modern routing. The UI is built with Tailwind CSS and Framer Motion for animations.
- **Backend (Service Layer):** Vercel Serverless Functions provide API endpoints, while Supabase serves as the primary database (PostgreSQL), authentication provider, and backend-as-a-service.
- **Content Layer (Data Source):** The primary source for project showcases and articles is a private Git repository connected to the main project as a Git submodule, ensuring content is version-controlled and decoupled from the application logic.
- **Automation (CI/CD & Ingestion):** GitHub Actions orchestrate the entire CI/CD pipeline, including running cron jobs for data ingestion and triggering deployments to Vercel.

# Enterprise-Grade Documentation
This project is supported by a comprehensive suite of documentation that covers every aspect of its lifecycle, sufficient for a new developer to replicate the project from scratch. The documentation includes:
- Project Vision and Scope
- System Architecture Diagrams
- A complete Developer Onboarding Guide
- Deployment and Operations Manuals
- UI/UX Style Guides
- Content Management Procedures
- A Security and Access Control Brief

# Technology Stack
- **Frontend:** Next.js 14, React, Tailwind CSS, Framer Motion
- **Backend:** Supabase (PostgreSQL), Vercel Serverless Functions, NextAuth.js
- **Automation & CI/CD:** Python, GitHub Actions
- **Hosting:** Vercel
---
## Documentation
* [00_Documentation_Hub](/showcase/00_documentation_hub)
* [01_Project_Vision_and_Scope](/showcase/01_project_vision_and_scope)
* [02_System_Architecture](/showcase/02_system_architecture)
* [03_Developer_Onboarding_Guide](/showcase/03_developer_onboarding_guide)
* [04_Deployment_and_Operations_Manual](/showcase/04_deployment_and_operations_manual)
* [05_UI_UX_Style_Guide](/showcase/05_ui_ux_style_guide)
* [06_Content_Management_Guide](/showcase/06_content_management_guide)
* [07_Security_and_Access_Control](/showcase/07_security_access_control)