# **Frontend Engineer Assignment (No Code, High-Quality UI Focus)**

## **Evaluation Criteria**

* Frontend tech selection and reasoning (framework, state, data fetching)
* UI architecture (routing, component design, reusable patterns)
* API calling strategy (error handling, retries, abort, pagination)
* Browser-level caching + offline-friendly patterns
* Debugging + observability (logging, tracing, error boundaries)
* Security basics on client (token handling, safe downloads, XSS considerations)
* UX quality for async jobs (progress, partial results, resilience)

---

## **Problem 1: Video-to-Notes Platform (Frontend System Design)**

**Goal:** Upload video → job runs async → user sees status + outputs: Summary.md, highlights (timestamps), assets. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

**Your solution must include**

* **Screens:** Upload, Jobs list, Job detail (status/logs), Results (markdown + highlights)
* **UI states:** loading, queued, processing, success, failed, retry, partial output
* **API calling plan:** how you poll/stream job progress (polling vs SSE), abort on navigation
* **Caching:** what to cache in browser (job list, job detail, results), TTL strategy, invalidation
* **Debugging plan:** how you would debug “stuck processing” from frontend side (network logs, correlation id display)

**Your Solution for problem 1:**

1. Architecture Overview

The platform will be built as a Single Page Application (SPA) that interacts with an async job-based backend. When a user uploads a video, the backend creates a processing job and returns a jobId. The frontend will track job progress via controlled polling (every 5 seconds) and render results (Summary.md, highlights, assets) once processing completes. Markdown output will be safely rendered using a sanitized markdown renderer.

**2. Screens & UI Structure**
**2.1 Upload Screen**

Purpose: Allow user to upload a video and initiate processing.

**Components:**

Drag & drop upload area

File picker button

File size validation (e.g., max 500MB)

File type validation (mp4, mov, mkv)

Upload progress bar

Start Processing button

Inline validation errors

**UX Behavior:**

Disable Start button until file passes validation

Show upload percentage + speed

Show clear error messages for network or validation failures

**2.2 Jobs List Screen**

Purpose: Display all video processing jobs.

**Components:**

Table with:

Job ID

File name

Status badge (Queued / Processing / Completed / Failed)

Created time

Last updated time

Action (View / Retry)

Auto refresh indicator

Manual refresh button

**UX Behavior:**

Color-coded status badges

Failed jobs show Retry button

Clicking row navigates to Job Detail screen

**2.3 Job Detail Screen**

Purpose: Show live progress of selected job.

**Components:**

Status badge

Progress bar (0–100%)

Logs panel (scrollable)

Cancel button (if allowed)

Partial output preview section

**UX Behavior:**

Polling indicator visible

Logs auto-scroll toggle

If partial output available, display preview while still processing

**2.4 Results Screen**

**Purpose:** Display final processed output.

**Components:**

Rendered Markdown (Summary.md)

Highlights list (clickable timestamps)

Asset preview (thumbnails)

Download buttons

“Download All” option

**UX Behavior:**

Timestamp click seeks video

Lazy load asset previews

Use signed URLs for secure downloads

**3. UI States**

The system supports the following UI states:

**Idle**: No upload started. Upload area visible.

**Uploading**: Progress bar active, controls disabled.

**Queued**: Job created, waiting for backend processing.

**Processing**: Progress updating, logs visible.

**Partial Output**: Some results visible while remaining processing continues.

**Success**: Full results rendered. Polling stops.

**Failed**: Error message shown. Retry option available.

**Retrying:** Previous state cleared, polling restarts.

**4. API Calling Strategy**
**Upload Flow**

POST /upload → returns jobId

Redirect to /jobs/:id

**Job Tracking**

GET /jobs

GET /job/:id

**Polling Strategy**

Poll every 5 seconds

Stop polling when status = success or failed

Use AbortController to cancel polling when navigating away

**Retry Strategy**

Automatic retry for network errors (max 3 attempts)

Exponential backoff (2s → 4s → 8s)

Do not retry for validation (4xx) errors

**Auth Handling**

On 401 → clear session → redirect to login

**5. Browser Caching Strategy**

Cache _GET /jobs_ for 30 seconds (memory cache)

Do not cache processing jobs

Cache completed job results in IndexedDB

Invalidate cache if job retried

Refetch job detail on window focus

**6. Debugging & Observability**

If job appears stuck in “processing”:

Inspect polling requests in Network tab

Verify status updates in response

Display backend correlation ID in UI

Log frontend timestamps of last poll

Provide “Report Issue” button including:

jobId

correlationId

last API response

---

## **Problem 2: LinkedIn Automation Platform (Frontend System Design)**

**Goal:** Connect LinkedIn → persona setup → draft preview → approve → schedule → posting history. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**Your solution must include**

* **Screens:** Connect, Persona editor, Drafts (3 variants), Approval, Scheduler, Post history
* **Form UX:** persona inputs validation, topic input rules, guardrails for scheduling
* **API calling:** draft generation request lifecycle, optimistic UI vs strict confirmation
* **Caching:** drafts caching, schedule list caching, refetch triggers after approval/post
* **Debugging:** how you surface posting failures to user and capture details for support

**Your Solution for problem 2:**

**1. Screens & Flow**

* Connect Screen
* OAuth connect button
* Connection status indicator
* Error handling for auth failure

**Persona Editor**

* Input fields: tone, industry, audience, goals
* Character limit indicators
* Inline validation
* Save persona button

**Draft Preview Screen**

* Show 3 AI-generated variants
* Loading skeleton while generating
* Regenerate option
* Edit draft option

**Approval Screen**

* Approve / Reject button
* Edit before approve
* Confirmation modal

**Scheduler**

* Date/time picker
* Prevent scheduling in past
* Timezone display
* Conflict warning

**Post History**

* Table view of posts
* Status badge (Scheduled / Posted / Failed)
* Retry option for failed posts

**2. Form UX**

* Required field validation
* Character count for topic input
* Disable schedule button if invalid
* Show inline validation errors
* Confirm dialog before scheduling

**3. API Lifecycle**

* _POST /generate-draft_
* Show loading state
* Disable regenerate button during request
* On success → display 3 drafts
* On approve → POST /approve
* On schedule → POST /schedule
* Refetch post history after scheduling

Optimistic UI used only for draft save.
Scheduling requires strict backend confirmation.

**4. Caching Strategy**

* Cache persona config in memory
* Cache drafts temporarily (TTL 10 mins)
* Refetch schedule list after approval
* Cache post history (TTL 30 sec)

**5. Debugging Strategy**

* Display posting failure reason
* Show backend error message
* Include correlation ID
* Provide retry option
* Log failure event for support review

---

## **Problem 3: DOCX Template → Bulk Generator (Frontend System Design)**

**Goal:** Upload template → review fields → single generate → bulk via CSV → ZIP download + per-row report. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

**Your solution must include**

* **Screens:** Template upload, Field review/editor, Single fill form, Bulk upload, Bulk run status, Report table, Downloads
* **Field UI:** field types (text/number/date), required/default, inline validation
* **Bulk UX:** CSV upload constraints, mapping UI (optional), progress + partial success
* **Browser caching:** template metadata caching, field schema caching, bulk report pagination caching
* **Downloads:** safe download UX (signed URL flow assumed), progress indicator

**Your Solution for problem 3:**

**1. Screens**

**Template Upload**

* Upload DOCX
* Show parsing progress

**Field Review Screen**

* Display detected fields
* Field type dropdown (text/number/date)
* Required toggle
* Default value input

**Single Fill Form**

* Render dynamic form from schema
* Inline validation
* Generate button

**Bulk Upload**

* CSV upload
* Validate file structure
* Show preview rows

**Bulk Run Status**

* Progress bar
* Success/failed count
* Cancel option

**Report Table**

* Paginated results
* Per-row status
* Error reason column

**Downloads**

* Download ZIP button
* Individual file download

**2. Field UI**

* Field types selectable
* Required field validation
* Default value fallback
* Inline validation errors

**3. Bulk UX**

* CSV size limit
* Show column mapping UI (optional)
* Show partial success count
* Allow download of error report CSV

**4. Browser Caching**

* Cache template metadata in memory
* Cache field schema in localStorage
* Cache bulk report pages (TTL 1 min)

**5. Download Handling**

* Use signed URLs
* Show download progress
* Disable button while generating
* Handle expired URL errors gracefully

---

## **Problem 4: Character-Based Video Series Generator (Frontend System Design)**

**Goal:** Define characters once → create episode from story → view episode package (script/scenes/assets/render plan). [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

**Your solution must include**

* **Screens:** Character library, Relationship editor, Episode creator, Episode detail (scenes), Asset gallery
* **Consistency UX:** show “locked character profile” per episode, version badges
* **API calling:** long-running generation job UI (progress, resume)
* **Caching:** character library caching, episode package caching, asset thumbnails caching

**Your Solution for problem 4:**

**1. Screens**

**Character Library**

* Character list
* Create/edit character
* Version badge

**Relationship Editor**

* Visual mapping between characters
* Save relationship config

**Episode Creator**

* Story input field
* Select characters
* Generate episode button

**Episode Detail**

* Scenes list
* Script preview
* Render plan view

**Asset Gallery**

* Image thumbnails
* Video preview
* Download buttons

**2. Consistency UX**

* Lock character profile version per episode
* Show version badge
* Warn if character updated after episode created

**3. API Strategy**

* POST /generate-episode
* Long running job
* Show progress bar
* Allow resume if page refreshed
* Poll every 5 seconds

**4. Caching**

* Cache character library (TTL 1 min)
* Cache episode package once completed
* Lazy load and cache thumbnails

---

## **Cross-Cutting** 

Answer these in **bullet points** (max 1 page total):

1. **Frontend stack choice**

* EDIT YOUR ANSWER HERE: Framework (Next.js/Vue/etc), state management, router, UI kit, why.
* `*Framework: Next.js (App Router)
* State Management: Zustand
* Data Fetching: React Query
* UI Kit: Tailwind CSS
* Why:
* Built-in routing
  * Good performance
  * Strong caching via React Query
  * Scalable structure`

2. **API layer design**

* Fetch/Axios choice, typed client generation (OpenAPI), error normalization, retries, request dedupe, abort controllers.
  `
* Axios with interceptors
* Central error normalization
* Auto retry (max 3 attempts)
* AbortController for cancellation
* Request deduplication via React Query`

3. **Browser caching plan**

* What you cache (GET responses, derived state), where (memory, IndexedDB, localStorage), TTL/invalidation rules.
* How you handle “job status updates” without stale UI.
  `
* Memory cache for GET responses
* IndexedDB for large results
* localStorage for lightweight config
* TTL-based invalidation
* Refetch on window focus
* No cache for active processing jobs`

4. **Debugging & observability**

* Error boundaries, client-side logging approach, correlation id propagation, “report a problem” payload.
* How you would debug: slow uploads, failed downloads, intermittent 500s.
  `
* Global error boundary
* Client-side logging
* Correlation ID display
* “Report Problem” modal with:
  * jobId
  * API response
  * timestamp
* Debug slow uploads via Network tab
* Debug 500 errors via response logs`

5. **Security basics**

* Token storage approach, CSRF considerations (if cookies), XSS avoidance for markdown rendering, safe file download patterns.
  `
* Prefer HttpOnly cookies for tokens
* If JWT used, store in memory
* Sanitize markdown before rendering
* Use signed URLs for downloads
* Protect against CSRF if cookie-based auth used`
