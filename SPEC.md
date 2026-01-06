# Wedding Guest List Management System - Requirements

## Project Overview

A web-based application for managing wedding guest lists, supporting collaborative editing between multiple users without authentication. The system handles guest information, group associations, invitation tracking, and RSVP management for approximately 300-400 guests.

## Core Requirements

### User Access
- No authentication required (shared access model)
- Accessible via desktop and mobile browsers
- Multi-device support with centralized data storage

### Data Model

#### Guest Entity
```typescript
{
  id: string (UUID)
  firstName: string
  lastName: string
  email: string | null
  address: {
    street: string
    city: string
    state: string
    zipCode: string
  } | null
  inclusion_status: "definitely" | "maybe" | "not_invited"
  rsvp_status: null | "invited" | "attending" | "declined"
  side: "bride" | "groom" | "both"
  notes: string | null
  createdAt: timestamp
  updatedAt: timestamp
}
```

#### Group Entity
```typescript
{
  id: string (UUID)
  name: string
  type: "family" | "couple" | "friend_group"
  createdAt: timestamp
  updatedAt: timestamp
}
```

#### GuestGroup Junction Table (Many-to-Many)
```typescript
{
  guestId: string (FK)
  groupId: string (FK)
}
```

**Constraints:**
- Guests may belong to zero or more groups (group assignment is optional)
- Guests may belong to multiple groups
- Groups may contain multiple guests

## Data Validation Rules

### Guest Entity Validation

**firstName:**
- Required: Yes
- Min length: 1 character
- Max length: 100 characters
- Allowed characters: Letters (including accented), spaces, hyphens, apostrophes
- Pattern: `/^[a-zA-ZÀ-ÿ\s'-]+$/`

**lastName:**
- Required: Yes
- Min length: 1 character
- Max length: 100 characters
- Allowed characters: Letters (including accented), spaces, hyphens, apostrophes
- Pattern: `/^[a-zA-ZÀ-ÿ\s'-]+$/`

**email:**
- Required: No (nullable)
- Format: Standard email format validation
- Max length: 254 characters
- Uniqueness: Not enforced (multiple guests may share same email)
- Pattern: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`

**address:**
- Required: No (entire object can be null)
- If provided, all fields within address object are required
- Supports international addresses (not limited to US format)

**address.street:**
- Required: If address provided
- Max length: 200 characters

**address.city:**
- Required: If address provided
- Max length: 100 characters

**address.state:**
- Required: If address provided
- Max length: 100 characters
- Note: Can be state, province, region, or equivalent

**address.zipCode:**
- Required: If address provided
- Max length: 20 characters
- Note: Supports international postal codes (not limited to US zip format)

**inclusion_status:**
- Required: Yes
- Allowed values: "definitely", "maybe", "not_invited"
- Default: "maybe" (on create)

**rsvp_status:**
- Required: No (null is valid)
- Allowed values: null, "invited", "attending", "declined"
- Default: null

**side:**
- Required: Yes
- Allowed values: "bride", "groom", "both"
- No default (user must explicitly choose)

**notes:**
- Required: No (nullable)
- Max length: 1000 characters

**groupIds (relationship):**
- Required: No (group assignment is optional)
- Validation: If provided, all group IDs must exist in database
- Guests without group assignments are allowed (e.g., solo guests)

### Group Entity Validation

**name:**
- Required: Yes
- Min length: 1 character
- Max length: 200 characters
- Uniqueness: Must be unique across all groups
- Trim whitespace before uniqueness check

**type:**
- Required: Yes
- Allowed values: "family", "couple", "friend_group"
- No default (user must explicitly choose)

### Relationship Validation

**Guest-Group Association:**
- Guests may belong to zero or more groups (optional)
- Group assignment is not required for guest creation
- Groups can be deleted without restriction (guests without groups are allowed)
- Guests can be removed from all groups (solo guests are allowed)

### Validation Layers

**Client-side (React Hook Form + Zod):**
- Immediate field-level validation
- Format validation (email, character limits)
- Required field checks
- Pattern matching
- User-friendly error messages

**API Route (Zod schema):**
- Re-validate all client-side rules
- Business logic validation
- Uniqueness checks (group names)
- Relationship validation (group IDs exist)
- Cross-field validation (address all-or-nothing)

**Database (PostgreSQL constraints):**
- NOT NULL constraints
- CHECK constraints for enum values
- UNIQUE constraints (group names)
- Foreign key constraints
- ON DELETE CASCADE for guest_groups junction table (cleanup when guest or group deleted)

## Functional Requirements

### FR-1: Guest Management
- **FR-1.1:** Create, read, update, delete guest records
- **FR-1.2:** Group assignment is optional (support solo guests without group affiliation)
- **FR-1.3:** Support optional structured address input (street, city, state, zip)
- **FR-1.4:** Track guest metadata (side association, notes, contact information)

### FR-2: Group Management
- **FR-2.1:** Create, read, update, delete groups
- **FR-2.2:** Assign predefined group types: family, couple, friend_group
- **FR-2.3:** Support many-to-many guest-group relationships
- **FR-2.4:** Bulk guest creation: create a group and add multiple guests (new or existing) simultaneously

### FR-3: Inclusion Status Workflow
- **FR-3.1:** Track three inclusion states: "definitely", "maybe", "not_invited"
- **FR-3.2:** Allow status transitions between all states
- **FR-3.3:** Maintain historical records of guests not invited

### FR-4: RSVP Tracking
- **FR-4.1:** Track RSVP status: null (not sent), "pending", "attending", "declined"
- **FR-4.2:** Independent from inclusion status workflow
- **FR-4.3:** Support status updates as responses are received

### FR-5: Analytics & Reporting
- **FR-5.1:** Display counts by inclusion_status
- **FR-5.2:** Display counts by rsvp_status
- **FR-5.3:** Display counts by side (bride/groom/both)
- **FR-5.4:** Display counts by group
- **FR-5.5:** All counts should be real-time and reactive

## User Interface Requirements

### UI-1: Guest List Refinement View
- Three-column layout: "Definitely In" | "Maybe" | "Not Invited"
- Display guests grouped by their group associations
- Enable quick status transitions between columns
- Primary view for initial list curation phase

### UI-2: Main Guest List View
- Tabular display of all guests
- Search functionality (name, email)
- Filtering by: inclusion_status, rsvp_status, side, group
- Sorting capabilities
- Inline edit actions

### UI-3: Guest Form (Add/Edit)
- Single form for create and update operations
- Structured address input fields
- Group assignment (multi-select)
- Field validation

### UI-4: Group Management View
- List all groups with member counts
- Create/edit/delete operations
- Bulk add interface: create group + add multiple guests in one flow
- Support adding existing guests to groups

### UI-5: Dashboard/Analytics View
- Visual summary cards for key metrics
- Breakdown by inclusion_status, rsvp_status, side
- Group-level statistics

### UI-6: Responsive Design
- Mobile-first approach
- Touch-friendly interactions
- Responsive layouts for tablet and desktop

## Technical Stack

### Frontend
- **Framework:** Next.js 14+ with TypeScript
- **UI Library:** React 18+
- **Styling:** Tailwind CSS (recommended) or CSS Modules
- **State Management:** React Context or Zustand (if needed)

### Backend
- **API Routes:** Next.js API routes
- **Database:** Supabase (PostgreSQL)
- **ORM/Client:** Supabase JavaScript client or Prisma

### Hosting & Deployment
- **Platform:** Vercel (free tier)
- **Database:** Supabase (free tier: 500MB, sufficient for 300-400 guests)
- **Estimated Cost:** $0/month

### Concurrent Editing Strategy
- **Real-time Sync:** Leverage Supabase real-time subscriptions to broadcast changes across clients
- **Conflict Resolution:** Last-write-wins approach (simpler for 2-user scenario)
- **Optimistic Updates:** UI updates immediately, rolls back on server error
- **Stale Data Prevention:** Refresh guest list data when receiving real-time updates from other users
- **Visual Feedback:** Optional indicator showing when another user is actively editing (nice-to-have)

## API Endpoints

### Guest Endpoints

**GET /api/guests**
- List all guests with their group associations
- No server-side filtering (client-side filtering preferred for dataset size)
- Returns: Array of guest objects with populated group data

**POST /api/guests**
- Create new guest
- Request body:
  ```typescript
  {
    firstName: string
    lastName: string
    email?: string | null
    address?: {
      street: string
      city: string
      state: string
      zipCode: string
    } | null
    inclusion_status: "definitely" | "maybe" | "not_invited"
    rsvp_status?: null | "invited" | "attending" | "declined"
    side: "bride" | "groom" | "both"
    notes?: string | null
    groupIds?: string[]  // Optional - can create solo guests without groups
  }
  ```
- Returns: Created guest object

**GET /api/guests/[id]**
- Get single guest by ID
- Returns: Guest object with group associations

**PATCH /api/guests/[id]**
- Update guest (partial update)
- Request body: Partial guest object (any updatable fields)
- Returns: Updated guest object

**DELETE /api/guests/[id]**
- Delete guest and all guest-group associations
- Returns: Success confirmation

### Group Endpoints

**GET /api/groups**
- List all groups with member counts
- Returns: Array of group objects with member count

**POST /api/groups**
- Create group, optionally with guests (new or existing)
- Request body:
  ```typescript
  {
    name: string
    type: "family" | "couple" | "friend_group"
    guestIds?: string[]        // Add existing guests to this group
    newGuests?: Array<{        // Create new guests and add to this group
      firstName: string
      lastName: string
      email?: string | null
      address?: Address | null
      inclusion_status: "definitely" | "maybe" | "not_invited"
      rsvp_status?: null | "invited" | "attending" | "declined"
      side: "bride" | "groom" | "both"
      notes?: string | null
    }>
  }
  ```
- Returns: Created group object with associated guests

**GET /api/groups/[id]**
- Get single group with full member list
- Returns: Group object with populated guest data

**PATCH /api/groups/[id]**
- Update group name or type
- Request body: `{ name?: string, type?: string }`
- Returns: Updated group object

**DELETE /api/groups/[id]**
- Delete group and remove all guest-group associations
- Guest-group associations automatically cleaned up via cascade delete
- Returns: Success confirmation

### Guest-Group Relationship Endpoints

**POST /api/guests/[id]/groups**
- Add guest to one or more groups
- Request body: `{ groupIds: string[] }`
- Returns: Updated guest object

**DELETE /api/guests/[id]/groups/[groupId]**
- Remove guest from specific group
- Guest may be removed from all groups (solo guests allowed)
- Returns: Success confirmation

### Analytics
- No dedicated endpoint
- Analytics computed client-side from GET /api/guests data
- Calculations: counts by inclusion_status, rsvp_status, side, and group

## Application Routes

### Page Structure

**/ (Home/Dashboard)**
- Dashboard view with analytics and summary cards
- Real-time counts by inclusion_status, rsvp_status, side
- Group-level statistics
- Quick navigation to other views

**/guests**
- Main guest list view (tabular display)
- Client-side search (name, email)
- Client-side filtering (inclusion_status, rsvp_status, side, group)
- Sortable columns
- Inline edit and delete actions
- "Add Guest" button

**/guests/new**
- Guest creation form
- All guest fields
- Group assignment (multi-select, optional - supports solo guests)
- Cancel returns to /guests

**/guests/[id]/edit**
- Guest edit form
- Pre-populated with existing data
- Group management (add/remove groups with validation)
- Cancel returns to /guests

**/refine**
- Three-column refinement view
- Columns: "Definitely In" | "Maybe" | "Not Invited"
- Guests displayed grouped by their group associations
- Drag-and-drop or click to move between columns (updates inclusion_status)
- Primary view for initial list curation phase

**/groups**
- Group management view
- List of all groups with member counts
- Create/edit/delete group actions
- Bulk add interface: create group + add multiple guests
- Click group to see member list

## Non-Functional Requirements

### Performance
- **NFR-1:** Page load time < 2 seconds
- **NFR-2:** Search results return in < 500ms
- **NFR-3:** Support concurrent editing from 2 users

### Data Integrity
- **NFR-4:** All database operations must be atomic
- **NFR-5:** Foreign key constraints enforced for guest-group relationships
- **NFR-6:** Cascade delete guest-group associations when guest or group is deleted

### Usability
- **NFR-7:** Interface must be intuitive for non-technical users
- **NFR-8:** Mobile experience equivalent to desktop functionality
- **NFR-9:** Form validation with clear error messages

### Scalability
- **NFR-10:** Support up to 500 guests and 100 groups
- **NFR-11:** Database queries optimized for dataset size

## Out of Scope (Initial Release)

- User authentication and authorization
- Plus-one guest management and tracking
- Guest-facing RSVP portal
- Email notifications or bulk communications
- Invitation tracking (save-the-date, formal invites sent)
- Dietary restrictions and meal choices
- Data import/export functionality (CSV upload, mailing labels, vendor lists)
- Audit logging

## Future Considerations

- Plus-one guest management and relationship tracking
- RSVP tracking enhancements (dietary restrictions, meal choices)
- Invitation batch management
- Data import/export functionality (CSV, mailing labels)
- Guest-facing RSVP form
- Multi-user authentication

## Success Criteria

1. Both users can access and edit guest list from any device
2. Successfully manage 300-400 guests with optional group associations
3. Support solo guests without group affiliation
4. Refinement workflow enables efficient decision-making on inclusion
5. Real-time analytics provide accurate guest counts
6. System remains responsive with full dataset
7. Zero cost operation on free hosting tiers
