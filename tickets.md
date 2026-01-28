# Street League Workshop Management Platform - Implementation Tickets

## Project Overview
Build a full-stack workshop management platform for Street League charity to help them deliver sports and life skills programmes consistently across UK cities. The platform enables two user types: Head Office Staff (approve workshops, view analytics) and Local Academy Staff (create/manage workshops).

**Tech Stack:** Node.js + Express + PostgreSQL + Vanilla JavaScript
**Design:** Bold, energetic youth-focused aesthetic with Street League branding

---

## Epic 1: Project Foundation & Setup

### Ticket 1.1: Initialize Project Structure
**Priority:** P0 (Critical)
**Estimate:** 2 hours

**Description:**
Set up the complete project structure with backend, frontend, and database folders. Initialize Node.js project and install all required dependencies.

**Acceptance Criteria:**
- [ ] Directory structure created matching the plan (backend/, frontend/, database/, docs/)
- [ ] Backend package.json created with all dependencies:
  - express, pg, dotenv, bcrypt, jsonwebtoken, cors, express-validator
  - Dev dependencies: nodemon, jest, supertest
- [ ] .gitignore file created (excludes node_modules, .env, etc.)
- [ ] .env.example created with all required variables documented
- [ ] README.md created with project overview and setup instructions
- [ ] All folders have placeholder files so structure is visible
- [ ] `npm install` runs successfully without errors

**Files to Create:**
- `/backend/package.json`
- `/backend/.env.example`
- `/.gitignore`
- `/README.md`

---

### Ticket 1.2: Database Schema & Migrations
**Priority:** P0 (Critical)
**Estimate:** 3 hours

**Description:**
Create PostgreSQL database schema with users, workshops, and workshop_approvals tables. Write migration scripts and seed data.

**Acceptance Criteria:**
- [ ] `database/schema.sql` contains complete schema:
  - users table (id, email, password_hash, first_name, last_name, role, academy_location, is_active, timestamps)
  - workshops table (id, title, description, age_group, session_count, category, tags, thumbnail_gradient, status, created_by, academy_location, timestamps)
  - workshop_approvals table (id, workshop_id, reviewer_id, action, feedback, created_at)
- [ ] All foreign keys and indexes defined
- [ ] Migration scripts in `database/migrations/` folder (001-004)
- [ ] Seed script creates 2 test users:
  - `hq@streetleague.co.uk` / `password123` (hq_staff role)
  - `academy@streetleague.co.uk` / `password123` (local_academy role)
- [ ] Seed script creates ~20 sample workshops from existing HTML
- [ ] Database can be created with: `createdb streetleague_db`
- [ ] Migrations run successfully: `npm run migrate`
- [ ] Seeds load successfully: `npm run seed`

**Files to Create:**
- `/database/schema.sql`
- `/database/migrations/001_create_users_table.sql`
- `/database/migrations/002_create_workshops_table.sql`
- `/database/migrations/003_create_approvals_table.sql`
- `/database/migrations/004_seed_initial_data.sql`

---

## Epic 2: Backend - Authentication System

### Ticket 2.1: Database Connection & Configuration
**Priority:** P0 (Critical)
**Estimate:** 2 hours

**Description:**
Set up PostgreSQL connection pool and environment configuration for the backend.

**Acceptance Criteria:**
- [ ] `backend/src/config/database.js` exports PostgreSQL pool connection
- [ ] `backend/src/config/environment.js` loads and validates env variables
- [ ] Connection pool uses proper configuration (max connections, timeouts)
- [ ] Database connection can be tested with simple query
- [ ] Error handling for failed database connections
- [ ] Environment validation fails gracefully with clear error messages

**Files to Create:**
- `/backend/src/config/database.js`
- `/backend/src/config/environment.js`

---

### Ticket 2.2: User Model & Authentication Service
**Priority:** P0 (Critical)
**Estimate:** 4 hours

**Description:**
Create User model and authentication service with password hashing, JWT token generation, and user CRUD operations.

**Acceptance Criteria:**
- [ ] `User.js` model with methods:
  - `findByEmail(email)` - returns user or null
  - `findById(id)` - returns user or null
  - `create(userData)` - creates new user with hashed password
  - `updateLastLogin(id)` - updates last_login timestamp
- [ ] Passwords hashed with bcrypt (10 salt rounds)
- [ ] `auth.service.js` with methods:
  - `login(email, password)` - validates credentials, returns JWT token and user
  - `verifyToken(token)` - decodes and validates JWT
  - `hashPassword(password)` - hashes password
  - `comparePassword(plain, hashed)` - compares passwords
- [ ] JWT tokens include payload: userId, email, role, academyLocation
- [ ] Token expiry set to 1 hour
- [ ] Unit tests pass for all methods

**Files to Create:**
- `/backend/src/models/User.js`
- `/backend/src/services/auth.service.js`

---

### Ticket 2.3: Authentication Middleware & Controllers
**Priority:** P0 (Critical)
**Estimate:** 4 hours

**Description:**
Implement JWT authentication middleware and authentication endpoints (login, logout, me, refresh).

**Acceptance Criteria:**
- [ ] `auth.js` middleware verifies JWT from Authorization header
- [ ] Middleware attaches decoded user to `req.user`
- [ ] Middleware returns 401 for invalid/missing tokens
- [ ] Role-based middleware `requireRole(['hq_staff'])` checks user role
- [ ] `auth.controller.js` implements:
  - `POST /api/auth/login` - returns token and user data
  - `POST /api/auth/logout` - clears session (future: blacklist token)
  - `GET /api/auth/me` - returns current user info (requires auth)
  - `POST /api/auth/refresh` - refreshes access token
- [ ] Login validates email and password
- [ ] Login returns 401 for invalid credentials
- [ ] All endpoints have proper error handling
- [ ] Routes mounted in `auth.routes.js`

**Files to Create:**
- `/backend/src/middleware/auth.js`
- `/backend/src/controllers/auth.controller.js`
- `/backend/src/routes/auth.routes.js`

---

## Epic 3: Backend - Workshop Management

### Ticket 3.1: Workshop Model
**Priority:** P0 (Critical)
**Estimate:** 4 hours

**Description:**
Create Workshop model with CRUD operations and status management.

**Acceptance Criteria:**
- [ ] `Workshop.js` model with methods:
  - `findAll(filters)` - returns workshops with optional filters (status, category, ageGroup, createdBy)
  - `findById(id)` - returns workshop with creator info
  - `create(workshopData)` - creates workshop with status 'draft'
  - `update(id, updates)` - updates workshop fields
  - `delete(id)` - deletes workshop (only if status is 'draft')
  - `updateStatus(id, status, userId)` - changes workshop status
- [ ] All queries use parameterized statements (SQL injection protection)
- [ ] Creator and approver joined in `findById` query
- [ ] Tags stored as PostgreSQL array
- [ ] Timestamps auto-managed (created_at, updated_at)

**Files to Create:**
- `/backend/src/models/Workshop.js`

---

### Ticket 3.2: Workshop Controllers & Routes
**Priority:** P0 (Critical)
**Estimate:** 5 hours

**Description:**
Implement workshop CRUD endpoints with proper authorization and validation.

**Acceptance Criteria:**
- [ ] `workshops.controller.js` implements:
  - `GET /api/workshops` - list workshops (filtered by role: HQ sees all, Local Academy sees own)
  - `GET /api/workshops/:id` - get workshop details
  - `POST /api/workshops` - create workshop (Local Academy only)
  - `PUT /api/workshops/:id` - update workshop (owner only, draft/rejected status only)
  - `DELETE /api/workshops/:id` - delete workshop (owner only, draft status only)
  - `POST /api/workshops/:id/submit` - submit for approval (owner only, draft status)
  - `GET /api/workshops/my-workshops` - get current user's workshops
- [ ] Validation middleware checks:
  - title: required, min 5 chars, max 255 chars
  - description: required, min 20 chars
  - age_group: required, matches pattern (e.g., "8-12")
  - session_count: required, positive integer
  - category: required, one of valid categories
- [ ] Authorization checks ownership for edit/delete
- [ ] Submit endpoint changes status from 'draft' to 'pending'
- [ ] Returns 403 if user tries to edit approved workshop
- [ ] Routes mounted in `workshops.routes.js`

**Files to Create:**
- `/backend/src/controllers/workshops.controller.js`
- `/backend/src/routes/workshops.routes.js`
- `/backend/src/middleware/validation.js`

---

## Epic 4: Backend - Approval Workflow

### Ticket 4.1: Workshop Approval Model & Service
**Priority:** P1 (High)
**Estimate:** 3 hours

**Description:**
Create WorkshopApproval model to track approval history and audit trail.

**Acceptance Criteria:**
- [ ] `WorkshopApproval.js` model with methods:
  - `findByWorkshopId(workshopId)` - returns approval history for workshop
  - `create(approvalData)` - creates approval record
  - `getApprovalStats()` - returns stats (total approved, rejected, pending)
- [ ] `workshop.service.js` with methods:
  - `approveWorkshop(workshopId, reviewerId, feedback)` - updates workshop status to 'approved', creates approval record
  - `rejectWorkshop(workshopId, reviewerId, feedback)` - updates workshop status to 'rejected', creates approval record
- [ ] Both methods set submitted_at, approved_at timestamps
- [ ] Transactions used to ensure workshop status and approval record created atomically

**Files to Create:**
- `/backend/src/models/WorkshopApproval.js`
- `/backend/src/services/workshop.service.js`

---

### Ticket 4.2: Approval Controllers & Routes
**Priority:** P1 (High)
**Estimate:** 3 hours

**Description:**
Implement approval endpoints for HQ staff to approve/reject workshops.

**Acceptance Criteria:**
- [ ] `approvals` routes in `workshops.routes.js` (or separate file):
  - `GET /api/approvals/pending` - list pending workshops (HQ staff only)
  - `POST /api/approvals/:workshopId/approve` - approve workshop (HQ staff only)
  - `POST /api/approvals/:workshopId/reject` - reject with feedback (HQ staff only)
  - `GET /api/approvals/:workshopId/history` - get approval history
- [ ] Role middleware ensures only HQ staff can approve/reject
- [ ] Approve/reject endpoints validate feedback (optional for approve, required for reject)
- [ ] Returns 404 if workshop not found
- [ ] Returns 400 if workshop not in 'pending' status
- [ ] Approval history includes reviewer name, action, feedback, timestamp
- [ ] Email notification placeholder (console.log for MVP)

**Files to Create:**
- `/backend/src/controllers/workshops.controller.js` (add approval methods)
- `/backend/src/routes/workshops.routes.js` (add approval routes)

---

## Epic 5: Backend - Analytics & Dashboard

### Ticket 5.1: Analytics Service & Endpoints
**Priority:** P1 (High)
**Estimate:** 4 hours

**Description:**
Create analytics endpoints for dashboard statistics (role-based data).

**Acceptance Criteria:**
- [ ] `analytics.service.js` with methods:
  - `getDashboardStats(userId, role)` - returns stats based on role
  - `getWorkshopStats()` - returns category breakdown, status counts
  - `getCategoryBreakdown()` - returns workshops per category
- [ ] `analytics.controller.js` implements:
  - `GET /api/analytics/dashboard` - role-based dashboard data
    - HQ: total workshops, pending count, approved count, rejected count, category breakdown
    - Local Academy: my workshops count by status, recent submissions
  - `GET /api/analytics/workshops/stats` - workshop statistics (HQ only)
- [ ] Dashboard stats cached for 5 minutes (optional optimization)
- [ ] Routes mounted in `analytics.routes.js`

**Files to Create:**
- `/backend/src/services/analytics.service.js`
- `/backend/src/controllers/analytics.controller.js`
- `/backend/src/routes/analytics.routes.js`

---

### Ticket 5.2: Search & Filter Functionality
**Priority:** P2 (Medium)
**Estimate:** 3 hours

**Description:**
Implement search and filter functionality for workshops.

**Acceptance Criteria:**
- [ ] `GET /api/search` endpoint with query params:
  - `q` - text search (searches title, description)
  - `category` - filter by category
  - `status` - filter by status
  - `ageGroup` - filter by age group
- [ ] Text search uses PostgreSQL `ILIKE` for case-insensitive matching
- [ ] Multiple filters can be combined (AND logic)
- [ ] Results paginated (default 20 per page)
- [ ] Returns workshop count and results
- [ ] Search respects user role (HQ sees all, Local Academy sees own)

**Files to Create:**
- `/backend/src/controllers/workshops.controller.js` (add search method)
- `/backend/src/routes/workshops.routes.js` (add search route)

---

## Epic 6: Backend - Error Handling & Validation

### Ticket 6.1: Global Error Handler & Validation Middleware
**Priority:** P1 (High)
**Estimate:** 3 hours

**Description:**
Implement comprehensive error handling and request validation across the API.

**Acceptance Criteria:**
- [ ] `errorHandler.js` middleware catches all errors:
  - Database errors (connection, query failures)
  - Validation errors (400 Bad Request)
  - Authentication errors (401 Unauthorized)
  - Authorization errors (403 Forbidden)
  - Not found errors (404)
  - Server errors (500)
- [ ] Errors formatted consistently:
  ```json
  {
    "success": false,
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Validation failed",
      "details": [{"field": "title", "message": "Title is required"}]
    }
  }
  ```
- [ ] Validation middleware uses express-validator
- [ ] Sensitive info not leaked in production errors
- [ ] All controllers use try-catch with proper error throwing

**Files to Create:**
- `/backend/src/middleware/errorHandler.js`
- `/backend/src/middleware/validation.js` (update with more validators)

---

### Ticket 6.2: Express Server Setup & Route Mounting
**Priority:** P0 (Critical)
**Estimate:** 3 hours

**Description:**
Create main Express server with all middleware, routes, and configuration.

**Acceptance Criteria:**
- [ ] `server.js` sets up Express app with:
  - Body parser (JSON)
  - CORS (configured for frontend origin)
  - Helmet for security headers
  - Morgan for logging (development only)
  - Static file serving for frontend
- [ ] Routes mounted:
  - `/api/auth` → auth routes
  - `/api/workshops` → workshop routes
  - `/api/approvals` → approval routes
  - `/api/analytics` → analytics routes
  - `/api/search` → search route
- [ ] Error handler mounted last
- [ ] 404 handler for undefined routes
- [ ] Server starts on PORT from env (default 3000)
- [ ] Health check endpoint: `GET /api/health` returns `{status: "ok"}`
- [ ] Server can be started with `npm run dev` (uses nodemon)

**Files to Create:**
- `/backend/src/server.js`

---

## Epic 7: Frontend - Authentication UI

### Ticket 7.1: Login Page
**Priority:** P0 (Critical)
**Estimate:** 4 hours

**Description:**
Create login page with form validation and authentication flow.

**Acceptance Criteria:**
- [ ] `frontend/public/login.html` created with Street League design:
  - Logo in header
  - Centered login form
  - Email and password fields
  - "Remember me" checkbox (UI only for MVP)
  - Submit button
  - Error message display area
- [ ] Form validates:
  - Email required and valid format
  - Password required, min 6 characters
  - Shows validation errors inline
- [ ] On successful login:
  - Token stored in localStorage
  - User data stored in localStorage
  - Redirects to appropriate dashboard based on role
- [ ] On failed login:
  - Shows error message ("Invalid email or password")
  - Clears password field
- [ ] Loading state during submission (disable button, show spinner)
- [ ] Styled with Street League branding (bold fonts, blue gradient buttons)

**Files to Create:**
- `/frontend/public/login.html`
- `/frontend/public/js/auth.js` (login logic)

---

### Ticket 7.2: Authentication Helper & API Client
**Priority:** P0 (Critical)
**Estimate:** 3 hours

**Description:**
Create reusable authentication helpers and API client for frontend.

**Acceptance Criteria:**
- [ ] `frontend/public/js/auth.js` exports:
  - `login(email, password)` - calls API, stores token/user
  - `logout()` - clears token/user, redirects to login
  - `getCurrentUser()` - returns user from localStorage
  - `getToken()` - returns token from localStorage
  - `isAuthenticated()` - checks if user is logged in
  - `checkAuth()` - redirects to login if not authenticated
- [ ] `frontend/public/js/app.js` (API client) exports:
  - `api.get(endpoint)` - makes GET request with auth header
  - `api.post(endpoint, data)` - makes POST request with auth header
  - `api.put(endpoint, data)` - makes PUT request
  - `api.delete(endpoint)` - makes DELETE request
  - Automatically adds `Authorization: Bearer <token>` header
  - Handles 401 responses (logout and redirect)
  - Returns JSON response or throws error
- [ ] All API calls use BASE_URL from config (http://localhost:3000)

**Files to Create:**
- `/frontend/public/js/auth.js`
- `/frontend/public/js/app.js`
- `/frontend/public/js/utils.js` (helper functions)

---

## Epic 8: Frontend - Dashboards

### Ticket 8.1: Local Academy Dashboard
**Priority:** P0 (Critical)
**Estimate:** 5 hours

**Description:**
Create dashboard for Local Academy staff showing their workshops and stats.

**Acceptance Criteria:**
- [ ] `frontend/public/dashboard-local.html` displays:
  - Header with logo, user name, logout button
  - Quick stats cards: Total Workshops, Pending Approval, Approved, Rejected
  - "Create Workshop" button (prominent, top-right)
  - Workshop list with filters: All, Draft, Pending, Approved, Rejected
  - Each workshop card shows: thumbnail, title, age group, status badge, edit/delete buttons
- [ ] Status badges color-coded:
  - Draft: gray
  - Pending: yellow/orange
  - Approved: green
  - Rejected: red
- [ ] Edit button only shown for draft/rejected workshops
- [ ] Delete button only shown for draft workshops
- [ ] Clicking workshop card navigates to detail page
- [ ] Empty state shown when no workshops
- [ ] Loading spinner while fetching data
- [ ] Protected route (redirects to login if not authenticated)

**Files to Create:**
- `/frontend/public/dashboard-local.html`
- `/frontend/public/js/dashboard.js`

---

### Ticket 8.2: HQ Staff Dashboard
**Priority:** P0 (Critical)
**Estimate:** 5 hours

**Description:**
Create dashboard for HQ staff showing pending approvals and analytics.

**Acceptance Criteria:**
- [ ] `frontend/public/dashboard-hq.html` displays:
  - Header with logo, user name, logout button
  - Analytics cards: Total Workshops, Pending Approvals, Approved This Month, Category Breakdown
  - "Pending Approvals" section (prominently featured)
  - List of pending workshops with: thumbnail, title, creator, academy location, age group
  - "Approve" and "Reject" buttons on each workshop
  - "All Workshops" section with filters: All, Approved, Rejected, Category filter
- [ ] Clicking approve opens modal/inline form:
  - Optional feedback textarea
  - Confirm approve button
  - Cancel button
- [ ] Clicking reject opens modal/inline form:
  - Required feedback textarea
  - Confirm reject button
  - Cancel button
- [ ] After approval/rejection:
  - Workshop removed from pending list
  - Success toast notification shown
  - Stats updated
- [ ] Protected route (HQ staff only)

**Files to Create:**
- `/frontend/public/dashboard-hq.html`
- `/frontend/public/js/dashboard.js` (update with HQ logic)

---

## Epic 9: Frontend - Workshop Management

### Ticket 9.1: Create Workshop Form
**Priority:** P0 (Critical)
**Estimate:** 5 hours

**Description:**
Create form for Local Academy staff to create new workshops.

**Acceptance Criteria:**
- [ ] `frontend/public/workshop-create.html` displays form:
  - Title input (required, max 255 chars)
  - Description textarea (required, min 20 chars)
  - Age group select (8-12, 9-13, 10-14, 10-15, 10-16, 11-15, 11-16, 12-16, 13-16)
  - Session count input (required, positive integer)
  - Category select (Football, Basketball, Life Skills, Fitness, Team Building, Creative, Health)
  - Tags input (comma-separated or chip input)
  - Thumbnail gradient selector (visual grid of gradient options)
  - "Save as Draft" button
  - "Submit for Approval" button
  - Cancel button (returns to dashboard)
- [ ] Client-side validation:
  - Shows error messages inline
  - Disables submit until form valid
- [ ] Save as Draft:
  - Creates workshop with status 'draft'
  - Redirects to dashboard with success message
- [ ] Submit for Approval:
  - Creates workshop with status 'pending'
  - Redirects to dashboard with success message
- [ ] Form styled with Street League branding

**Files to Create:**
- `/frontend/public/workshop-create.html`
- `/frontend/public/js/workshops.js`

---

### Ticket 9.2: Edit Workshop Form
**Priority:** P1 (High)
**Estimate:** 3 hours

**Description:**
Create form for editing existing workshops (draft/rejected only).

**Acceptance Criteria:**
- [ ] `frontend/public/workshop-edit.html` reuses create form:
  - Pre-fills all fields with existing workshop data
  - Loads workshop on page load
  - Shows "Update Draft" and "Submit for Approval" buttons
  - If workshop is rejected, shows rejection feedback prominently
- [ ] Update Draft:
  - Updates workshop keeping status as 'draft'
  - Redirects to dashboard
- [ ] Submit for Approval:
  - Updates workshop and changes status to 'pending'
  - Redirects to dashboard
- [ ] Cannot edit if workshop is approved or pending
- [ ] Shows 403 error if user doesn't own workshop

**Files to Create:**
- `/frontend/public/workshop-edit.html`
- `/frontend/public/js/workshops.js` (update with edit logic)

---

### Ticket 9.3: Workshop Detail Page
**Priority:** P1 (High)
**Estimate:** 4 hours

**Description:**
Create detail page showing full workshop information and approval controls.

**Acceptance Criteria:**
- [ ] `frontend/public/workshop-detail.html` displays:
  - Large thumbnail with gradient
  - Title, description, age group, session count
  - Category and tags
  - Creator info (name, academy location)
  - Status badge
  - Approval history section (if any)
- [ ] For Local Academy (owner):
  - Edit button (if draft/rejected)
  - Delete button (if draft)
  - "Submit for Approval" button (if draft)
  - Shows rejection feedback if rejected
- [ ] For HQ Staff:
  - Approve button (if pending)
  - Reject button (if pending)
  - Feedback modal on approve/reject
- [ ] Approval history shows:
  - Reviewer name
  - Action (approved/rejected)
  - Feedback (if any)
  - Timestamp
- [ ] Styled with Street League branding

**Files to Create:**
- `/frontend/public/workshop-detail.html`
- `/frontend/public/js/workshops.js` (update with detail logic)

---

## Epic 10: Frontend - Dynamic Index Page

### Ticket 10.1: Refactor Index.html with Dynamic Data
**Priority:** P1 (High)
**Estimate:** 4 hours

**Description:**
Convert static index.html to fetch and display workshops dynamically from API.

**Acceptance Criteria:**
- [ ] Hero section remains static (no changes)
- [ ] Filter bar functional:
  - Clicking filter fetches workshops by category
  - Active filter highlighted
  - "All" filter shows all workshops
- [ ] Workshop sections populated from API:
  - "Trending This Week" - approved workshops, sorted by recent
  - "Life Skills Development" - approved workshops, category equals Life Skills
  - "Sports Development" - approved workshops, category in Football or Basketball
  - "Recently Added" - approved workshops, sorted by created_at descending
- [ ] Only approved workshops shown to public
- [ ] Workshop cards rendered dynamically with data from API
- [ ] Clicking workshop card navigates to detail page
- [ ] Search bar functional:
  - Typing filters workshops by title/description
  - Shows results in real-time (debounced)
- [ ] Loading spinner shown while fetching

**Files to Create:**
- Update `/frontend/public/index.html` (remove static workshop data)
- `/frontend/public/js/workshops.js` (update with dynamic rendering)

---

## Epic 11: UI Polish & User Experience

### Ticket 11.1: Toast Notifications
**Priority:** P2 (Medium)
**Estimate:** 2 hours

**Description:**
Implement toast notification system for user feedback.

**Acceptance Criteria:**
- [ ] `utils.js` exports `showToast(message, type)` function
- [ ] Toast types: success, error, warning, info
- [ ] Toasts appear top-right corner
- [ ] Auto-dismiss after 3 seconds
- [ ] Can manually dismiss with X button
- [ ] Multiple toasts stack vertically
- [ ] Styled with Street League colors:
  - Success: green
  - Error: red
  - Warning: yellow
  - Info: blue
- [ ] Smooth slide-in animation

**Usage Examples:**
- Workshop created: "Workshop created successfully!"
- Workshop approved: "Workshop approved!"
- Login failed: "Invalid email or password"
- Delete confirmed: "Workshop deleted"

**Files to Update:**
- `/frontend/public/js/utils.js`
- `/frontend/public/css/streetleague-design.css`

---

### Ticket 11.2: Confirmation Dialogs
**Priority:** P2 (Medium)
**Estimate:** 2 hours

**Description:**
Implement confirmation modal for destructive actions.

**Acceptance Criteria:**
- [ ] `utils.js` exports `confirmDialog(title, message, onConfirm)` function
- [ ] Modal overlays page with backdrop
- [ ] Shows title, message, Confirm and Cancel buttons
- [ ] Confirm button styled as danger (red) for delete actions
- [ ] Cancel closes modal without action
- [ ] Confirm executes callback then closes
- [ ] ESC key closes modal
- [ ] Click outside modal closes it

**Usage Examples:**
- Delete workshop: "Are you sure you want to delete this workshop? This action cannot be undone."
- Submit for approval: "Submit this workshop for approval? You won't be able to edit it until it's reviewed."
- Reject workshop: "Reject this workshop? The creator will be notified with your feedback."

**Files to Update:**
- `/frontend/public/js/utils.js`
- `/frontend/public/css/streetleague-design.css`

---

### Ticket 11.3: Loading States & Spinners
**Priority:** P2 (Medium)
**Estimate:** 2 hours

**Description:**
Add consistent loading indicators across the application.

**Acceptance Criteria:**
- [ ] `utils.js` exports `showLoader()` and `hideLoader()` functions
- [ ] Full-page loader with backdrop and spinner (for page loads)
- [ ] Button loader (for form submissions):
  - Button disabled
  - Text changes to "Loading..." or shows spinner icon
  - Reverts after API response
- [ ] Content skeleton loaders for workshop cards (gray animated placeholders)
- [ ] Loading spinners match Street League blue brand color
- [ ] Smooth fade-in/out animations

**Files to Update:**
- `/frontend/public/js/utils.js`
- `/frontend/public/css/streetleague-design.css`

---

## Epic 12: Testing & Documentation

### Ticket 12.1: Backend API Tests
**Priority:** P2 (Medium)
**Estimate:** 6 hours

**Description:**
Write unit and integration tests for backend API.

**Acceptance Criteria:**
- [ ] Test suite uses Jest and Supertest
- [ ] Unit tests for:
  - User model methods
  - Workshop model methods
  - Auth service methods
- [ ] Integration tests for:
  - `POST /api/auth/login` - valid/invalid credentials
  - `GET /api/auth/me` - authenticated/unauthenticated
  - `POST /api/workshops` - create workshop with/without auth
  - `PUT /api/workshops/:id` - update own/other's workshop
  - `DELETE /api/workshops/:id` - delete own/other's workshop
  - `POST /api/approvals/:id/approve` - approve as HQ/Local Academy
  - `GET /api/analytics/dashboard` - HQ vs Local Academy stats
- [ ] Test database setup/teardown between tests
- [ ] All tests pass: `npm test`
- [ ] Code coverage >70%

**Files to Create:**
- `/backend/tests/unit/User.test.js`
- `/backend/tests/unit/Workshop.test.js`
- `/backend/tests/integration/auth.test.js`
- `/backend/tests/integration/workshops.test.js`
- `/backend/tests/integration/approvals.test.js`

---

### Ticket 12.2: API Documentation
**Priority:** P2 (Medium)
**Estimate:** 3 hours

**Description:**
Write comprehensive API documentation.

**Acceptance Criteria:**
- [ ] `docs/API.md` documents all endpoints:
  - Method, path, description
  - Request headers (Authorization)
  - Request body schema (JSON)
  - Response schema (success and error)
  - Example requests with curl
  - Example responses
- [ ] Grouped by resource: Auth, Workshops, Approvals, Analytics
- [ ] Error codes documented (400, 401, 403, 404, 500)
- [ ] Authentication flow explained
- [ ] Role permissions clearly stated

**Files to Create:**
- `/docs/API.md`

---

### Ticket 12.3: Setup & Deployment Documentation
**Priority:** P2 (Medium)
**Estimate:** 2 hours

**Description:**
Write setup instructions and deployment guide.

**Acceptance Criteria:**
- [ ] `docs/SETUP.md` includes:
  - Prerequisites (Node.js, PostgreSQL, npm)
  - Installation steps (clone, install deps, setup db, run migrations/seeds)
  - Environment variables explained
  - Running development server
  - Running tests
  - Troubleshooting common issues
- [ ] `README.md` updated with:
  - Project overview
  - Features list
  - Tech stack
  - Quick start guide
  - Links to detailed docs
  - Screenshots (optional)
  - License and contribution info
- [ ] Clear, step-by-step instructions
- [ ] Works for someone setting up project from scratch

**Files to Update:**
- `/docs/SETUP.md`
- `/README.md`

---

## Epic 13: End-to-End Testing & Bug Fixes

### Ticket 13.1: Manual E2E Testing - Local Academy Flow
**Priority:** P1 (High)
**Estimate:** 3 hours

**Description:**
Manually test complete Local Academy user journey and fix bugs.

**Test Scenario:**
1. Login as Local Academy user
2. View dashboard (check stats load correctly)
3. Create new workshop (save as draft)
4. Edit workshop (update fields)
5. Submit workshop for approval
6. Verify status changed to pending
7. Try to edit pending workshop (should be blocked)
8. Logout

**Acceptance Criteria:**
- [ ] All steps complete without errors
- [ ] Data persists correctly
- [ ] UI updates reflect backend changes
- [ ] Error messages clear and helpful
- [ ] Responsive design works on mobile/tablet
- [ ] All bugs found are fixed
- [ ] Test passes on Chrome, Firefox, Safari

---

### Ticket 13.2: Manual E2E Testing - HQ Staff Flow
**Priority:** P1 (High)
**Estimate:** 3 hours

**Description:**
Manually test complete HQ Staff user journey and fix bugs.

**Test Scenario:**
1. Login as HQ Staff user
2. View dashboard (check pending approvals load)
3. View workshop detail from pending list
4. Reject workshop with feedback
5. Verify workshop status changed to rejected
6. View another pending workshop
7. Approve workshop
8. Verify workshop no longer in pending list
9. Check analytics updated correctly
10. Logout

**Acceptance Criteria:**
- [ ] All steps complete without errors
- [ ] Approval/rejection updates database correctly
- [ ] Dashboard stats refresh after actions
- [ ] Feedback saved and displayed correctly
- [ ] Cannot approve already approved/rejected workshop
- [ ] All bugs found are fixed
- [ ] Test passes on Chrome, Firefox, Safari

---

### Ticket 13.3: Cross-Browser & Mobile Testing
**Priority:** P2 (Medium)
**Estimate:** 2 hours

**Description:**
Test application across different browsers and mobile devices.

**Acceptance Criteria:**
- [ ] Tested on browsers:
  - Chrome (latest)
  - Firefox (latest)
  - Safari (latest)
  - Edge (latest)
- [ ] Tested on devices:
  - Desktop (1920x1080)
  - Laptop (1366x768)
  - Tablet (iPad, 768x1024)
  - Mobile (iPhone, 375x667)
- [ ] All core features work on all platforms
- [ ] Layout responsive and looks good on all screen sizes
- [ ] No console errors
- [ ] Performance acceptable (page loads <2s)
- [ ] Any browser-specific bugs fixed

---

## Project Completion Checklist

### MVP Ready When:
- [ ] All P0 (Critical) tickets completed
- [ ] Backend API fully functional
- [ ] Frontend pages implemented
- [ ] Authentication working
- [ ] Workshop CRUD working
- [ ] Approval workflow working
- [ ] Dashboards showing correct data
- [ ] End-to-end flows tested
- [ ] Documentation complete
- [ ] No critical bugs remaining

### Demo Ready When:
- [ ] Sample data seeded
- [ ] UI polished (loading states, toasts, etc.)
- [ ] Responsive design works
- [ ] Can demo full user journey for both personas
- [ ] README has screenshots/demo video

---

## Prompt for Implementation

**Use this prompt to start implementation:**

```
I need you to build a workshop management platform for Street League charity based on the tickets in tickets.md.

Start with the Project Foundation (Epic 1) and work sequentially through the epics. For each ticket:
1. Read the ticket description and acceptance criteria carefully
2. Create the files listed in "Files to Create/Update"
3. Implement all acceptance criteria
4. Test that each criterion is met before marking the ticket complete
5. Move to the next ticket

Focus on getting a working MVP first (all P0 tickets), then enhance with P1/P2 features.

Tech stack: Node.js + Express + PostgreSQL + Vanilla JS
Design: Use existing streetleague-design.css with bold, energetic aesthetic

Ready to start with Ticket 1.1: Initialize Project Structure?
```
