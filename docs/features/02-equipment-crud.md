# Feature Shape: Equipment CRUD

## Problem

Supervisors and operators need to know what equipment exists in the fleet, but there's no way to
manage the equipment catalog. Admins need to add new forklifts, update their information, and
deactivate equipment that's no longer in service.

Without equipment data, operators cannot perform inspections.

## Solution (Broad Strokes)

Admin-only equipment management with the following capabilities:

- **Create** equipment with: name, internal code, CACES category, ownership type
- **Edit** equipment details
- **Deactivate** equipment (soft delete — keeps inspection history intact)
- **Reactivate** deactivated equipment
- **List** all equipment with filtering by status, category, ownership

Equipment has a calculated `status` field that comes from inspections (not this feature), but
defaults to `OK` for newly created equipment.

UI follows the Spark prototype ([fleetchariot](https://github.com/TituxMetal/fleetchariot)) for
visual reference.

## User Flow

### Create Equipment

1. Admin navigates to Equipment Management
2. Admin clicks "Add Equipment" button
3. Form appears with fields:
   - Name (text) — e.g., "Transpalette T100"
   - Code (text) — e.g., "TP-100" (internal identifier)
   - CACES Category (select) — 1, 3, or 5
   - Ownership Type (select) — Propriété, Location courte durée, Location longue durée
4. Admin fills the form and submits
5. Equipment is created with status `OK` and `isActive: true`
6. Admin sees the new equipment in the list

### Edit Equipment

1. Admin clicks on an equipment item in the list
2. Detail view shows current values
3. Admin clicks "Edit"
4. Form is pre-filled with current values
5. Admin modifies fields and saves
6. Changes are persisted

### Deactivate Equipment

1. Admin views equipment detail
2. Admin clicks "Deactivate"
3. Confirmation dialog appears: "This equipment will no longer appear in inspection lists.
   Continue?"
4. Admin confirms
5. Equipment is marked `isActive: false`
6. Equipment disappears from operator's inspection list but remains in history

### Reactivate Equipment

1. Admin filters equipment list to show "Inactive" items
2. Admin clicks on deactivated equipment
3. Admin clicks "Reactivate"
4. Equipment is marked `isActive: true`
5. Equipment reappears in operator's inspection list

### List/Filter Equipment

1. Admin sees all equipment in a table/grid
2. Admin can filter by:
   - Status (OK, À surveiller, Hors service)
   - CACES Category (1, 3, 5)
   - Ownership Type
   - Active/Inactive
3. Admin can search by name or code
4. List shows: Name, Code, Category, Ownership, Status, Last Inspection Date

## Dependencies

**Requires:**

- Auth with Roles (Feature 01) — Admin role must exist
- Role-based route protection

**Enables:**

- Daily Inspection (Feature 07) — Operators need equipment to inspect
- Supervisor Dashboard (Feature 09) — Fleet overview shows equipment
- Inspection flow depends on equipment existing

## What Must Exist (Backend)

### Domain Layer

- **Equipment entity**: Core domain object with business rules
  - Properties: id, name, code, cacesCategory, ownershipType, status, isActive, createdAt, updatedAt
  - Status is read-only in this feature (calculated by inspection results later)
- **Value Objects**:
  - EquipmentId (UUID wrapper)
  - EquipmentCode (validated format, uniqueness)
  - CacesCategory (enum: 1, 3, 5)
  - OwnershipType (enum: property, short_rental, long_rental)
  - EquipmentStatus (enum: ok, warning, out_of_service)
- **Equipment repository interface**: CRUD operations contract
- **Domain exceptions**: EquipmentNotFound, DuplicateEquipmentCode

### Application Layer

- **Create equipment use case**: Validate and persist new equipment
- **Update equipment use case**: Modify equipment details
- **Deactivate equipment use case**: Set isActive to false
- **Reactivate equipment use case**: Set isActive to true
- **Get equipment by id use case**: Single equipment retrieval
- **List equipment use case**: With filtering and pagination
- **DTOs**: CreateEquipmentDto, UpdateEquipmentDto, EquipmentResponseDto, EquipmentListFiltersDto
- **Mapper**: Entity to DTO transformation

### Infrastructure Layer

- **Database schema**: Equipment table with all fields
- **Prisma repository implementation**: Implements domain repository interface
- **Equipment controller**: HTTP endpoints for all operations
- **Admin role guard**: Protect all equipment endpoints

### API Endpoints

- `POST /api/equipment` — Create equipment (admin)
- `GET /api/equipment` — List equipment with filters (admin, supervisor, operator)
- `GET /api/equipment/:id` — Get single equipment (admin, supervisor, operator)
- `PATCH /api/equipment/:id` — Update equipment (admin)
- `PATCH /api/equipment/:id/deactivate` — Deactivate (admin)
- `PATCH /api/equipment/:id/reactivate` — Reactivate (admin)

## What Must Exist (Frontend)

### Pages/Routes

- **Equipment list page** (admin): Table with filters, add button
- **Equipment detail page** (admin): View details, edit/deactivate actions
- **Equipment form page/modal** (admin): Create and edit form

### Components

- **Equipment table**: Sortable, filterable list of equipment
- **Equipment row**: Single equipment with status badge, category badge
- **Equipment form**: Validated form for create/edit
- **Equipment detail card**: Display all equipment properties
- **Status badge**: Color-coded badge (green OK, orange warning, red out of service)
- **Category badge**: CACES category indicator
- **Ownership badge**: Property type indicator
- **Filter bar**: Dropdowns/chips for filtering the list
- **Confirmation dialog**: For deactivate action

### State Management

- **Equipment store**: List of equipment, loading states, filters
- **Current equipment atom**: Selected equipment for detail view
- **Filter atoms**: Active filters for the list

### Validation

- **Equipment schema** (Zod): Validate form inputs
  - Name: required, 2-100 characters
  - Code: required, unique, alphanumeric with dashes, 2-20 characters
  - Category: required, one of 1, 3, 5
  - Ownership: required, one of the three types

## Open Questions

1. Should equipment code be auto-generated or manually entered? (Spark prototype seems manual)
2. Is there a maximum number of equipment items? (Pagination threshold)
3. Should deactivated equipment show a different visual treatment in lists?
4. Can equipment be permanently deleted, or only deactivated? (Keeping history suggests no hard
   delete)

## Out of Scope

- Equipment photos/images
- Equipment location/GPS tracking
- Maintenance schedules
- Equipment assignment to specific operators
- Equipment categories beyond CACES (1, 3, 5)
- Bulk import/export of equipment
- Equipment transfer between sites (single-site MVP)
- Status change from this feature (status comes from inspections only)

## Risks / Gotchas

- **Code uniqueness**: Equipment codes must be unique. Need database constraint and validation.
- **Status confusion**: Admins might expect to set status manually. Make it clear status is
  calculated from inspections. New equipment defaults to OK.
- **Cascade on deactivate**: When equipment is deactivated, it should not appear in operator's
  "start inspection" list, but existing inspections remain visible in history.
- **French UI**: All labels, messages, and options must be in French as per MVP requirements.
  - CACES Category labels: "CACES 1", "CACES 3", "CACES 5"
  - Ownership labels: "Propriété", "Location courte durée", "Location longue durée"
  - Status labels: "OK", "À surveiller", "Hors service"
- **Empty state**: What does the admin see when there's no equipment? Need helpful empty state UI.
- **Spark prototype alignment**: Use the prototype as visual reference but don't copy code — rebuild
  with proper architecture.
