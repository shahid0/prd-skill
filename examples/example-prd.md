# PRD: Saved Searches

**Generated with `/prd "saved searches"`**

---

### 1. Overview

**Feature / Project Name:** Saved Searches

**Problem Statement:**
Power users in our SaaS analytics dashboard run the same complex search queries repeatedly throughout the day — filtering by date range, event type, user segment, and custom properties. There's no way to save these queries. Every session starts from scratch, costing users 2–5 minutes of setup time per query and creating friction that discourages deep exploration.

**Proposed Solution:**
Let users save, name, and quickly re-apply search queries from a persistent sidebar panel. Saved searches are scoped to the user's account and accessible across devices.

**AI Build Summary:**
> Build a Next.js 14 + Supabase feature that lets authenticated users save, name, organize, and re-apply search queries in an analytics dashboard. Saved searches persist to a Postgres table via Supabase RLS. UI is a collapsible sidebar panel with a list of saved searches, a save-current-search button in the search bar, and a delete confirmation modal. No sharing or team-level saved searches in MVP.

---

### 2. Goals & Success Metrics

**Primary Goal:** Reduce time-to-query for returning users by making complex searches reusable.

**Success Metrics:**
- 30% of power users (users who search >5x/week) save at least one search within 2 weeks of launch
- Median time-to-apply-a-search drops from 3 min to under 15 seconds for saved searches
- <2% error rate on save/load operations

**Anti-goals:**
- Not building team-level or shared saved searches (a future phase)
- Not building search folders or tagging (keep it flat for MVP)
- Not supporting public/shareable saved search URLs

---

### 3. Scope & Constraints

**In scope:**
- Save current search state as a named bookmark
- List all saved searches in the sidebar
- Click a saved search to re-apply it instantly
- Rename and delete saved searches
- Persist saved searches across sessions and devices (server-side)

**Out of scope:**
- Sharing saved searches with teammates
- Folders, tags, or nesting
- Search history (distinct from saved searches)
- Bulk operations (bulk delete, bulk rename)

**Technical constraints:**
- Web only (no mobile app changes needed)
- Must use existing Supabase auth — no new auth dependency
- RLS must prevent users from accessing each other's saved searches
- Existing search state is serialized to URL params — saved searches must capture the full URL params string
- WCAG AA accessibility required
- No offline support needed

---

### 4. Jobs to Be Done (JTBD)

| Priority | Job Statement |
|----------|---------------|
| 1 | When I open the dashboard each morning, I want to instantly apply my standard filters, so I can get to the data I need without spending time reconstructing my query. |
| 2 | When I'm in a meeting and need to pull up a specific segment quickly, I want to navigate to a known query in one click, so I can stay present in the conversation instead of fumbling with filters. |
| 3 | When I'm training a new teammate, I want to share a named, reproducible query setup, so they can see exactly what I'm looking at without me screen-sharing. |
| 4 | When I want to explore a variation of an existing query, I want to load a saved search and modify it, so I don't have to rebuild the base query from scratch. |

---

### 5. User Stories

| ID  | Role | Action | Benefit | JTBD Ref |
|-----|------|--------|---------|----------|
| US1 | Analyst | I want to save my current search with a name | so that I can re-apply it in future sessions without rebuilding it | J1 |
| US2 | Analyst | I want to see all my saved searches in the sidebar | so that I can find and apply the right one quickly | J1, J2 |
| US3 | Analyst | I want to click a saved search to instantly apply it | so that I get to the right data in one interaction | J2 |
| US4 | Analyst | I want to rename a saved search | so that I can keep my list organized as queries evolve | J4 |
| US5 | Analyst | I want to delete a saved search | so that I can remove stale or irrelevant queries from my list | J1 |
| US6 | Analyst | I want my saved searches to be available on any device | so that I'm not locked to a single machine | J2 |

---

### 6. Proposed Experience

**Design Direction:**
Familiar bookmark/favorites pattern — no learning curve. Saved searches feel like browser bookmarks for your data queries. The sidebar is non-disruptive: it collapses and doesn't interfere with the main query builder.

**Key Screens / States:**

- **Sidebar (default):** List of saved searches, each with name, last-used timestamp, and overflow menu (rename, delete). Sorted by last-used descending.
- **Save dialog:** Inline popover on the search bar's "Save" button. Single text input for name, "Save" and "Cancel" buttons. Validates uniqueness of name.
- **Empty state:** "No saved searches yet. Run a search and click Save to bookmark it." with an illustration.
- **Rename inline:** Click the name in the sidebar to enter inline edit mode; confirm on Enter or blur.
- **Delete confirmation modal:** "Delete [name]? This cannot be undone." with "Delete" (destructive) and "Cancel" buttons.
- **Loading state:** Skeleton list items while saved searches are fetching.
- **Error state (load failure):** "Couldn't load saved searches. Try refreshing." with retry button.
- **Error state (save failure):** Inline error in the save dialog. "Couldn't save search. Please try again."

**Interaction Model:**
1. User builds a search query (existing functionality)
2. User clicks "Save" button adjacent to the search bar
3. Popover appears, user types a name, confirms
4. Saved search appears at the top of the sidebar list
5. On subsequent visits: user opens sidebar → clicks saved search → filters apply instantly (URL params update, data refetches)

**Accessibility Notes:**
- Sidebar toggle button must have clear `aria-label` and `aria-expanded` state
- Saved search list must be keyboard-navigable (arrow keys, Enter to apply, Delete key to trigger delete flow)
- Delete confirmation modal must trap focus and return focus to the list item on cancel
- All interactive elements meet 4.5:1 color contrast ratio

**Figma / Design Link:** [placeholder — link to designs when available]

---

### 7. Component Inventory

| Component | Type | Description | Linked Stories |
|-----------|------|-------------|----------------|
| SavedSearchSidebar | Layout | Collapsible sidebar panel containing the full saved searches list | US2, US6 |
| SavedSearchList | Display | Scrollable list of SavedSearchItem components | US2 |
| SavedSearchItem | Display | Single row: name, last-used timestamp, overflow menu trigger | US3, US4, US5 |
| SavedSearchEmptyState | Display | Illustration + copy when no saved searches exist | US2 |
| SaveSearchButton | Action | Button in the search bar that triggers the save popover | US1 |
| SaveSearchPopover | Form | Inline popover with name input and Save/Cancel actions | US1 |
| SavedSearchOverflowMenu | Navigation | Dropdown with Rename and Delete options per item | US4, US5 |
| DeleteConfirmationModal | Modal | Confirmation dialog for destructive delete action | US5 |
| SidebarToggleButton | Action | Button to collapse/expand the sidebar | US2 |
| SkeletonSavedSearchItem | Display | Placeholder skeleton for loading state | US2 |

---

### 8. Data Models

```typescript
interface SavedSearch {
  id: string;                   // UUID, generated by Supabase
  userId: string;               // FK → auth.users.id, enforced by RLS
  name: string;                 // User-defined name, max 100 chars, unique per user
  queryParams: string;          // Serialized URL search params string (e.g., "?from=2024-01-01&segment=enterprise")
  createdAt: string;            // ISO8601
  lastUsedAt: string;           // ISO8601 — updated every time the search is applied
}

// Supabase table: saved_searches
// RLS policy: users can only SELECT, INSERT, UPDATE, DELETE their own rows
// Index: (userId, lastUsedAt DESC) for efficient list queries
```

---

### 9. API / Integration Surface

Using Supabase client directly (no custom API routes needed for MVP).

| Operation | Supabase Call | Description | Auth Required | Response Shape |
|-----------|---------------|-------------|---------------|----------------|
| List saved searches | `supabase.from('saved_searches').select('*').order('last_used_at', { ascending: false })` | Fetch all saved searches for the current user | Yes (RLS) | `SavedSearch[]` |
| Create saved search | `supabase.from('saved_searches').insert({ name, queryParams, userId })` | Save a new search | Yes (RLS) | `SavedSearch` |
| Update name | `supabase.from('saved_searches').update({ name }).eq('id', id)` | Rename a saved search | Yes (RLS) | `SavedSearch` |
| Update last used | `supabase.from('saved_searches').update({ lastUsedAt: new Date().toISOString() }).eq('id', id)` | Record when a search was applied | Yes (RLS) | `SavedSearch` |
| Delete saved search | `supabase.from('saved_searches').delete().eq('id', id)` | Remove a saved search | Yes (RLS) | `void` |

**External integrations:** None for MVP.

---

### 10. State Management Map

| State | Location | Persistence | Notes |
|-------|----------|-------------|-------|
| savedSearches | Server (Supabase) | Persistent | Source of truth; fetched on sidebar mount |
| savedSearchesLoading | Local UI (React state) | Session | True during initial fetch |
| savedSearchesError | Local UI (React state) | Session | Set if Supabase call fails |
| sidebarOpen | Local UI (React state) | Session | Could persist to localStorage in Phase 2 |
| savePopoverOpen | Local UI (React state) | Session | Controlled by SaveSearchButton |
| pendingSaveName | Local UI (React state) | Session | Input value in the save popover |
| deleteConfirmTarget | Local UI (React state) | Session | ID of the saved search pending deletion |
| activeSearchParams | URL (search params) | Session/URL | The current search state; applied when a saved search is clicked |

---

### 11. Tech Stack Recommendation

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Frontend | Next.js 14 (App Router) | Existing stack; server components reduce client bundle |
| Styling | Tailwind CSS | Existing stack; utility-first for rapid component building |
| Database | Supabase (Postgres) | Existing stack; RLS handles multi-tenant data isolation cleanly |
| Auth | Supabase Auth | Existing; no new dependency |
| Client data fetching | TanStack Query v5 | Optimistic updates, background refetch, and cache invalidation for the saved searches list |
| Hosting | Vercel | Existing stack |
| Key new libraries | `@radix-ui/react-popover`, `@radix-ui/react-dialog` | Accessible popover and modal primitives; consistent with design system |

---

### 12. Suggested File Structure

```
app/
└── dashboard/
    └── layout.tsx               # Add SavedSearchSidebar here

components/
└── saved-searches/
    ├── SavedSearchSidebar.tsx
    ├── SavedSearchList.tsx
    ├── SavedSearchItem.tsx
    ├── SavedSearchEmptyState.tsx
    ├── SaveSearchButton.tsx
    ├── SaveSearchPopover.tsx
    ├── SavedSearchOverflowMenu.tsx
    ├── DeleteConfirmationModal.tsx
    └── SkeletonSavedSearchItem.tsx

lib/
├── saved-searches.ts            # Supabase query functions
└── types.ts                     # Add SavedSearch interface here

supabase/
└── migrations/
    └── 20240301_saved_searches.sql   # Table + RLS policy migration
```

---

### 13. Acceptance Criteria

**US1 — Save current search**
- [ ] "Save" button is visible in the search bar when a query is active
- [ ] Clicking "Save" opens a popover with a text input
- [ ] Submitting with an empty name shows an inline validation error
- [ ] Submitting with a duplicate name shows: "You already have a saved search with this name"
- [ ] On success, the popover closes and the new saved search appears at the top of the sidebar list
- [ ] On failure, an inline error appears in the popover without closing it

**US2 — View saved searches in sidebar**
- [ ] Sidebar shows all saved searches sorted by `lastUsedAt` descending
- [ ] Each item shows: name and last-used timestamp (formatted as relative time: "3 days ago")
- [ ] Empty state renders when the user has no saved searches
- [ ] Skeleton items render during the initial fetch
- [ ] Error state with retry button renders if the Supabase call fails

**US3 — Apply a saved search**
- [ ] Clicking a saved search item updates the URL params to match `queryParams`
- [ ] The dashboard data refetches using the applied params
- [ ] `lastUsedAt` is updated in Supabase on click
- [ ] The applied saved search is visually indicated as active in the list

**US4 — Rename a saved search**
- [ ] Overflow menu has a "Rename" option
- [ ] Selecting "Rename" puts the item name into inline edit mode
- [ ] Pressing Enter or blurring confirms the rename
- [ ] Pressing Escape cancels without saving
- [ ] Duplicate name validation applies on rename too
- [ ] Optimistic update: name changes in the UI immediately; reverts on failure

**US5 — Delete a saved search**
- [ ] Overflow menu has a "Delete" option
- [ ] Selecting "Delete" opens the confirmation modal
- [ ] Modal shows the name of the search being deleted
- [ ] Confirming delete removes the item from the list and deletes from Supabase
- [ ] Canceling closes the modal with no changes
- [ ] Focus returns to the list item's overflow trigger on cancel

**US6 — Cross-device persistence**
- [ ] Saved searches created on Device A appear when the same user logs in on Device B
- [ ] Deleting a saved search on one device removes it when the other device refreshes

---

### 14. Open Questions & Risks

- **Q:** Should the sidebar be open by default on first load, or collapsed? — *Owner: Design*
- **Q:** Is 100-character name limit the right constraint, or should we match the search bar's existing URL param length limits? — *Owner: Engineering*
- **Risk:** URL params serialization format may change if the search feature evolves. Saved searches that captured old param formats could become invalid. — *Mitigation: Add a `version` field to `SavedSearch` now, even if we don't use it in MVP, so we can handle migrations later.*
- **Tradeoff:** Using Supabase client directly (no API route) is faster to build but means Supabase credentials are needed client-side. This is the existing pattern in the app, so no new risk introduced.

---

### 15. Rollout & Next Steps

**MVP scope:** Save, list, apply, rename, and delete saved searches. Server-persisted. No sharing.
- Includes: all 6 user stories above
- Excludes: team/shared saved searches, folders, search history, bulk operations

**Phase 2+ ideas:**
- Pin saved searches to the top of the list
- Share a saved search via a link or with specific teammates
- Organize saved searches into folders
- Search within saved searches (for users with many)
- Keyboard shortcut to apply a saved search by number (⌘1, ⌘2…)

**Sign-off needed from:**
- [ ] PM — confirm success metrics baseline
- [ ] Engineering lead — confirm Supabase RLS approach and migration process
- [ ] Design — confirm sidebar interaction pattern and component designs

**Next steps:**
1. Share this PRD for async review — *Owner: Product Designer, by [date]*
2. Engineering scoping + story pointing — *Owner: Engineering lead, by [date]*
3. Figma designs for all screens/states — *Owner: Designer, by [date]*
4. Supabase migration written and reviewed — *Owner: Backend engineer, by [date]*
