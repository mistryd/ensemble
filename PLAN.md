# Wedding Guest List Management System - Implementation Plan

## Overview
This document outlines the step-by-step implementation plan for building the wedding guest list management system as specified in SPEC.md.

## Phase 1: Project Setup & Infrastructure

### Step 1.1: Initialize Next.js Project
- [ ] Create Next.js 14+ project with TypeScript
- [ ] Configure project structure:
  ```
  /src
    /app              # Next.js 14 app router
    /components       # Reusable UI components
    /lib              # Utilities, db client, types
    /types            # TypeScript type definitions
  ```
- [ ] Install core dependencies:
  - React 18+
  - TypeScript
  - Tailwind CSS
  - React Hook Form
  - Zod (validation)
- [ ] Configure Tailwind CSS
- [ ] Set up ESLint and Prettier
- [ ] Initialize git repository (already done)

**Deliverable:** Working Next.js app with TypeScript and Tailwind configured

### Step 1.2: Supabase Setup
- [ ] Create Supabase project (free tier)
- [ ] Note database connection credentials
- [ ] Set up environment variables (.env.local):
  ```
  NEXT_PUBLIC_SUPABASE_URL=
  NEXT_PUBLIC_SUPABASE_ANON_KEY=
  SUPABASE_SERVICE_ROLE_KEY=
  ```
- [ ] Install Supabase client: `@supabase/supabase-js`
- [ ] Create Supabase client utility in `/src/lib/supabase.ts`

**Deliverable:** Supabase project created and connected to Next.js

## Phase 2: Database Schema & Setup

### Step 2.1: Create Database Schema
- [ ] Create migration file for database schema
- [ ] Define `guests` table:
  ```sql
  CREATE TABLE guests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(254),
    address JSONB,
    inclusion_status VARCHAR(20) NOT NULL CHECK (inclusion_status IN ('definitely', 'maybe', 'not_invited')),
    rsvp_status VARCHAR(20) CHECK (rsvp_status IN ('invited', 'attending', 'declined')),
    side VARCHAR(10) NOT NULL CHECK (side IN ('bride', 'groom', 'both')),
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
  );
  ```
- [ ] Define `groups` table:
  ```sql
  CREATE TABLE groups (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL UNIQUE,
    type VARCHAR(20) NOT NULL CHECK (type IN ('family', 'couple', 'friend_group')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
  );
  ```
- [ ] Define `guest_groups` junction table:
  ```sql
  CREATE TABLE guest_groups (
    guest_id UUID NOT NULL REFERENCES guests(id) ON DELETE CASCADE,
    group_id UUID NOT NULL REFERENCES groups(id) ON DELETE CASCADE,
    PRIMARY KEY (guest_id, group_id)
  );
  ```
- [ ] Run migration in Supabase

**Deliverable:** Database tables created in Supabase

### Step 2.2: Create Database Indexes
- [ ] Add index on `guest_groups.guest_id`
- [ ] Add index on `guest_groups.group_id`
- [ ] Add index on `guests.inclusion_status` (for filtering)
- [ ] Add index on `guests.rsvp_status` (for filtering)
- [ ] Add index on `guests.side` (for filtering)

**Deliverable:** Optimized database with indexes

### Step 2.3: Set Up Real-time Subscriptions
- [ ] Enable real-time on `guests` table in Supabase
- [ ] Enable real-time on `groups` table in Supabase
- [ ] Enable real-time on `guest_groups` table in Supabase

**Deliverable:** Real-time enabled for concurrent editing

## Phase 3: TypeScript Types & Validation Schemas

### Step 3.1: Define TypeScript Types
- [ ] Create `/src/types/index.ts`
- [ ] Define `Guest` type
- [ ] Define `Group` type
- [ ] Define `GuestGroup` type
- [ ] Define API request/response types

**Deliverable:** Type-safe TypeScript definitions

### Step 3.2: Create Zod Validation Schemas
- [ ] Create `/src/lib/validation.ts`
- [ ] Define `GuestSchema` with all validation rules from SPEC.md
- [ ] Define `GroupSchema` with all validation rules
- [ ] Define `AddressSchema` (nested in GuestSchema)
- [ ] Export validation functions for use in API routes and forms

**Deliverable:** Reusable validation schemas

## Phase 4: API Routes (Backend)

### Step 4.1: Guest API Endpoints
- [ ] Create `/src/app/api/guests/route.ts`:
  - [ ] GET /api/guests - List all guests with groups
  - [ ] POST /api/guests - Create new guest
- [ ] Create `/src/app/api/guests/[id]/route.ts`:
  - [ ] GET /api/guests/[id] - Get single guest
  - [ ] PATCH /api/guests/[id] - Update guest
  - [ ] DELETE /api/guests/[id] - Delete guest
- [ ] Implement validation using Zod schemas
- [ ] Implement error handling with user-friendly messages

**Deliverable:** Working guest CRUD API

### Step 4.2: Group API Endpoints
- [ ] Create `/src/app/api/groups/route.ts`:
  - [ ] GET /api/groups - List all groups with member counts
  - [ ] POST /api/groups - Create group (with optional guests)
- [ ] Create `/src/app/api/groups/[id]/route.ts`:
  - [ ] GET /api/groups/[id] - Get single group with members
  - [ ] PATCH /api/groups/[id] - Update group
  - [ ] DELETE /api/groups/[id] - Delete group
- [ ] Implement bulk guest creation logic in POST /api/groups
- [ ] Implement validation and error handling

**Deliverable:** Working group CRUD API

### Step 4.3: Guest-Group Relationship Endpoints
- [ ] Create `/src/app/api/guests/[id]/groups/route.ts`:
  - [ ] POST - Add guest to groups
- [ ] Create `/src/app/api/guests/[id]/groups/[groupId]/route.ts`:
  - [ ] DELETE - Remove guest from group
- [ ] Implement validation (group IDs exist)

**Deliverable:** Working relationship management API

### Step 4.4: API Testing
- [ ] Test all endpoints with sample data
- [ ] Verify validation rules are enforced
- [ ] Test error cases (invalid data, missing fields, etc.)
- [ ] Verify database constraints work correctly

**Deliverable:** Fully tested backend API

## Phase 5: Frontend - Core Components

### Step 5.1: Layout & Navigation
- [ ] Create `/src/components/Layout.tsx` - Main layout wrapper
- [ ] Create `/src/components/Navigation.tsx` - Nav menu with links to all pages
- [ ] Add responsive mobile navigation
- [ ] Style with Tailwind CSS

**Deliverable:** App layout and navigation

### Step 5.2: Shared Components
- [ ] Create `/src/components/Button.tsx` - Reusable button component
- [ ] Create `/src/components/Input.tsx` - Form input with validation display
- [ ] Create `/src/components/Select.tsx` - Dropdown select
- [ ] Create `/src/components/Card.tsx` - Card container
- [ ] Create `/src/components/Modal.tsx` - Modal dialog
- [ ] Create `/src/components/LoadingSpinner.tsx`
- [ ] Create `/src/components/ErrorMessage.tsx`

**Deliverable:** Reusable component library

### Step 5.3: Data Management Setup
- [ ] Create `/src/lib/hooks/useGuests.ts` - Hook to fetch/manage guests
- [ ] Create `/src/lib/hooks/useGroups.ts` - Hook to fetch/manage groups
- [ ] Implement real-time subscription in hooks
- [ ] Set up client-side caching and state management
- [ ] Implement optimistic updates

**Deliverable:** Data fetching and real-time sync

## Phase 6: Frontend - Guest Management

### Step 6.1: Guest List Page
- [ ] Create `/src/app/guests/page.tsx`
- [ ] Display guests in table format
- [ ] Implement client-side search (name, email)
- [ ] Implement client-side filters:
  - [ ] Filter by inclusion_status
  - [ ] Filter by rsvp_status
  - [ ] Filter by side
  - [ ] Filter by group
- [ ] Implement sortable columns
- [ ] Add inline edit/delete actions
- [ ] Add "Add Guest" button linking to /guests/new
- [ ] Handle loading and error states
- [ ] Make responsive for mobile

**Deliverable:** Working guest list view

### Step 6.2: Guest Form Component
- [ ] Create `/src/components/GuestForm.tsx`
- [ ] Implement React Hook Form with Zod validation
- [ ] Add all guest fields:
  - [ ] First name, last name (required)
  - [ ] Email (optional)
  - [ ] Address fields (optional, all-or-nothing)
  - [ ] Inclusion status (dropdown)
  - [ ] RSVP status (dropdown)
  - [ ] Side (dropdown)
  - [ ] Notes (textarea)
  - [ ] Group assignment (multi-select)
- [ ] Display validation errors inline
- [ ] Handle form submission with loading state
- [ ] Implement optimistic updates

**Deliverable:** Reusable guest form component

### Step 6.3: Add Guest Page
- [ ] Create `/src/app/guests/new/page.tsx`
- [ ] Use GuestForm component
- [ ] Handle POST /api/guests
- [ ] Redirect to /guests on success
- [ ] Show success/error toast notifications

**Deliverable:** Add guest functionality

### Step 6.4: Edit Guest Page
- [ ] Create `/src/app/guests/[id]/edit/page.tsx`
- [ ] Fetch guest data and pre-populate form
- [ ] Use GuestForm component
- [ ] Handle PATCH /api/guests/[id]
- [ ] Redirect to /guests on success
- [ ] Handle not found case

**Deliverable:** Edit guest functionality

## Phase 7: Frontend - Group Management

### Step 7.1: Group List Page
- [ ] Create `/src/app/groups/page.tsx`
- [ ] Display all groups with member counts
- [ ] Add create/edit/delete actions
- [ ] Show group members on expand/click
- [ ] Add "Create Group" button
- [ ] Make responsive for mobile

**Deliverable:** Working group list view

### Step 7.2: Group Form Component
- [ ] Create `/src/components/GroupForm.tsx`
- [ ] Implement React Hook Form with Zod validation
- [ ] Add fields:
  - [ ] Name (required, unique)
  - [ ] Type (dropdown)
  - [ ] Guest selection (multi-select existing guests)
  - [ ] Bulk add new guests (dynamic form array)
- [ ] Handle both simple and bulk creation
- [ ] Display validation errors

**Deliverable:** Group form with bulk add functionality

### Step 7.3: Group CRUD Operations
- [ ] Implement create group functionality
- [ ] Implement edit group functionality
- [ ] Implement delete group functionality
- [ ] Handle add/remove guests to/from groups
- [ ] Show confirmation for destructive actions

**Deliverable:** Full group management

## Phase 8: Frontend - Refinement View

### Step 8.1: Refinement Board Component
- [ ] Create `/src/app/refine/page.tsx`
- [ ] Create three-column layout:
  - [ ] "Definitely In" (inclusion_status = "definitely")
  - [ ] "Maybe" (inclusion_status = "maybe")
  - [ ] "Not Invited" (inclusion_status = "not_invited")
- [ ] Display guests grouped by their groups within each column
- [ ] Implement drag-and-drop between columns (optional, can start with click)
- [ ] Update inclusion_status when moving guests
- [ ] Show guest count per column
- [ ] Make responsive for mobile (vertical stack)

**Deliverable:** Working refinement view

### Step 8.2: Refinement Interactions
- [ ] Implement smooth animations for status changes
- [ ] Add bulk select and move functionality (nice-to-have)
- [ ] Show visual feedback on hover/drag
- [ ] Handle optimistic updates

**Deliverable:** Polished refinement experience

## Phase 9: Frontend - Dashboard & Analytics

### Step 9.1: Analytics Logic
- [ ] Create `/src/lib/analytics.ts`
- [ ] Implement functions to compute:
  - [ ] Counts by inclusion_status
  - [ ] Counts by rsvp_status
  - [ ] Counts by side
  - [ ] Counts by group
- [ ] Create custom hook `useAnalytics` to compute from guest data

**Deliverable:** Analytics computation logic

### Step 9.2: Dashboard Page
- [ ] Create `/src/app/page.tsx` (homepage = dashboard)
- [ ] Create stat cards showing:
  - [ ] Total guests
  - [ ] By inclusion status (definitely/maybe/not_invited)
  - [ ] By RSVP status (invited/attending/declined/pending)
  - [ ] By side (bride/groom/both)
- [ ] Create group breakdown section
- [ ] Add quick navigation to other views
- [ ] Make responsive for mobile

**Deliverable:** Working dashboard with analytics

## Phase 10: Polish & User Experience

### Step 10.1: Real-time Updates
- [ ] Verify real-time sync works across multiple browser tabs
- [ ] Test concurrent editing scenarios
- [ ] Add visual indicators for data updates (optional)
- [ ] Handle connection loss gracefully

**Deliverable:** Robust real-time collaboration

### Step 10.2: Error Handling & Validation
- [ ] Ensure all forms show clear validation errors
- [ ] Add user-friendly error messages for API failures
- [ ] Add toast notifications for success/error states
- [ ] Handle network errors gracefully
- [ ] Add loading states to all async operations

**Deliverable:** Polished error handling

### Step 10.3: Mobile Responsiveness
- [ ] Test all pages on mobile devices
- [ ] Ensure touch-friendly interactions
- [ ] Optimize table display for small screens
- [ ] Test forms on mobile
- [ ] Verify navigation works on mobile

**Deliverable:** Mobile-optimized experience

### Step 10.4: Performance Optimization
- [ ] Implement virtual scrolling if needed for large guest lists
- [ ] Optimize re-renders with React.memo where appropriate
- [ ] Ensure search/filter performance is < 500ms
- [ ] Test with 400+ guest records
- [ ] Optimize images and assets

**Deliverable:** Performant application

## Phase 11: Testing & QA

### Step 11.1: Manual Testing
- [ ] Test all CRUD operations (guests, groups)
- [ ] Test all validation rules from SPEC.md
- [ ] Test edge cases (empty states, max character limits, etc.)
- [ ] Test concurrent editing from 2 browsers
- [ ] Test on desktop and mobile browsers
- [ ] Verify all success criteria from SPEC.md

**Deliverable:** Thoroughly tested application

### Step 11.2: Data Integrity Testing
- [ ] Test cascade deletes (guest, group)
- [ ] Test uniqueness constraints (group names)
- [ ] Test address all-or-nothing validation
- [ ] Test optional group assignment
- [ ] Test solo guests without groups

**Deliverable:** Verified data integrity

## Phase 12: Deployment

### Step 12.1: Prepare for Deployment
- [ ] Set up environment variables in Vercel
- [ ] Configure Supabase for production
- [ ] Review security settings (RLS policies if needed)
- [ ] Set up proper CORS configuration
- [ ] Add .env.example file to repository

**Deliverable:** Deployment-ready configuration

### Step 12.2: Deploy to Vercel
- [ ] Connect GitHub repository to Vercel
- [ ] Configure build settings
- [ ] Deploy to production
- [ ] Test production deployment
- [ ] Verify environment variables are set correctly

**Deliverable:** Live production application

### Step 12.3: Post-Deployment
- [ ] Share URL with stakeholders
- [ ] Monitor for errors using Vercel dashboard
- [ ] Test from multiple devices
- [ ] Verify free tier limits are sufficient
- [ ] Document any issues for future fixes

**Deliverable:** Deployed and monitored application

## Phase 13: Documentation & Handoff

### Step 13.1: Technical Documentation
- [ ] Document environment setup in README.md
- [ ] Document database schema and migrations
- [ ] Document API endpoints (can reference SPEC.md)
- [ ] Add code comments for complex logic
- [ ] Document deployment process

**Deliverable:** Complete technical documentation

### Step 13.2: User Guide
- [ ] Create basic user guide for non-technical users
- [ ] Document common workflows (add guest, create group, etc.)
- [ ] Document the refinement process
- [ ] Add troubleshooting tips

**Deliverable:** User-facing documentation

## Estimated Timeline

**Phase 1-2 (Setup & Database):** 2-4 hours
**Phase 3-4 (Types & API):** 4-6 hours
**Phase 5 (Core Components):** 3-4 hours
**Phase 6 (Guest Management):** 4-6 hours
**Phase 7 (Group Management):** 3-5 hours
**Phase 8 (Refinement View):** 2-4 hours
**Phase 9 (Dashboard):** 2-3 hours
**Phase 10 (Polish):** 3-5 hours
**Phase 11 (Testing):** 2-4 hours
**Phase 12 (Deployment):** 1-2 hours
**Phase 13 (Documentation):** 1-2 hours

**Total Estimated Time:** 27-45 hours

## Success Metrics

- [ ] All success criteria from SPEC.md are met
- [ ] Application loads in < 2 seconds
- [ ] Search/filter returns results in < 500ms
- [ ] Successfully handles 300-400 guests
- [ ] Works on desktop and mobile browsers
- [ ] Concurrent editing works for 2 users
- [ ] Hosted on free tier ($0/month cost)
- [ ] Zero critical bugs in production

## Notes

- This plan assumes solo development or pair programming
- Timeline estimates are approximate and may vary
- Each phase should be committed to git separately
- Testing should be done continuously, not just in Phase 11
- Can adjust priorities based on user feedback
