# Hyscaller Task: Employee Management & Leave Portal

## 🚀 Project Overview
This project is a comprehensive **Employee Management and Leave Portal** developed as part of a hiring process for **BA Organization**. It features a robust FastAPI backend combined with a premium, strictly black-and-white React frontend built with `shadcn/ui`.

The application differentiates between two primary roles:
- **Managers**: Can manage the team, review leave applications, and track overall employee status.
- **Employees**: Can apply for leaves, track their application status, and view their remaining leave balances.

---

## 🏗️ Architecture & Flow

### 📋 Backend Flow (FastAPI)
1. **Request Reception**: handles requests via REST endpoints in `main.py`.
2. **Authentication**: JWT-based security. Tokens include user identity and expiration.
3. **Database**: MongoDB Atlas stores the state of the organization.
4. **Logic**: `main.py` coordinates between database operations and external services (SMTP).

### 💻 Frontend Flow (React)
1. **Entry**: `main.jsx` initializes the app and `App.jsx` handles routing.
2. **Security**: Private routes redirect unauthenticated users to `/login`.
3. **Dashboards**: Components fetch data from the backend via the `/api` service on mount.
4. **Interactions**: User actions trigger API calls, followed by local state updates to reflect changes instantly.

---

## 📂 Project Structure

```text
├── backend
│   ├── .env                # Secure secrets (DB, SMTP, Tokens)
│   ├── main.py             # CORE logic, route handlers & SMTP calls
│   ├── requirements.txt    # Project dependencies
│   └── models
│       └── schemas.py      # Pydantic data validation models
├── frontend
│   ├── .env                # Frontend environment configuration (VITE_API_URL)
│   ├── public/             # Static assets like vite.svg
│   ├── src
│   │   ├── api
│   │   │   └── index.js    # Centralized Axios logic & API functions
│   │   ├── components/ui   # shadcn/ui components (Atomic units)
│   │   ├── contexts
│   │   │   └── AuthContext.jsx # Authentication state provider
│   │   ├── lib/utils.js    # Tailwind merging utility
│   │   ├── pages/          # Feature-specific page components
│   │   ├── App.jsx         # Routing configuration
│   │   └── main.jsx        # React root initialization
│   ├── package.json        # Frontend configuration
│   └── vite.config.js      # Vite build settings
```

---

## 💻 Frontend Pages & Implementation

| File | Purpose | Key Features |
| :--- | :--- | :--- |
| `Landing/index.jsx` | Public Entry Point | Premium visual welcome, navigation to Login. |
| `Login/index.jsx` | Auth Portal | Password visibility toggle, role-based redirection. |
| `EmployeeDashboard/index.jsx` | Worker View | Leave balance cards, self-application list, "Apply" portal. |
| `ManagerDashboard/index.jsx` | Admin View | Team directory, global leave feed, Review/Approve dialogs. |
| `Profile/index.jsx` | User Settings | Personal details view, leave usage history, account security. |

---

## 📡 API Detailed Documentation

### 1. User Authentication
*   **Login** (`POST /login`)
    *   **Payload**: `{"email": "user@example.com", "password": "..."}`
    *   **Response**: `{"access_token": "token_string", "user": {"id": "...", "role": "..."}}`

### 2. Employee Management (Manager Role)
*   **Create Employee** (`POST /employees`)
    *   **Payload**: `{"name": "...", "email": "...", "password": "..."}`
    *   **Logic**: Creates user + initializes balance + sends "Welcome Email".
*   **List Team** (`GET /employees`)
    *   **Response**: `[{"id": "...", "name": "...", "email": "...", "role": "employee"}]`

### 3. Leave Operations
*   **Apply for Leave** (`POST /leaves`)
    *   **Payload**: `{"leave_type": "Vacation Leave", "start_date": "2024-03-20", "end_date": "...", "reason": "..."}`
    *   **Logic**: Auto-calculates days (excluding Sundays).
*   **Update Leave** (`PUT /leaves/{id}`)
    *   **Payload**: `{"status": "approved", "manager_comment": "Enjoy!"}`
    *   **Logic**: Deducts from balance + sends "Decision Email".
*   **Delete/Cancel** (`DELETE /leaves/{id}`)
    *   **Logic**: 
        - **Managers**: Performs a **Soft-Archive**. Marks the record as hidden from the global feed but preserves it for employee history and balance accuracy.
        - **Employees**: (Planned) Allows cancelling pending requests.

### 4. Financials & Balances
*   **My Balance** (`GET /my-balance`)
    *   **Response**: `{"vacation_remaining": 15, "sick_remaining": 2, "extra_leave": 0}`

---

## 🎨 Technology Breakdown

### shadcn/ui (The UI Engine)
We used **shadcn/ui** to ensure the hiring organization sees a state-of-the-art interface.
- **Why?**: It offers raw code access. We modified the default "Slate" themes to **Pure Black & White** to create a high-contrast, premium "Dark Mode" feel where requested.
- **Customizations**: Dialogs use black headers (`bg-black`), Stat cards use vibrant border accents for status readability.

### Email System (SMTP)
- **Functions**: `send_email` in `main.py`.
- **Flow**: Triggered during Registration and Request Decision stages. Uses secure TLS connection (Port 587).

---

## 📖 Setup & Deployment Guide

1.  **Environment Setup**: 
    - Copy backend secrets into `backend/.env`.
    - Create `frontend/.env` and set `VITE_API_URL=http://localhost:8000`.
2.  **Dependencies**:
    - Backend: `pip install -r backend/requirements.txt`
    - Frontend: `npm install --prefix frontend`
3.  **Run Pipeline**:
    - Start Backend: `uvicorn backend.main:app --host 0.0.0.0 --port 8000`
    - Start Frontend: `npm run dev --prefix frontend`
4.  **Swagger Access**: Open `http://localhost:8000/docs` to test all endpoints manually.

---

## 🛡️ Error Handling & Reliability

### Backend Health Checks
The system includes a **Health Monitoring** system.
- **Endpoint**: `GET /health`
- **Logic**: During startup, the backend pings MongoDB. If the connection fails, it sets a global `DB_CONNECTED` flag to `False`. 
- **Response**: Returns `503 Service Unavailable` if the database is disconnected, alerting the frontend immediately.

### Frontend Global Error Page
We implemented a premium, high-contrast **ErrorPage** component.
- **Automatic Detection**: Axios interceptors monitor all responses. If a `503` is detected (DB Offline), the user is automatically redirected to `/db-error`.
- **404 Handling**: A custom "Page Not Found" experience is provided for invalid routes, maintaining the consistent design aesthetic.
- **Recovery**: Both error states provide "Try Again" (Refresh) and "Back to Home" actions for a seamless user recovery path.

---

## ⚡ Performance Optimization

To ensure a professional-grade user experience, the application implements several front-end optimization patterns:

### 1. Context Memoization
The `AuthContext` uses `useMemo` to wrap its provider value. This ensures that child components (dashboards, navbars) do not trigger unnecessary re-renders unless the actual user session state changes.

### 2. API Fetch Locking (Singleton Pattern)
We use the `useRef` hook in all major dashboard pages to implement a "Fetch Lock". This prevents:
- **Double-calling APIs** during React's initial mount (or StrictMode).
- **Redundant fetches** when incidental component state updates trigger a re-render.

### 3. Environment Awareness
The frontend is fully decoupled from the backend URL via `VITE_API_URL`, allowing for seamless transition between local development and production environments without modifying source code.

---

## 🧭 Application Walkthrough (User Flows)

This guide outlines the complete operational cycle for both Managers and Employees within the portal.

### 1. Manager Authentication (Initial Step)
The system initializes with a default administrative account for primary setup.
- **Login Credentials**:
    - **Email**: `biswapvt506@gmail.com`
    - **Password**: `Manager1@123`
- **Actions**: Log in as a Manager. You will be redirected to the **Manager Dashboard**, where you can see the Team Directory and the global Leave Feed.

### 2. Team Expansion (Manager Flow)
If the directory is empty, the Manager must register team members.
- **Onboard Employee**: Click the **"+ Add Employee"** button.
- **Credential Storage**: Enter the employee's name, email, and a temporary password.
- **Automated Notification**: Upon submission, the system creates the profile in the database and **automatically sends an email** to the employee with their credentials.

### 3. Employee Portal & Leave Request (Employee Flow)
- **Portal Access**: The employee logs in using the credentials received via email.
- **Dashboard Insights**: Employees see their **Vacation** and **Sick Leave** balances updated in real-time.
- **Apply for Leave**: Click **"+ Apply for Leave"**. Fill in the dates and reason. 
- **Calendar Visualization (Pending)**: On the Profile page, the requested dates will appear in **Yellow (Amber)** on the calendar, indicating they are awaiting review.
- **Manager Review**: The Manager receives the request in their "Leave Feed". They can either **Approve** or **Reject** with an internal comment.

### 4. Decision & Status Feedback (Integrated Flow)
- **Approved State**: Once approved, the employee's calendar dates turn **Red**.
- **Out of Office Indicator**: If today's date falls within an approved leave range:
    - The employee's avatar turns **Red**.
    - A pulse badge appears: **"On Leave Today"**.
    - The Manager Dashboard will display **"On Leave"** status next to the employee's name in the directory.
- **Rejected State**: The request remains in the history with the Manager's remarks but is removed from the calendar highlights.

---

## 🔗 Project Links

- **GitHub Repository**: [https://github.com/Biswa-source45/Leave-Management-System--Task.git](https://github.com/Biswa-source45/Leave-Management-System--Task.git)
- **Live Application**: [https://leave-mng-task.vercel.app/](https://leave-mng-task.vercel.app/)
- **Swagger API Docs**: [https://task-backend-4gwa.onrender.com/docs](https://task-backend-4gwa.onrender.com/docs)

> [!IMPORTANT]
> **Performance Note**: The backend is hosted on a free-tier service. Please wait **30 seconds** after the first interaction (or login attempt) for the server to spin up and process your credentials.
