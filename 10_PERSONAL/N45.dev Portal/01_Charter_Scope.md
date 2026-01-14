---
rationale: SSOT - The Boundary. Definitive source for what is and is not in scope to prevent scope creep
key content: Objectives, In-Scope/Out-of-Scope, Constraints
visibility: public
---
[[00_Portal_Master]]
[[01_Charter_Scope]]
[[02_Architecture_Specs]]
[[03_Design_UX]]
[[04_Implementation_Plan]]
# Project Title
N45.dev - Professional and Personal Portal
# Primary Objective & Core Mission
This website serves to provide a unified web presence that demonstrates Adam Ulbricht's technical competency in game development and cybersecurity while also providing a secure private journal and consolidated social media feed for selected authorised users.
# Scope
## In-Scope:
* A three tile high impact **Landing Page** directs users immediately to the section relevant to their needs. 
* Professional recruiters are directed to the **Professional Profile**, which displays completed projects in the **Showcase**, an **ATS-Optimised resume**, and **Professional PKB**. This presents them with a digestible layout of relevant information displaying not just certifications, but demonstrable examples of completed works, documentation, and project breakdowns.
* A **CI/CD Pipeline** utilising Obsidian frontmatter tags provides the website with the content for the PKB in both personal and professional sections, as well as project documentation, descriptions, and breakdowns. 
* A **Showcase** which primarily displays completed projects in a 'journey' style narrative. Users are presented with a chronological order of events, with project purpose, research conducted, solutions attempted, failures, successes, lessons learnt, and final state. Screenshots, diagrams, and images are essential for ensuring digestible content.
* A manual social media **Aggregator** for Facebook, Instagram, YouTube, TikTok, and personal journals consolidates all social media posts into one comprehensive **Social Feed** accessible via **Secure Login** for authorised users. 
* **Federated Identity** (Google/Social) for approved secure users, allowing them access to the **Social Feed** without the need for additional accounts or passwords.
* A simplified, automated process for uploading of new projects to the showcase, secured behind the **Admin** role. 
## Out-of-Scope:
* Commercial e-commerce functionality is not needed. 
* Public-facing social media management (editing/posting from the site). The site is only to present posts which have been created on the external platform.
* High-fidelity asset refinement (this is a content task, not an infrastructure task). The scope of this project is restricted to the construction of the website and its functionality, not the population of its content. 
## Functional Requirements
### Landing Page
* If a user is already signed in, and navigates to the **Landing Page**, they remain on the **Landing Page**. The user is not prevented from access the **Landing Page** just because they are signed in. Requests for the user to sign in do not occur until they attempt to navigate to the **Social Feed**. 
* The three tiles will be laid out horizontally on desktop, and will stack vertically on mobile. 
* The page must act as a gateway with three distinct hit boxes. Clicking a tile must trigger an immediate route change without secondary confirmation.
### CI/CD Pipeline
* If a file has no `visibility` tags it is to be assumed as private. This establishes a "Default-Deny" policy.
* The site should update its obsidian-sourced content when the obsidian vault repo on GitHub receives a Push. If a new pull is available, the site should make the pull request and update its content from the repo.
* The pipeline must reject any file that does not contain valid YAML frontmatter; specifically the absence of a `visibility` key results in total exclusion.
### Federated Identity & Secure Login
* **Authorisation Level:** Once a user logs in via google, the system checks the users email against an `approved_users` table. If the email is stored in the whitelist, they are permitted access. If the email is unknown to the system, the user is presented with an opportunity to request access, after which, Admin will receive a notification indicating a new user is requesting access. It is then at the administrators discretion to approve or deny the request. If the request is denied, that email may no longer send access requests. A list of previous access request is stored in the **Supabase Database**. If the request is approved, the user is then redirected to the  **Social Feed**.
* Access to the secure route is forbidden unless the users session token is valid AND their unique ID exists in the `approved_users` table. 
* Confirmation of the sign out popup will immediately invalidate the users session token and redirect them back to the landing page to prevent session hijacking. 
* Approval and denial of users is performed manually via the the **Supabase Dashboard** 'Table Editor'. There is no in-site UI for admin approvals in Version 1.0. Admin must manually toggle the `is_active` flag in the `approved_users` table to grant access.
# Technical Constraints and Assumptions
## Repository-Driven Synchronisation
The Git-centric update logic entails the website pulling from the repo when a change in the repo occurs after updates are pushed from the local machine. The site will regularly poll the repo in order to detect changes. When a change is detected, a pull request is made, updating the website content. 
The website does not pull from a local directory, but monitors a specific GitHub repository branch. 
Any changes must be pushed to this repo to trigger the site update. 
The **Parent Website Repo** contains the site code. The **Child Obsidian Repo** is linked as a 'submodule' within the `/content` folder. Pushing to the **Child Obsidian Repo** triggers a webhook that tells **Vercel** to redeploy the site using the fresh content without needing  a full code rebuild.
* **Website (Parent) Repo:** `n45-portal-web` (Branch: `main`).
* **Documentation (Child) repo:** `n45-portal-content` (Branch: `main`).
## Build Environment Dependencies
The system relies on the **Vercel** build environment to parse the Markdown and frontmatter. The "Source of Truth" for the build is the latest commit in the repository.
## File Path Consistency
The build process assumes a static directory structure within the GitHub Repo. Moving files within the repo will break the automated pull logic. 
# Success Criteria:
## Phase 1 - Infrastructure
Complete when the domain n45.dev resolves to a **Vercel**-hosted 'coming soon' page.
### Verification Test
**Verification:** Navigate to `N45.dev` in a browser
	**Success:** Confirmed if the **Vercel** default page renders via HTTPS
## Phase 2 - Core UI
Complete when:
* Standardised theme/layout across the entire site is established and replicable when adding new pages.
* **Landing Page** theme/layout is finalised and constructed, showing the finished version on the live domain.
* **Professional Profile** layout is finalised and constructed, showing **ATS**-optimised resume and certifications. 
* **PKB** theme and layout is finalised and ready for population
* **Showcase** page is constructed with theme/layout standardised to match the rest of the site.
### Verification Test
**Verification:** Navigate to pages `N45.dev`, `N45.dev/professional-profile`, `N45.dev/pkb`, `N45.dev/showcase`. 
	**Success:** Confirmed when all four pages render, correctly displaying the standardised theme and stylings, and navigation between them is operational.
## Phase 3 - Data Pipe
Complete when public Obsidian files are demonstrably present on the domain and all other files are demonstrably not. **PKB** page accessible and explorable as a wiki via the website.
### Verification Test
**Verification:** Push a change to the **Child Obsidian Repo**. 
	**Success:** Confirmed if the **Vercel** Dashboard shows a 'Deployment Triggered by Webhook' and the change appears on the site without a manual push to the **Parent Website Repo**.
## Phase 4 - Aggregator
Complete when **Social Feed** demonstrably shows content from the connected APIs AND removes deleted posts from the cache. 
### Verification Test
**Verification:** Create a new 'TEST' post on each social media platform to confirm the aggregator is pulling new data correctly.
	**Success:** Confirmed when the new posts are visible on the conglomerated feed.
 **Verification:** Delete the newly created 'TEST' post of each platform. 
	 **Success:** Confirmed then the previously visible test post is no longer visible in the feed.
## Phase 5 - Hardening
Complete when the Secure Portal is operational, authorised users are able to access Social Media Feed, and all other users are redirected to a '403 Forbidden' page.
### Verification Test
**Verification:** Authenticate with a whitelisted email.
	**Success:** Confirmed if `secure/social-feed` renders.
 **Verification:** Attempt to access with a non-whitelisted email. 
	 **Success:** Confirmed if the system displays a custom 403 error page.
# Risk Management & Contingency
* **Risk:** Social API changes (e.g., TikTok changes its data access).
	* **Mitigation:** The manual aggregator will be documented to fail gracefully, showing a "Source Offline" placeholder instead of breaking the entire feed.
* **Risk:** Accidental exposure of private Obsidian notes.
	* **Mitigation:** Implementation of the "Public Wrapper" directory strategy mentioned previously, ensuring the build tool never scans the entire vault.
# Data Retention and Archiving
The GitHub repository is the absolute Source of Truth. Any manual changes made directly in the Supabase DB or Vercel console will be overwritten by the next Git-triggered build.
# Maintenance Schedule
Maintenance will be carried out quarterly, to update all elements of the social media aggregator to match any changes in the social media APIs. 
If the GitHub-to-Vercel webhook fails to trigger a build for more than  1 hour after a push, an emergency maintenance action is required to check the CI/CD integration status.
