I've updated the document by expanding **Step 2** with guidance on how the frontend/mobile should render the Google button.

# Google Registration Flow (Frontend / Mobile)

## Overview

Google registration is a two-step onboarding process. The frontend/mobile application is responsible for authenticating the user with Google, while the backend validates the Google identity and completes the registration.

```
App
 │
 │ GET /api/v1/sports360/auth/init
 ▼
Backend
 │
 │ Returns application configuration
 │ (including googleClientId)
 ▼
App
 │
 │ Render "Continue with Google"
 │ using googleClientId
 ▼
Google
 │
 │ User authenticates
 │
 │ Returns idToken
 ▼
App
 │
 │ POST /api/v1/sports360/public/users/google-verify
 │ { idToken }
 ▼
Backend
 │
 │ Validates Google token
 │
 │ Returns:
 │ • email
 │ • firstName
 │ • lastName
 │ • onboardingToken
 ▼
App
 │
 │ Show registration popup
 │
 │ Prefill firstName & lastName
 │
 │ User may edit them
 ▼
App
 │
 │ POST /api/v1/sports360/public/users/google-register
 │ Authorization: Bearer {onboardingToken}
 │
 │ {
 │   "firstName": "...",
 │   "lastName": "..."
 │ }
 ▼
Backend
 │
 │ Complete user registration
 ▼
Registration Complete
```

## Step 1 – Load Authentication Configuration

Call:

```
GET /api/v1/sports360/auth/init
```

The response contains all authentication configuration, including the Google Client ID.

Example:

```json
{
  "code": 200,
  "message": "Operation successful.",
  "data": {
    "...": "...",
    "googleClientId": "9718......s25bclpsgf.apps.googleusercontent.com"
  }
}
```

Use `googleClientId` to initialize the Google Sign-In SDK.

---

## Step 2 – Render the Google Sign-In Button

Use the `googleClientId` returned from the previous step to initialize the official Google Sign-In SDK for your platform.

* **Web:** Initialize Google Identity Services (GIS) using the `googleClientId`, then render a **Continue with Google** button on the registration page.
* **Android:** Configure the Google Sign-In SDK with the `googleClientId` (Web Client ID) and launch the Google Sign-In flow when the user taps the button.
* **iOS:** Configure the Google Sign-In SDK with the same `googleClientId` (Web Client ID) and launch the Google Sign-In flow.

After the user successfully authenticates with Google, the SDK returns an **ID Token (`idToken`)**. This token must be sent to the backend for verification.

> **Note:** The frontend/mobile application does **not** validate the Google token locally. It simply obtains the `idToken` from Google and forwards it to the backend.

---

## Step 3 – Verify Google Identity

Call:

```
POST /api/v1/sports360/public/users/google-verify
```

Request:

```json
{
  "idToken": "eyJhbGciOi..."
}
```

Example response:

```json
{
  "code": 200,
  "message": "Operation successful.",
  "data": {
    "email": "shamimahmed3082@gmail.com",
    "firstName": "Ahmed",
    "lastName": "Shamim",
    "onboardingToken": "eyJhbGciOi..."
  }
}
```

The backend validates the Google ID Token and returns the user's profile information together with an `onboardingToken`.

---

## Step 4 – Complete Registration

Display a registration popup using the returned `firstName` and `lastName`.

* Prefill both fields.
* Allow the user to edit them before submission.

Call:

```
POST /api/v1/sports360/public/users/google-register
```

Authorization:

```
Bearer {onboardingToken}
```

Request:

```json
{
  "firstName": "Shamim",
  "lastName": "Ahmed"
}
```

If the request succeeds, the user's registration is complete.
