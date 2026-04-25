# Dev Care — API Endpoint Documentation

**Base URL:** `http://localhost:8000`
**Authentication:** JWT (via `Authorization: Bearer <access_token>`)

---

## 1. Authentication Endpoints (`/api/auth/`)

### 1.1 Register

| | |
|---|---|
| **URL** | `POST /api/auth/register/` |
| **Auth** | ❌ None |
| **Description** | Register a new user (patient or doctor). Returns JWT tokens immediately so the user is logged in on registration. |

**Request Body:**
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "securePass123",
  "password_confirm": "securePass123",
  "first_name": "John",
  "last_name": "Doe",
  "role": "patient"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `username` | string | ✅ | Unique username |
| `email` | string | ✅ | Unique email |
| `password` | string | ✅ | Min 8 characters |
| `password_confirm` | string | ✅ | Must match `password` |
| `first_name` | string | ❌ | |
| `last_name` | string | ❌ | |
| `role` | string | ❌ | `"patient"` (default) or `"doctor"` |

**Success Response (`201 Created`):**
```json
{
  "message": "Registration successful.",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com",
    "role": "patient"
  },
  "refresh": "eyJhbGciOiJIUzI1NiIs...",
  "access": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Response (`400 Bad Request`):**
```json
{
  "password_confirm": ["Passwords do not match."],
  "email": ["Email is already in use."]
}
```

---

### 1.2 Login

| | |
|---|---|
| **URL** | `POST /api/auth/login/` |
| **Auth** | ❌ None |
| **Description** | Authenticate a user and return JWT tokens along with user info including role. |

**Request Body:**
```json
{
  "username": "john_doe",
  "password": "securePass123"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `username` | string | ✅ | |
| `password` | string | ✅ | |

**Success Response (`200 OK`):**
```json
{
  "refresh": "eyJhbGciOiJIUzI1NiIs...",
  "access": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com",
    "role": "patient"
  }
}
```

**Error Response (`401 Unauthorized`):**
```json
{
  "detail": "No active account found with the given credentials"
}
```

---

### 1.3 Refresh Token

| | |
|---|---|
| **URL** | `POST /api/auth/refresh/` |
| **Auth** | ❌ None |
| **Description** | Get a new access token using a valid refresh token. |

**Request Body:**
```json
{
  "refresh": "eyJhbGciOiJIUzI1NiIs..."
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `refresh` | string | ✅ | A valid refresh token |

**Success Response (`200 OK`):**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Response (`401 Unauthorized`):**
```json
{
  "detail": "Token is invalid or expired",
  "code": "token_not_valid"
}
```

---

## 2. Rehab Endpoints (`/api/rehab/`)

> [!NOTE]
> All rehab endpoints require a valid JWT access token in the `Authorization` header.

### 2.1 List Exercise Templates

| | |
|---|---|
| **URL** | `GET /api/rehab/exercises/` |
| **Auth** | ✅ Any authenticated user |
| **Description** | Returns all available exercise templates ordered by name. Used by doctors when building rehab plans and by patients to understand exercise parameters. |

**Request Body:** _None_

**Success Response (`200 OK`):**
```json
[
  {
    "id": 1,
    "name": "bicep curl",
    "description": "Curl the forearm toward the shoulder.",
    "target_joint": "elbow",
    "instructions": "Stand with arms at your sides, curl forearm up slowly.",
    "min_angle": 30.0,
    "max_angle": 160.0
  },
  {
    "id": 2,
    "name": "shoulder abduction",
    "description": "Raise arm sideways away from the body.",
    "target_joint": "shoulder",
    "instructions": "Stand straight, raise arm to shoulder height, then lower.",
    "min_angle": 0.0,
    "max_angle": 90.0
  }
]
```

---

### 2.2 Create Rehab Plan

| | |
|---|---|
| **URL** | `POST /api/rehab/plans/` |
| **Auth** | ✅ Doctor only |
| **Description** | Doctor creates a new rehab plan and assigns it to a patient. Includes an ordered list of exercises with target reps. |

**Request Body:**
```json
{
  "patient_id": 3,
  "name": "Post-Surgery Elbow Recovery - Week 1",
  "exercises": [
    {
      "exercise_id": 1,
      "order": 1,
      "target_reps": 10
    },
    {
      "exercise_id": 2,
      "order": 2,
      "target_reps": 15
    }
  ]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `patient_id` | integer | ✅ | ID of a user with `role=patient` |
| `name` | string | ✅ | Max 120 characters |
| `exercises` | array | ✅ | At least one exercise required |
| `exercises[].exercise_id` | integer | ✅ | Must reference an existing `ExerciseTemplate` |
| `exercises[].order` | integer | ✅ | Must be unique within the plan (≥ 1) |
| `exercises[].target_reps` | integer | ✅ | Target number of reps (≥ 0) |

**Success Response (`201 Created`):**
```json
{
  "id": 1,
  "doctor_id": 2,
  "patient_id": 3,
  "name": "Post-Surgery Elbow Recovery - Week 1",
  "created_at": "2026-04-25T14:00:00Z",
  "exercises": [
    {
      "order": 1,
      "target_reps": 10,
      "exercise": {
        "id": 1,
        "name": "bicep curl",
        "description": "Curl the forearm toward the shoulder.",
        "target_joint": "elbow",
        "instructions": "Stand with arms at your sides, curl forearm up slowly.",
        "min_angle": 30.0,
        "max_angle": 160.0
      }
    },
    {
      "order": 2,
      "target_reps": 15,
      "exercise": {
        "id": 2,
        "name": "shoulder abduction",
        "description": "Raise arm sideways away from the body.",
        "target_joint": "shoulder",
        "instructions": "Stand straight, raise arm to shoulder height, then lower.",
        "min_angle": 0.0,
        "max_angle": 90.0
      }
    }
  ]
}
```

**Error Responses:**

`403 Forbidden` — Non-doctor user:
```json
{ "detail": "Only doctor users can create rehab plans." }
```

`400 Bad Request` — Invalid patient or exercises:
```json
{ "patient_id": ["Patient does not exist."] }
```
```json
{ "exercises": ["Exercise order values must be unique."] }
```

---

### 2.3 Get Rehab Plan Detail

| | |
|---|---|
| **URL** | `GET /api/rehab/plans/<plan_id>/` |
| **Auth** | ✅ Assigned doctor or patient only |
| **Description** | Retrieve a specific rehab plan with its ordered exercises. Doctors can only view plans they created; patients can only view plans assigned to them. |

**Request Body:** _None_

**URL Parameters:**

| Param | Type | Notes |
|---|---|---|
| `plan_id` | integer | ID of the rehab plan |

**Success Response (`200 OK`):**
```json
{
  "id": 1,
  "doctor_id": 2,
  "patient_id": 3,
  "name": "Post-Surgery Elbow Recovery - Week 1",
  "created_at": "2026-04-25T14:00:00Z",
  "exercises": [
    {
      "order": 1,
      "target_reps": 10,
      "exercise": {
        "id": 1,
        "name": "bicep curl",
        "description": "Curl the forearm toward the shoulder.",
        "target_joint": "elbow",
        "instructions": "Stand with arms at your sides, curl forearm up slowly.",
        "min_angle": 30.0,
        "max_angle": 160.0
      }
    },
    {
      "order": 2,
      "target_reps": 15,
      "exercise": {
        "id": 2,
        "name": "shoulder abduction",
        "description": "Raise arm sideways away from the body.",
        "target_joint": "shoulder",
        "instructions": "Stand straight, raise arm to shoulder height, then lower.",
        "min_angle": 0.0,
        "max_angle": 90.0
      }
    }
  ]
}
```

**Error Responses:**

`403 Forbidden`:
```json
{ "detail": "You can only view plans you created." }
```
```json
{ "detail": "You can only view plans assigned to you." }
```

`404 Not Found`:
```json
{ "detail": "No RehabPlan matches the given query." }
```

---

### 2.4 Start Exercise Session

| | |
|---|---|
| **URL** | `POST /api/rehab/sessions/start/` |
| **Auth** | ✅ Patient only |
| **Description** | Patient starts a new exercise session for one of their assigned rehab plans. Creates an open session with `started_at` timestamp and `completed_at = null`. |

**Request Body:**
```json
{
  "plan_id": 1
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `plan_id` | integer | ✅ | ID of a `RehabPlan` assigned to this patient |

**Success Response (`201 Created`):**
```json
{
  "id": 1,
  "patient_id": 3,
  "plan_id": 1,
  "started_at": "2026-04-25T14:30:00Z",
  "completed_at": null,
  "results": []
}
```

**Error Responses:**

`403 Forbidden`:
```json
{ "detail": "Only patient users can start rehab sessions." }
```
```json
{ "detail": "You can only start sessions for plans assigned to you." }
```

`404 Not Found`:
```json
{ "detail": "No RehabPlan matches the given query." }
```

---

### 2.5 Complete Exercise Session

| | |
|---|---|
| **URL** | `POST /api/rehab/sessions/<session_id>/complete/` |
| **Auth** | ✅ Patient only (session owner) |
| **Description** | Patient completes a session by submitting per-exercise performance results evaluated by the frontend MediaPipe module. Replaces any existing results for this session. Sets the `completed_at` timestamp. |

**URL Parameters:**

| Param | Type | Notes |
|---|---|---|
| `session_id` | integer | ID of the exercise session |

**Request Body:**
```json
{
  "completed_at": "2026-04-25T14:45:00Z",
  "results": [
    {
      "exercise_id": 1,
      "order": 1,
      "reps": 8,
      "accuracy": 85.5,
      "duration": 120.0
    },
    {
      "exercise_id": 2,
      "order": 2,
      "reps": 12,
      "accuracy": 92.3,
      "duration": 90.0
    }
  ]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `completed_at` | datetime (ISO 8601) | ❌ | Defaults to server time if omitted |
| `results` | array | ✅ | At least one result required |
| `results[].exercise_id` | integer | ✅ | Must be an exercise assigned in the plan |
| `results[].order` | integer | ✅ | Unique within results (≥ 1) |
| `results[].reps` | integer | ✅ | Number of reps completed (≥ 0) |
| `results[].accuracy` | float | ✅ | Accuracy percentage from MediaPipe (0.0–100.0) |
| `results[].duration` | float | ✅ | Duration in seconds (≥ 0.0) |

**Success Response (`200 OK`):**
```json
{
  "id": 1,
  "patient_id": 3,
  "plan_id": 1,
  "started_at": "2026-04-25T14:30:00Z",
  "completed_at": "2026-04-25T14:45:00Z",
  "results": [
    {
      "order": 1,
      "exercise_id": 1,
      "exercise_name": "bicep curl",
      "reps": 8,
      "accuracy": 85.5,
      "duration": 120.0
    },
    {
      "order": 2,
      "exercise_id": 2,
      "exercise_name": "shoulder abduction",
      "reps": 12,
      "accuracy": 92.3,
      "duration": 90.0
    }
  ]
}
```

**Error Responses:**

`403 Forbidden`:
```json
{ "detail": "Only patient users can complete rehab sessions." }
```
```json
{ "detail": "You can only complete your own sessions." }
```

`400 Bad Request` — Exercise not in plan:
```json
{
  "detail": "Submitted results include exercises not assigned in this plan.",
  "exercise_ids": [99]
}
```

`400 Bad Request` — Duplicate order:
```json
{ "results": ["Result order values must be unique."] }
```

---

## 3. AI Module Endpoints (`/api/ai/`)

> [!IMPORTANT]
> This is a legacy/utility endpoint. The primary rehab flow uses the `/api/rehab/` endpoints above.

### 3.1 Upload Session (Legacy)

| | |
|---|---|
| **URL** | `POST /api/ai/upload-session/` |
| **Auth** | ✅ Doctor or Patient |
| **Description** | Upload a standalone exercise session result. If called by a doctor, `patient_id` is required. If called by a patient, the session is automatically assigned to them. |

**Request Body (as Patient):**
```json
{
  "exercise": "bicep curl",
  "reps": 10,
  "avg_range": 130.5,
  "form_accuracy": 88.0,
  "duration": 45.0
}
```

**Request Body (as Doctor):**
```json
{
  "patient_id": 3,
  "exercise": "bicep curl",
  "reps": 10,
  "avg_range": 130.5,
  "form_accuracy": 88.0,
  "duration": 45.0
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `patient_id` | integer | Only for doctors | Must reference a patient user |
| `exercise` | string | ✅ | Exercise name (lowercased) |
| `reps` | integer | ✅ | ≥ 0 |
| `avg_range` | float | ✅ | Average range of motion (≥ 0) |
| `form_accuracy` | float | ✅ | 0.0–100.0 |
| `duration` | float | ✅ | Duration in seconds (≥ 0) |

**Success Response (`201 Created`):**
```json
{ "status": "saved" }
```

**Error Response (`400 Bad Request`):**
```json
{ "detail": "exercise is required." }
```

---

## Quick Reference

| Method | Endpoint | Auth | Role | Description |
|---|---|---|---|---|
| `POST` | `/api/auth/register/` | ❌ | Any | Register a new user |
| `POST` | `/api/auth/login/` | ❌ | Any | Login and get JWT tokens |
| `POST` | `/api/auth/refresh/` | ❌ | Any | Refresh an access token |
| `GET` | `/api/rehab/exercises/` | ✅ | Any | List all exercise templates |
| `POST` | `/api/rehab/plans/` | ✅ | Doctor | Create a rehab plan |
| `GET` | `/api/rehab/plans/<id>/` | ✅ | Doctor/Patient | Get plan with exercises |
| `POST` | `/api/rehab/sessions/start/` | ✅ | Patient | Start an exercise session |
| `POST` | `/api/rehab/sessions/<id>/complete/` | ✅ | Patient | Complete session with results |
| `POST` | `/api/ai/upload-session/` | ✅ | Doctor/Patient | Upload standalone session (legacy) |
