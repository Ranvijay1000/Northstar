# Northstar — Class 10 Student OS

A local-first student workspace with a free core and a separately configured optional AI layer.

## Netlify deployment

The interface uses HTML, CSS and vanilla JavaScript. Netlify deployment is configured in `netlify.toml`; see [DEPLOY.md](DEPLOY.md) for publishing and optional hosted AI setup.

## Login and cloud saving

A login screen now supports email/password accounts, password reset and guest access. Connect your Firebase project using [FIREBASE.md](FIREBASE.md). Signed-in workspaces save automatically to Firestore and load on other devices. Guest records stay local and separate.

## Run
Requires Node.js 20 or later. No npm dependencies or paid subscriptions are needed.

Double-click **Start Northstar.cmd**, then open **http://127.0.0.1:3000**.
Or run these commands in this directory:

    npm start

Checks:

    npm run check
    npm test

The app runs on loopback only. Keep the terminal open while using it. To stop it, press Ctrl+C.

## Included
- Home dashboard: daily time, streak, syllabus, readiness, ranked next steps, homework, exams, school timetable.
- Planner: homework and assignments with edit/delete/status; tests and marks; exams with editable chapter selection; current-month calendar; configurable Monday–Saturday timetable.
- Learn: editable subjects and chapters, ordered lecture lists, resource links, notes, completion dates.
- Syllabus: five chapter stages, lecture and practice progress, deterministic readiness, spaced revision.
- Practice: a small saved question bank, custom questions, self-assessment, mistake notebook, optional AI oral test with typed answers.
- Progress: study activity, marks history, subject readiness, weak chapters, weekly summary, mistake patterns.
- Revision vault: searchable editable formulas, favorites, revised status.
- Standard study planning and syllabus feasibility calculations.
- Profile, light/dark/system appearance, goals, backup/restore and separately confirmed resets.
- Service worker caches app files for offline use after a successful first visit.
- Optional browser WebMCP tools where supported.

The first visit uses an editable sample workspace. Choose **Profile → Start fresh** to clear its study records while retaining the starter chapter list. The chapter list is an editable starting point, not an official syllabus certification; adjust it to your school's board, Hindi course and academic year. Sample dates are relative to the first visit.

## Student data
Records are stored in this browser's localStorage, separately per account. Signed-in records also sync to Firestore when Firebase is configured. The running focus timer uses sessionStorage and wall-clock time, so switching views and refreshing the tab retain it. Finish sessions to save study time, including partial minutes. Each browser/origin has separate data. Browser clearing can erase data: export backups regularly. Restore validates before replacement, and asks for confirmation. No student data is sent to AI unless you explicitly use an enabled AI action. No API keys are stored in readable browser study data or backups. Hosted AI uses a separately encrypted HttpOnly session cookie; see DEPLOY.md.

Firebase provides optional accounts, authentication, sign-out and study-data synchronization after configuration. Voice recognition and push notifications are not included. In-app deadlines remain available. Local profile deletion is distinct from study-data deletion.

## Algorithms
Readiness uses 15% chapter learning stage, 25% lecture completion, 20% practice, 20% revision completion and 20% recent marks. Missing evidence contributes zero. Each open mistake deducts 4 points (maximum 18); revision age beyond 14 days deducts up to 15 points. Scores are clamped to 0–100. Weights can be changed in public/core.js. These are preparation estimates, not predicted board marks.

Priority uses homework urgency, exam/test urgency, low readiness, recent marks, overdue reviews, unfinished lectures and open mistakes. Standard planning fills available minutes with these tasks and 5-minute breaks. Workload estimates count unfinished lecture minutes, up to 60 practice minutes and up to three 20-minute reviews per chapter; a chapter with no lectures starts with a 60-minute learning estimate.

Revision intervals after reviews are 3, 7, 14 and 30 days, then 30 days for subsequent reviews. Week summaries count chapters reviewed; all review counts remain on chapters.

## Optional AI on the local server
In **Profile > Optional AI**, enter your OpenAI API key and model, then choose **Save & connect**. The server verifies model access, saves the key privately in .env, and activates AI immediately. Existing configuration is kept if validation or saving fails. The key is never returned by the server or included in browser storage/backups. **Disconnect AI** clears the configured key and disables the provider.

Alternatively, copy .env.example to .env and configure on the SERVER:

    AI_ENABLED=true
    AI_PROVIDER=openai
    OPENAI_API_KEY=your_key
    AI_MODEL=your_supported_model

Restart the server, then enable AI in Profile. The server reads .env without dependencies. The .env file is excluded from source control; never share it. AI is disabled by default and requires API access, not ChatGPT Plus. Configure question/answer limits and daily usage via the documented environment variables.

The provider abstraction in server/ai.mjs defines generateQuestions, evaluateAnswer, explainConcept, generateStudyPlan and analyzePerformance. This version's UI connects the optional oral test (question generation + answer evaluation). Other AI entry points and additional providers are extension contracts, not implemented UI features.

AI errors leave core tools and student data intact. Oral-test failures offer reference-answer self-assessment. Requests time out. Responses are cached for 10 minutes. Usage limits are per local server IP and reset on server restart. The Netlify adapter isolates visitor credentials in encrypted browser sessions. Its per-instance quotas are not durable billing controls; see DEPLOY.md. The server rejects cross-origin AI requests, caps request size, sets security headers and never returns credentials.

## Architecture
public/core.js — data model, linked records, algorithms, seed and backup validation.
public/app.js — seven views and interactions; accesses AI only via /api.
public/styles.css — responsive design and theme tokens.
public/sw.js — offline app-shell caching.
server/index.mjs — local static server and API boundary.
server/ai.mjs — central provider abstraction, limits, timeout, cache and failure handling.
tests/core.test.mjs — domain correctness and AI-off tests.

This is a complete local MVP with several version-2 study tools. Netlify deployment is supported. Firebase accounts and synchronization are included; later voice/adaptive AI features require further work.

## Verification
JavaScript syntax and domain tests pass. Browser checks covered mobile/desktop rendering, homework persistence across reload, marks bounds, lecture completion updates, saved-question practice, exam feasibility, analytics and timer persistence. Browser tool registration, a valid shared-state action and intentional invalid-ID failure were checked. Live paid-provider requests were not tested because no AI key is configured.

## Dashboard synchronization and session progress
Home and Progress recalculate after every saved session, import and relevant record edit. Browser storage events synchronize open tabs, and returning to the tab refreshes derived totals. The dashboard date updates across midnight.

Choose Learning, Practice or Revision before starting. Learning sessions contribute to learning progress using the chapter's total lecture duration (minimum 60 minutes). Practice credits 100% per 60 minutes, capped at 100%. Every accumulated 20 revision minutes records one spaced review. Syllabus completion combines lecture/learning progress (50%), practice (25%) and revisions (25%), with the manually selected chapter stage as a floor. Readiness uses the existing weighting and includes learning-session evidence. Time alone never awards test marks or completes lecture checkboxes. High-progress chapters may show no percentage change until another preparation factor improves. Session IDs prevent repeat saves after refresh.

Import/export is available on Home and under Profile > Your data. Backups include session activity and fractional minutes; server credentials stay separate.

The local settings endpoint accepts only same-origin JSON requests with a loopback Host header. Key validation uses the documented OpenAI model retrieval endpoint: https://developers.openai.com/api/reference/resources/models/methods/retrieve . A successful connection confirms key/model access; provider billing limits can still affect later generation.

The latest changes include mocked-provider connection tests and session/backup integration tests. A real API key is required to test live generation. Restart Start Northstar.cmd after updating server code.

## Sample papers
Practice > Sample Papers logs one record per attempted paper, with subject, date, required marks obtained / total marks and required minutes taken. Paper name and major mistake topics are optional. Counts by subject, average percentage and paper time update immediately. Edit or delete records from paper history. Records are included in backups and Firebase synchronization. Paper time stays separate from focus-session time to avoid duplicate study totals; subject-wide scores do not fabricate chapter readiness.

## Add your syllabus
Open Syllabus > Add syllabus. Choose an existing subject or create one, then paste chapter names one per line and save. Existing progress is preserved, duplicate chapter names are skipped, and the subject is included in your dashboard. Chapters also appear in Learn and are included in backups and Firebase synchronization.
