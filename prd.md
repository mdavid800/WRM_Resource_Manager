PRD


# Resource Tracker

## Product & Implementation Specification (JNBP Standard)

This document defines **what** the Resource Tracker app should do and **how** it should be implemented using JNBP standards.

---

# 1. App Overview

### 1.1 Purpose

**Resource Tracker** is a web application for managing people, projects, and time-based allocations. It enables organizations to:

* Understand who is working on what, and when
* Prevent overloading team members and avoid burnout
* Identify under-utilised resources
* Support time-varying project demand (e.g. 100% for early phases, 50% later)
* Plan capacity for upcoming work
* Assign people efficiently using drag-and-drop and matching suggestions

### 1.2 Target Users

1. **Resource Managers** – Optimise team allocation across all projects
2. **Project Managers** – Request and secure resources for their projects
3. **Department Heads** – Assess capacity and utilisation at team/role level
4. **Operations / PMO** – Maintain data quality and resolve conflicts

### 1.3 Key Value Propositions

* Prevent over-allocation
* Spot under-utilised resources
* Handle time-varying demand
* Surface best-fit staffing recommendations
* Provide powerful visual analytics
* Simplify staffing workflows with drag-and-drop allocation

---

# 2. JNBP Technology & Implementation Standards

### 2.1 Core Tech Stack

* **Framework:** Next.js (App Router, TypeScript)
* **UI:** React 18
* **Design System:** `shadcn` components styled with **JNBP_cn**
* **Styling:** Tailwind CSS
* **Database/Backend:** PostgreSQL (e.g. via Supabase or API layer)
* **State Management:** React hooks + custom hooks
* **Date Handling:** `date-fns`
* **Testing:**

  * Vitest
  * React Testing Library
  * All tests under `__tests__/`

### 2.2 TypeScript Rules

* **No `any` usage**
* Use:

  * Explicit interfaces and type aliases
  * Discriminated unions
  * Generics when appropriate
  * `unknown` + narrowing where necessary

### 2.3 Recommended Project Structure

```
src/
  app/
    layout.tsx
    page.tsx                // Dashboard
    resources/
      page.tsx
    projects/
      page.tsx
    analytics/
      page.tsx
    roles/
      page.tsx
  components/
    ui/
    features/
      dashboard/
      resources/
      projects/
      allocations/
      analytics/
      roles/
  hooks/
    useResources.ts
    useProjects.ts
    useAllocations.ts
    useRequirements.ts
    useRequirementTimeSegments.ts
    useTeamRoles.ts
  lib/
    api/
      resources.ts
      projects.ts
      allocations.ts
      requirements.ts
      timeSegments.ts
      roles.ts
    util/
      dates.ts
      formatting.ts
      calculations.ts
      matching.ts
  types/
    resource.ts
    project.ts
    allocation.ts
    requirement.ts
    timeSegment.ts
    role.ts
  __tests__/
    features/
    lib/
    hooks/
```

---

# 3. Domain Model

### 3.1 Resources (Team Members)

Represents a person available for allocation.

```
id: string
name: string
role: string
team: string | null
standardCapacity: number (0–200, default 100)
status: 'active' | 'on_leave' | 'inactive'
createdAt: string
updatedAt: string
```

---

### 3.2 Projects

Represents a project with a timeline and priority.

```
id: string
name: string
startDate: string
endDate: string
owner: string | null
priority: 'low' | 'medium' | 'high' | 'critical'
status: 'planned' | 'active' | 'completed' | 'archived'
description: string | null
createdAt: string
updatedAt: string
```

---

### 3.3 Allocations

Assignment of a resource to a project during a time period with a percentage.

```
id: string
resourceId: string
projectId: string
allocationPercentage: number (0–200)
startDate: string
endDate: string
notes: string | null
createdAt: string
updatedAt: string
```

---

### 3.4 Resource Requirements

Demand placed by a project for a role or skill.

```
id: string
projectId: string
roleNeeded: string
quantity: number
allocationPercentage: number
startDate: string
endDate: string
priority: 'low' | 'medium' | 'high' | 'critical'
status: 'unfulfilled' | 'partially_fulfilled' | 'fulfilled'
notes: string | null
createdAt: string
updatedAt: string
```

---

### 3.5 Requirement Time Segments

Time-varying demand periods.

```
id: string
requirementId: string
startDate: string
endDate: string
allocationPercentage: number
notes: string | null
createdAt: string
updatedAt: string
```

---

### 3.6 Team Roles

Central list of roles used in the system.

```
id: string
name: string
createdAt: string
updatedAt: string
```

---

# 4. Feature Specifications

---

## 4.1 Dashboard

### Purpose

Provide an overview of resource health and utilisation.

### Components

#### Summary Cards

* Total resources
* Active resources
* Projects count
* Allocations count

#### Utilisation Metrics

* Average utilisation (current period)
* Over-allocated resources
* Balanced resources
* Under-utilised resources

#### Alerts & Insights

* Over-allocated resources
* Under-utilised resources
* Capacity by role (progress bars)
* Resources with no allocations

#### Quick Actions

* Navigate to Resources
* Navigate to Projects
* Navigate to Analytics

---

## 4.2 Resource Management

### Resource List View

* Card/grid layout
* Fields shown:

  * Name, role, team
  * Status indicator
  * Standard capacity
  * Current utilisation %
  * Colour-coded utilisation bar
  * Actions (edit/delete)

### Status Indicators

* **Active**: Green
* **On Leave**: Yellow
* **Inactive**: Gray

### Resource Form

* Name (min 2, required)
* Role (required, autocomplete)
* Team (optional)
* Standard capacity (0–200)
* Status (default: active)

### Deletion Rules

* Cannot delete a resource with active allocations

### Drag-and-Drop Support

* Resources can be dragged onto project cards
* Dropping opens Quick Allocation Modal

---

## 4.3 Project Management

### Project List View

* Card layout
* Filters: All / Planned / Active / Completed / Archived
* Each card shows:

  * Status badge
  * Priority
  * Owner
  * Date range
  * Requirements summary
  * Allocation summary
  * Fulfilment status

### Project Form

* Name (required)
* Description (optional)
* Dates (start/end)
* Owner (optional)
* Priority
* Status

### Requirements Section

* Add one or many requirements
* Role, quantity, dates, priority
* Time-varying demand toggle
* Time segment editor:

  * Multiple segments
  * Must not overlap
  * Must be within project timeline
  * Must cover full requirement date range

---

## 4.4 Quick Allocation Modal

### Purpose

Fast creation of allocations with support for time-varying demand.

### Features

#### Context Summary

* Resource details
* Project details
* Timeline

#### Modes

* **Single allocation**

  * One percentage + date range
* **Segmented allocation**

  * Multiple segments
  * Pre-filled if requirement uses time segments

#### Validation & Warnings

* Over-allocation checks
* Date range validation
* Segment checks
* Colour-coded statuses (green/yellow/red)

#### Submission

* Single → one allocation
* Segmented → multiple allocations

---

## 4.5 Resource Matching & Suggestions

### Matching Algorithm

**Score components:**

1. Role match: 40%
2. Availability: 40%
3. Skills/experience placeholder: 20%

### Match Display

* Name, role, utilisation
* Match score badge
* Under- or over-utilisation badges
* Details explaining score

### Actions

* Select Resource → opens Quick Allocation Modal pre-filled

---

## 4.6 Allocation Analytics

### Summary Cards

* Totals
* Average utilisation
* Category breakdown

### Trends Graph

* Utilisation over time
* Over-/under-allocation counts

### Heatmap View

* Resource vs time
* Colour-coded utilisation levels
* Click cell for details

### Timeline / Gantt View

* All allocations on a time axis
* Supports click-to-edit

### Date Controls

* Presets (Month, Quarter, Year)
* Custom range

---

## 4.7 Team Roles Management

### Features

* View roles
* Add/edit roles
* Delete roles (only if not referenced)
* Show usage counts
* Used in:

  * Resource form
  * Requirement form
  * Matching algorithm

---

# 5. Business Rules & Constraints

### 5.1 Allocation Rules

* 0–200% allowed
* > 100% → over-allocated
* > 120% → critical
* Optimal utilisation: 70–90%
* Under-utilised: <50%

### 5.2 Date Rules

* End date > start date
* Allocation dates should overlap project dates
* Segments must not overlap
* Segments must be within requirement timeline

### 5.3 Deletion Rules

* Cannot delete resource with active allocations
* Deleting project cascades allocations and requirements
* Deleting requirement cascades segments
* Cannot delete a role in use

---

# 6. Non-Functional Requirements

### Performance

* Dashboard load: <2s
* Calculations: <1s for ~500 resources
* Filters/search: <500ms

### Scalability

* Up to ~1000 resources
* 500 projects
* 10,000+ allocations

### Accessibility

* WCAG AA
* Keyboard navigation
* ARIA where needed
* Colour-blind friendly palette

### Security

* RLS or equivalent
* Future: RBAC, organisation isolation

---

# 7. Testing Requirements

* **All tests in `__tests__/`**
* Use **Vitest**
* Use **React Testing Library** for UI
* Test:

  * Date utilities
  * Utilisation calculations
  * Matching algorithm
  * Critical forms and modals
  * Drag-and-drop behaviours (where feasible)

---

This is now a clean, complete specification for building the **Resource Tracker** app using **JNBP standards**—with implementation rules, feature definitions, data models, business constraints, and testing expectations all integrated.
