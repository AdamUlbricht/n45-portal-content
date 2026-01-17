---
date modified: 2022-01-17
parent: n45-portal
slug: ui-ux-style-guide
status: Live
subtitle: The "Look & Feel." Documents all branding, color palettes, typography, and
  component styling.
tags: null
title: Showcase - N45.io - UI/UX Style Guide
visibility: public
---

# N45.io Portal - Style Guide

This document outlines the core HTML structure, CSS frameworks, and typography conventions used in the `n45.io` website project. The information is derived from the project logs dated 2026-01-15.

---

## 1. Core Technologies

-   **CSS Framework:** **Tailwind CSS v4**. This is a utility-first framework. Styling is applied directly in the HTML via class names (e.g., `pt-16`, `text-white`).
-   **Content Styling:** The **`@tailwindcss/typography`** plugin is used to apply consistent styling to markdown-generated content. This is applied globally in `app/globals.css`.
-   **Component Framework:** **Next.js 14** with React Server Components.

---

## 2. Color Palette

The primary branding color is a "terminal" green, used for logos and key elements.

-   **Primary Accent:** `Terminal Emerald`
    -   **HEX:** `#2ECC71`
-   **Background:** Black / Dark Gray themes are used to support the terminal aesthetic.

---

## 3. Typography

The website's typography is currently based on Tailwind's default configuration, with plans for a future update.

### Current Font Stack

-   **Primary Font:** **Inter** (or a system default sans-serif font). This is the default font provided by the Tailwind CSS framework. It is used for all body copy, headings, and navigation unless otherwise specified.

### Font Styling & HTML Elements

Styling for rich text content (from markdown files) is handled automatically by the `@tailwindcss/typography` plugin. When markdown is rendered into HTML, it is wrapped in an element with the `prose` class, which applies styles to standard HTML tags.

| Element | Font / Style Description | Example Tailwind Class |
| :--- | :--- | :--- |
| `<h1>`, `<h2>`, `<h3>` | **Inter (Bold)**. Used for page titles and section headings. The typography plugin handles sizing. | `.prose h1` |
| `<p>` | **Inter (Regular)**. Standard body text. | `.prose p` |
| `<a>` | **Inter (Regular)**. Will likely be styled with the `Terminal Emerald` color for contrast. | `.prose a` |
| `<code>` | **Default Monospace Font**. Used for inline code snippets. | `.prose code` |
| `<pre>` | **Default Monospace Font**. Used for code blocks. | `.prose pre` |

### Future Direction (Backlog Item)

As noted in the project logs, there is an active backlog item to update the site's typography to better match the "Tech/Code" aesthetic.

-   **Proposed Font:** **JetBrains Mono** or a similar monospaced/technical font.
-   **Goal:** To replace "Inter" as the primary font for headings and body text to create a more cohesive brand identity.

---

## 4. Key Components & Styling

-   **Header/Navigation Bar:**
    -   **File:** `components/Header.tsx`
    -   **Style:** A fixed, "glass-morphism" style navbar.
    -   **Layout Consideration:** A top padding of `pt-16` is applied to the main content area in `app/layout.tsx` to prevent the fixed header from overlapping the page content.
-   **Responsive Layout:**
    -   The landing page (`/`) uses a three-column layout on desktop, which collapses to a single vertical stack on mobile devices.
    -   The Showcase page (`/showcase`) uses a responsive grid for project cards.