# HCM Lite — Modern Human Capital Management Web App
**Saylani Hackathon — Beginner Track**

HCM Lite is a modern, responsive Human Capital Management (HR) web application built entirely with **pure HTML5, CSS3, and Vanilla JavaScript (ES6+)**. It requires **no backend, no npm packages, and no build tools**. All application state and relational business records are securely persisted directly in the browser's `localStorage`.

---

## 🚀 Live Demo & How to Run

### Option 1: Direct File Open
Simply double-click or open `index.html` directly in any modern web browser (Google Chrome, Microsoft Edge, Firefox, or Safari).

### Option 2: Simple Local Static Server
If you prefer running via a local HTTP server:
```bash
# Python 3
python -m http.server 8000

# or Node npx (http-server)
npx http-server . -p 8000
```
Then navigate to `http://localhost:8000` in your browser.

### Option 3: Cloud Deployment (GitHub Pages / Netlify / Vercel)
The application uses strict **relative paths** everywhere (`../css/style.css`, `../js/...`, etc.), meaning it works seamlessly out-of-the-box when deployed to:
- **GitHub Pages**: Push repo & enable GitHub Pages on `main` root.
- **Netlify**: Drag-and-drop the project folder to Netlify Drop.
- **Vercel**: Deploy repository with default static settings.

---

## 🌟 Key Features

1. **Zero-Seed, Clean State**:
   - The app starts **completely empty**. There is zero hardcoded or dummy seed data. Every single number and record is generated through real user actions.
2. **First-Run Setup Wizard**:
   - On initial launch, the system detects that no organization is configured and presents an onboarding wizard to set up:
     - Company Name
     - Super Admin Account (Full Name, Email, Password)
     - Office Start Time & Grace Minutes (e.g. 09:00 + 15 mins)
     - Annual Paid Leave Quota (e.g. 12 days)
     - Designated Company Working Days (Monday through Friday)
3. **Role-Based Access Control & Strict Route Guards**:
   - **Super Admin**: Company policies, employee directory CRUD, attendance monitoring, leave approvals/rejections with mandatory comments, executive analytics, company-wide payroll statements, and broadcast announcements.
   - **Employee**: Dedicated self-service workspace, live clock shift check-in/out, today's status badges, attendance history, leave applications with real-time balance rings, personal payslips, and company bulletin feed.
   - Route guards automatically prevent employees from accessing administrative pages and redirect unauthenticated sessions.
4. **Policy-Driven Attendance Engine**:
   - **Late Detection**: Automatically compares check-in time against `startTime + graceMinutes`.
   - **One-Shift-Per-Day**: Restricts employees to one check-in and one subsequent check-out per day.
   - **Half-Day Rule**: Calculates work hours on check-out (`checkOut - checkIn`). If under 5.0 hours, flags as `half_day` while preserving late arrival flags.
   - **Non-Working Day Guard**: Blocks attendance submissions on non-working company days with a friendly notice.
   - **Dynamic Absence Computation**: Working days in the past with no record are dynamically computed as `absent` unless covered by an `approved` leave request ("On leave"). Future days are never marked absent.
5. **Intelligent Leave Management**:
   - Computes only valid company working days between dates (skips weekends and off-days).
   - Validates that date ranges have at least 1 working day (`From <= To`).
   - Prevents overlapping applications with any existing pending or approved requests.
   - Strictly enforces remaining annual quotas (including pending requests in the validation check).
   - Admin approval/rejection modal requiring a mandatory justification comment on rejection.
6. **Payroll & Printable Payslips**:
   - Mathematical formula:
     $$\text{Per Day Rate} = \frac{\text{Monthly Salary}}{\text{Working Days in Month}}$$
     $$\text{Deduction Units} = \text{Absent Days} + (0.5 \times \text{Half Days})$$
     $$\text{Deduction Amount} = \text{Per Day Rate} \times \text{Deduction Units}$$
     $$\text{Net Salary} = \text{Monthly Salary} - \text{Deduction Amount}$$
   - Month picker for historical payroll calculation.
   - Print & "Save as PDF" button with custom `@media print` stylesheet.
7. **Smart HR Assistant (Floating Chatbot)**:
   - Natural keyword-matching assistant on both dashboards.
   - Interactive prompt suggestion chips and typing indicator animation.
   - **Strict Privacy Isolation**: Refuses employee inquiries regarding company-wide or peer metrics, restricting answers strictly to their personal records.
8. **Bonus Features**:
   - **Secure Camera & Geofence Check-in**: Live camera stream preview with facial reticle overlay and GPS perimeter distance verification against admin-configured coordinates.
   - **Dark Mode**: Persistent dark mode theme toggle stored in `localStorage`.
   - **CSV Export**: One-click download of company attendance records in CSV format.
   - **Live Ticking Clock**: Real-time 12-hour/24-hour synchronized clocks.

---

## 📂 Project Structure

```
Saylani/
├── index.html                 # Login page + First-run setup wizard
├── README.md                  # Complete architectural & operational guide
├── css/
│   └── style.css              # Modern design system (navy, teal, violet, amber) + print stylesheet
├── js/
│   ├── storage.js             # LocalStorage read/write engine & local timezone date utils
│   ├── auth.js                # Auth, session management, and role-based route guards
│   ├── ui.js                  # Modals, toasts, inline form errors, theme, and animations
│   ├── employees.js           # Employee directory controller & cascade delete
│   ├── attendance.js          # Shift check-in/out engine & policy computations
│   ├── leave.js               # Leave application validator & admin decision engine
│   ├── payslip.js             # Payroll calculation engine
│   └── helper.js              # Floating smart assistant widget
└── pages/
    ├── admin.html             # Executive dashboard with charts & stat cards
    ├── employees.html         # Employee directory CRUD & filters
    ├── leaves.html            # Leave request approval/rejection management
    ├── settings.html          # Company operational policies & geolocation settings
    ├── employee.html          # Employee portal & shift action console
    └── payslip.html           # Monthly payslip view & PDF export
```

---

## 🗄️ LocalStorage Data Model

The application isolates all data under the following structured keys:

| Key | Type | Description |
|---|---|---|
| `hcm_company` | `Object` | `{ name, startTime, graceMinutes, leaveQuota, workingDays, officeLat, officeLng, officeRadius, enableCameraCheckin }` |
| `hcm_users` | `Array<Object>` | `[{ id, name, email, password, role: "admin"|"employee", department, designation, salary, joinDate }]` |
| `hcm_attendance` | `Array<Object>` | `[{ id, employeeId, date: "YYYY-MM-DD", checkIn: "HH:MM", checkOut: "HH:MM"|null, status: "present"|"late"|"half_day", wasLate, workHours, verificationMethod }]` |
| `hcm_leaves` | `Array<Object>` | `[{ id, employeeId, type: "casual"|"sick"|"annual", from, to, days, reason, status: "pending"|"approved"|"rejected", comment, appliedOn, decidedOn }]` |
| `hcm_announcements` | `Array<Object>` | `[{ id, title, message, date, authorId }]` |
| `hcm_session` | `Object` | `{ userId, role, loggedInAt }` |
| `hcm_theme` | `String` | `'light'` or `'dark'` |

---

## 📋 Step-by-Step Demo Guide

### Step 1: Initial Setup Wizard
1. Launch `index.html`.
2. Notice the **Initial Setup Wizard** appears because `localStorage` is completely empty.
3. Enter your company details (e.g. `Acme Software Ltd.`), Super Admin Name (`Sarah Jenkins`), Email (`admin@acme.com`), and Password (`admin123`).
4. Click **Continue to Policies**. Review the start time (`09:00`), grace minutes (`15`), annual leave quota (`12`), and working days (Mon–Fri).
5. Click **Launch HCM Lite**. You are automatically authenticated and redirected to `pages/admin.html`.

### Step 2: Add Team Members
1. Go to **Employees** in the sidebar.
2. Click **+ Add Employee**.
3. Create 3 employees across 2 departments (e.g., Engineering and Design):
   - `alex@acme.com`, Password `pass123`, Engineering, Salary `$6,000`
   - `maya@acme.com`, Password `pass123`, Engineering, Salary `$5,500`
   - `liam@acme.com`, Password `pass123`, Design, Salary `$5,000`
4. Notice inline validation prevents duplicate emails, blank fields, and negative salaries.

### Step 3: Test Employee Portal & Attendance Rules
1. Log out from the Admin topbar.
2. Sign in as `alex@acme.com` (`pass123`).
3. You will land on `pages/employee.html`. Notice you cannot access `pages/admin.html`.
4. Click **Check In**. Depending on the current time and policy, your status is recorded as **Present** or **Late**.
5. Click **Check Out**. If under 5 hours, status is classified as **Half Day**.

### Step 4: Apply for Leave
1. In the employee portal, scroll to **Apply for Leave**.
2. Select upcoming dates (e.g., next Tuesday to Thursday = 3 working days).
3. Notice weekend dates are automatically excluded from the calculation.
4. Enter reason and click **Submit Leave Request**.
5. Notice your pending status appears in **My Leave Applications**.

### Step 5: Admin Approvals & Payslips
1. Log out and log back in as `admin@acme.com`.
2. Notice the stat card shows **1 Pending Leave**.
3. Go to **Leave Requests**, review Alex's request, and click **Approve** (or test **Reject** with a mandatory comment).
4. Go to **Payroll & Payslips** (`pages/payslip.html`).
5. Select Alex Morgan from the employee dropdown.
6. Verify the detailed breakdown: working days, present, late, half days, deductions, and net salary. Click **Print / Save as PDF**.

### Step 6: Test Smart Assistant
1. Open the floating assistant (bottom-right chat icon).
2. Ask: *"How many employees?"* &rarr; Responds with real departmental numbers.
3. Ask: *"Who is on leave today?"* &rarr; Lists employees currently on approved leave.
4. Log back in as an employee and ask *"Who was late today?"* &rarr; Notice it **strictly refuses** for security and privacy, prompting you to ask about your own data instead!

---

## 🛡️ Business Rules Implemented

- **Attendance**:
  - `late`: if check-in time > `startTime + graceMinutes`.
  - Exactly one check-in allowed per day; check-out only once after check-in.
  - Work hours = `checkOut - checkIn`. Under 5 hours = `half_day` (applied on check-out while preserving late status).
  - Check-in on non-working days blocked with friendly message.
  - Working days without attendance = `absent` (dynamically computed, never pre-stored). Days covered by approved leaves are marked `On leave` and not counted absent. Future days are never absent.
- **Leave**:
  - `From <= To`. Counts only working days (skips designated off-days). Must have $\ge 1$ working day.
  - Range cannot overlap any existing pending or approved leave of the same employee.
  - Cannot exceed annual quota (`quota - approved days - pending days`).
  - Balance updates only from approved leaves.
  - Admin approval/rejection with mandatory comment on reject.
- **Payroll**:
  - $\text{perDay} = \text{monthlySalary} / \text{workingDaysInMonth}$.
  - $\text{deduction} = \text{perDay} \times (\text{absentDays} + 0.5 \times \text{halfDays})$.
  - $\text{net} = \text{salary} - \text{deduction}$.

---

## 🎨 Design System & Palette

- **Default Appearance**: Fresh, Bright, Modern Light-Themed SaaS HR Platform
- **Base & Page Background**: Soft Blue-White (`#F7F9FC`)
- **Surfaces & Cards**: Pure Crisp White (`#FFFFFF`)
- **Secondary Surfaces & Table Headers**: Soft Slate Tint (`#F1F5F9`)
- **Borders**: Crisp Clean Border (`#E6EBF2`)
- **Primary Teal Accent**: `#0F9D8A` (Soft Tint: `#E3F6F3`, Dark: `#0B7F70`)
- **Secondary Violet Accent**: `#6D5EF5` (Soft Tint: `#EEEBFF`)
- **Warning / Late Amber**: `#F59E0B` (Soft Tint: `#FFF4DB`)
- **Danger / Absent Red**: `#EF4444` (Soft Tint: `#FDE8E8`)
- **Success / Present Green**: `#16A34A` (Soft Tint: `#E4F7EB`)
- **Info Blue Accent**: `#3B82F6` (Soft Tint: `#E5EFFF`)
- **Typography**: Plus Jakarta Sans (400, 500, 600, 700, 800) via Google Fonts
- **Theme Support**: Light Default everywhere with optional Dark Mode toggling (`[data-theme="dark"]`) persisted in `localStorage` (`"hcm_theme"`)

---

## 💡 Assumptions Made

1. **Local Timezone Consistency**: Date and time calculations intentionally utilize local browser time (e.g. `getFullYear()`, `getMonth()`, `getDate()`, `getHours()`) to avoid universal UTC date-shifting errors.
2. **First-Run State**: Because the hackathon requires the app to start completely empty, opening `index.html` on a fresh browser immediately presents the first-run configuration wizard. A "Reset HCM Lite Data" button is also provided in the Settings page and login footer for convenient testing.
3. **Calendar Working Days**: Standard default is Monday through Friday (JS days `1, 2, 3, 4, 5`), configurable by the Admin in Settings.

---

### Author
Built for the **Saylani Hackathon — Beginner Track**.
Pure HTML5, CSS3, and Modern Vanilla JavaScript.
