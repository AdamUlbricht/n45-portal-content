---
{}
---

To ensure all developments, technical specifications, and instructional logic from this session are preserved, the following text block consolidates the entire conversation into a structured format optimized for Obsidian storage.

---

# PROJECT LOG: n45.dev Portal Development

Session Date: 2025-12-25

Status: Architecture Finalized / Documentation Ready

Primary Goal: Create a standalone, granular instruction manual for a novice developer to build the n45.dev portal.

---

## 1. Project Vision & Mission

The n45.dev website provides a unified web presence demonstrating technical competency in game development and cybersecurity.

- **Professional Portal**: ATS-optimized resume, professional profile, and public knowledge base (PKB).
    
- **Secure Portal**: Authorized access to private journals and a consolidated social media feed.
    
- **Technical Ethos**: Implements a "Default-Deny" security posture and automated CI/CD synchronization.
    

---

## 2. Technical Baseline & Constraints

- **Runtime**: Node.js `20.x` (LTS) to ensure 1:1 parity between local development and the Vercel cloud environment.
    
- **Dual-Repo Strategy**:
    
    - **Website (Parent) Repo**: `n45-portal-web` (Branch: `main`).
        
    - **Documentation (Child) Repo**: `n45-portal-content` (Branch: `main`) linked as a git submodule.
        
- **Branch Protection**: The `main` branch requires a pull request before merging to prevent accidental deployment of broken code.
    

---

## 3. Database Schema Definitions (Supabase)

### Table: `posts` (The Aggregator Feed)

- `id`: `uuid` (Primary Key).
    
- `external_link`: `text` (Unique, Not Null) — Primary key for "Differential Fetching".
    
- `timestamp`: `timestamptz` (Not Null) — For chronological rendering.
    
- `source`: `text` (youtube, tiktok, instagram, facebook, obsidian).
    
- `content`: `text` (Caption/Body).
    
- `media_url`: `text` (Thumbnail/Preview link).
    
- `metadata`: `jsonb` (Platform-specific data).
    

### Table: `approved_users` (Whitelist)

- `email`: `text` (Primary Key).
    
- `is_active`: `boolean` (Default: `false`) — Master access toggle.
    
- `request_date`: `timestamptz` (Default: `now()`).
    

### Table: `sync_logs` (Forensics)

- `event_type`: `text` (SYNC_SUCCESS, SYNC_FAILURE, AUTH_DENIED).
    
- `error_code`: `text` (e.g., `ERR_API_TIMEOUT`, `ERR_DATA_MISMATCH`).
    
- `raw_payload`: `jsonb` (Rejected metadata for forensic review).
    

---

## 4. The Sync Engine: Differential Fetching Logic

The system maintains a live mirror of social presence using a 24-hour automated reconciliation loop.

1. **Normalization Rule**: All `external_link` URLs must be stripped of tracking parameters (e.g., `?si=…`) before database comparison.
    
2. **The Loop**:
    
    - **Fetch**: Pull current posts from Social APIs.
        
    - **Diffing**:
        
        - **New ID**: INSERT into database.
            
        - **Missing ID**: DELETE from database (Prune).
            
        - **Matching ID**: UPDATE metadata if changed.
            
3. **Safety Threshold**: If the deletion count exceeds 10% of the total feed in a single run, the script must Halt and Alert to prevent mass data loss.
    

---

## 5. Security Architecture (The Auth Guard)

A Next.js Middleware protects `/secure/*` routes through a 3-tier logical gate.

- **Tier 1 (Session)**: Checks for a valid Supabase Auth session token. If null, redirect to `/login`.
    
- **Tier 2 (Identity)**: Extracts user email and queries the `approved_users` table. If missing, redirect to `/access-request`.
    
- **Tier 3 (Authorization)**: Checks `is_active` flag. If `false`, terminate session and redirect to **403 Forbidden** page.
    

---

## 6. Obsidian Integration (The Vault)

Project documentation in Obsidian uses a strict metadata schema for automated web rendering.

- **Naming Convention**: `YYYY-MM_ProjectTitle_Status` (e.g., `2025-12_Non-Euclidean-Portal_Active.md`).
    
- **Mandatory YAML Keys**: `project_name`, `status`, `visibility`, `exploration_goal`, `doc_source`.
    
- **Validation**: Files missing mandatory keys or containing invalid enums are skipped and logged as `ERR_DATA_MISMATCH`.
    

---

## 7. Operational Roadmap (Phased Success)

- **Phase 1**: Domain `n45.dev` resolves to Vercel via HTTPS.
    
- **Phase 2**: Navigation between Core UI routes is operational and standardized.
    
- **Phase 3**: Public Obsidian files render live upon GitHub push; PDF Resume is auto-generated via Puppeteer `postbuild` script.
    
- **Phase 4**: Social Aggregator correctly Insert/Prunes 'TEST' posts.
    
- **Phase 5**: Secure Portal redirects non-whitelisted emails to a custom 403 page with error codes and timestamps.
    

---

**End of Session Log**.