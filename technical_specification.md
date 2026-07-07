# Technical Specification Document: Thrivo Web Platform

## 1. Executive Architecture Summary

Thrivo is an all-in-one web ecosystem designed to solve platform fragmentation. The system uses a highly decoupled backend API to power the web app (built for dense discovery, dashboards, and heavy workflows) and client-side interactions.

---

## 2. Core Technology Stack

| Layer | Technology Selected | Justification |
| :--- | :--- | :--- |
| **Web Frontend** | React / TypeScript / Vite | Quick build times, clean module bundling, and fast client-side rendering. |
| **Backend API** | Node.js / Express | Handles concurrent I/O connections efficiently for real-time services. |
| **Database** | Supabase (PostgreSQL) | Enforces strict relational integrity and enables Row Level Security (RLS). |
| **Real-time Pipeline** | Redis / Socket.io | Manages real-time DM pipelines and active transaction escrow flags. |
| **Object Storage** | Supabase Storage (S3 API) | Scalable CDN storage for media assets, PDFs, and pitch decks. |

---

## 3. Database Schema Design

The system implements a polymorphic structure allowing a user to possess multiple identity profiles.

### 3.1 Users Table
Stores global registration and identity flag mappings.
* `id` (UUID, Primary Key)
* `email` (VARCHAR, Unique)
* `role_flags` (ENUM: Founder, Creator, Investor, Consumer)
* `created_at` (TIMESTAMP)

### 3.2 Founder Profiles
Linked to the parent user entity.
* `id` (UUID, Primary Key)
* `user_id` (UUID, Foreign Key ref Users)
* `company_name` (VARCHAR)
* `pitch_deck_url` (VARCHAR)
* `funding_target` (NUMERIC)
* `vibe_metrics` (JSONB)

### 3.3 Creator Profiles
Linked to the parent user entity.
* `id` (UUID, Primary Key)
* `user_id` (UUID, Foreign Key ref Users)
* `niche_tags` (VARCHAR[])
* `escrow_wallet` (VARCHAR)
* `analytics_data` (JSONB)

### 3.4 Investor Profiles
Linked to the parent user entity.
* `id` (UUID, Primary Key)
* `user_id` (UUID, Foreign Key ref Users)
* `ticket_size_min` (NUMERIC)
* `ticket_size_max` (NUMERIC)
* `vetted_status` (BOOLEAN)

### 3.5 Match Deals and Escrow
Tracks matching milestones and funding allocations.
* `id` (UUID, Primary Key)
* `founder_id` (UUID, Foreign Key ref Founder Profiles)
* `creator_id` (UUID, Foreign Key ref Creator Profiles)
* `escrow_balance` (NUMERIC)
* `milestone_status` (ENUM: Deposited, InProgress, Disbursed)
* `created_at` (TIMESTAMP)

---

## 4. Security Architecture & RLS Policies

To secure founder intellectual property and startup metrics:
1. **Row Level Security (RLS):** Every database query on sensitive metrics checks RLS policies in PostgreSQL to verify that the reading party is either the owner or an authorized investor who has signed the required NDA.
2. **Data Encryption:** High-value documents (pitch decks, cap tables, NDAs) are encrypted using AES-256 at rest and delivered via short-lived, authenticated storage URLs.
3. **Session Integrity:** JWTs are stored in HTTP-only, secure cookies to prevent XSS-based access token leaks.

---

## 5. Web Client Optimization

* **State Management:** Zustand is used for fast, non-boilerplate global state handling.
* **Asset Loading:** Adaptive CDN streaming limits video load delay on web browsers.
* **Bi-directional Stream:** Socket.io client links dynamically to the Node.js WebSocket gateway.
