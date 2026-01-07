# Feature Shape: Auth with Roles

## Problem

The current authentication system has no role differentiation. All authenticated users have the same
access level. The application needs to distinguish between four user types (user, operator,
supervisor, admin) to control who can perform inspections, view fleet data, or manage the system.

Without roles, public registration would allow anyone to pollute inspection data.

## Solution (Broad Strokes)

Extend the existing Better Auth setup with a simple role system:

- Add a `role` field to the User model with four possible values: `user`, `operator`, `supervisor`,
  `admin`
- New registrations default to `user` role (no feature access)
- Only admins can promote users to other roles
- Role-based route guards on both backend and frontend
- Role-based UI: each role sees a different navigation and dashboard

The role is a simple enum — no complex permission tables or role hierarchies.

## User Flow

### Registration Flow

1. User registers via the existing signup form
2. Account is created with `role: user`
3. User receives email verification (existing flow)
4. After verification, user can log in but sees a "pending approval" screen
5. Admin sees new users in a user management screen
6. Admin promotes user to `operator`, `supervisor`, or `admin`
7. User refreshes or logs in again and now sees their role-appropriate dashboard

### Login Flow (by role)

**User (unpromoted):**

1. Logs in successfully
2. Sees a message: "Your account is pending approval. Contact your administrator."
3. No access to any features

**Operator:**

1. Logs in successfully
2. Redirected to operator dashboard (equipment list for inspection)
3. Navigation shows: Dashboard, My Inspections
4. Cannot access admin or supervisor routes

**Supervisor:**

1. Logs in successfully
2. Redirected to supervisor dashboard (KPIs, fleet overview)
3. Navigation shows: Dashboard, Fleet Status, Inspection History
4. Cannot access admin routes

**Admin:**

1. Logs in successfully
2. Redirected to admin dashboard
3. Navigation shows: Dashboard, Users, Equipment, Questions
4. Full access to all routes

### Role Change Flow

1. Admin navigates to User Management
2. Admin sees list of all users with their current role
3. Admin clicks on a user to view details
4. Admin selects new role from dropdown
5. System saves the change
6. Affected user sees new UI on next page load/login

## Dependencies

**Requires:**

- Existing Better Auth setup (already in place)
- Existing User entity and repository (already in place)
- Existing frontend auth flow (already in place)

**Enables:**

- Equipment CRUD (admin-only access)
- Question Management (admin-only access)
- Daily Inspection (operator access)
- Supervisor Dashboard (supervisor access)
- All role-protected features in the MVP

## What Must Exist (Backend)

### Domain Layer

- **Role enum/type**: Four values (`user`, `operator`, `supervisor`, `admin`)
- **User entity extension**: Add `role` field with default value `user`
- **Domain validation**: Role must be one of the four valid values

### Application Layer

- **Update user role use case**: Admin can change any user's role
- **Get users list use case**: For admin user management (extend existing if present)
- **Role validation in use cases**: Check caller's role before allowing operations

### Infrastructure Layer

- **Database migration**: Add `role` column to User table with default `user`
- **Role guard/decorator**: Protect routes by required role(s)
- **Current user decorator extension**: Include role in the extracted user data
- **User controller updates**: Endpoints for listing users, updating roles

### API Endpoints

- `GET /api/users` — List all users (admin only)
- `PATCH /api/users/:id/role` — Update user role (admin only)
- Existing endpoints should respect role guards where needed

## What Must Exist (Frontend)

### Pages/Routes

- **Pending approval page**: Shown to `user` role after login
- **Role-specific dashboards**: Redirect based on role after login
- **User management page** (admin): List users, change roles

### Components

- **Role-based navigation**: Different nav items per role
- **User list component**: Table of users with role badges
- **Role selector component**: Dropdown to change user's role
- **Pending approval message**: Informational component for unpromoted users

### State Management

- **User store extension**: Include role in the stored user data
- **Role-based computed states**: `$isAdmin`, `$isOperator`, `$isSupervisor`, `$canAccessFeatures`

### Route Protection

- **Frontend middleware update**: Check role before rendering protected pages
- **Redirect logic**: Route users to appropriate dashboard based on role
- **Unauthorized page**: Shown if user tries to access forbidden routes

## Open Questions

1. Should admin be able to demote themselves? (Risk of locking out all admins)
2. Should there be a "super admin" or seed admin that cannot be demoted?
3. When admin changes a user's role, should the user be notified (email)?
4. Should the pending approval screen show any useful info (who to contact)?

## Out of Scope

- Multiple roles per user
- Fine-grained permissions beyond role-based access
- Role hierarchy (supervisor is not "above" operator for permissions)
- Self-service role requests
- Audit log of role changes
- Bulk role assignment

## Risks / Gotchas

- **Session sync**: When admin changes a user's role, the user's session may still have the old
  role. Consider forcing a session refresh or showing stale role until re-login.
- **First admin problem**: After fresh deploy, how does the first admin get promoted? Need a seeding
  strategy (env var, CLI command, or first user becomes admin).
- **Better Auth integration**: The existing admin plugin may conflict or overlap. May need to
  disable it or align with the custom role system.
- **Frontend/backend sync**: Role checks must happen on BOTH sides. Frontend guards for UX, backend
  guards for security.
- **Migration for existing users**: If there are existing users in the database, they'll need a
  default role assigned.
