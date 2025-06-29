# 🔐 Authentication API

A secure and extensible authentication system supporting email-password, Google, and GitHub sign-ins with token-based session management.

---

## 📦 Features

- Email/Password Registration & Login
- OTP-based Email Verification & Password Reset
- Secure Refresh Token via HTTP-only cookies
- Google & GitHub OAuth2 Authentication
- Two-Factor Authentication (2FA)
- Admin Panel for User Management
- User Profile Info (`/auth/me`)

---

## 🧪 API Endpoints

### 🔐 Email and Password Auth

#### ✅ Register

**POST** `/auth/register`

```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:**

```json
{
  "message": "User registered successfully. OTP sent to email."
}
```

---

#### ✉️ Send OTP

**POST** `/auth/send-otp`

```json
{
  "email": "user@example.com"
}
```

**Response:**

```json
{
  "message": "OTP sent to email"
}
```

---

#### 🔑 Verify OTP

**POST** `/auth/verify-otp`

```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response:**

Success:

```json
{
  "message": "OTP verified successfully"
}
```

Failure:

```json
{
  "message": "OTP not verified"
}
```

---

#### 🔓 Login

**POST** `/auth/login`

```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:**

```json
{
  "access_token": "jwt_token_here",
  "token_type": "bearer"
}
```

> ℹ️ Sets a secure, HTTP-only cookie: `refresh_token`

---

#### 🔁 Refresh Token

**POST** `/auth/refresh-token`

> No request body required. Uses the `refresh_token` cookie.

**Response:**

```json
{
  "access_token": "new_access_token",
  "token_type": "bearer"
}
```

---

#### 🚪 Logout

**POST** `/auth/logout`

> Clears `refresh_token` cookie.

**Response:**

```json
{
  "message": "Logged out successfully"
}
```

---

### 🔁 Password Reset Flow

#### 📤 Forgot Password

**POST** `/auth/forgot-password`

```json
{
  "email": "user@example.com"
}
```

**Response:**

Success:

```json
{
  "message": "Password reset OTP sent to email."
}
```

Failure:

```json
{
  "detail": "User with this email does not exist."
}
```

---

#### 🔐 Verify Reset OTP

**POST** `/auth/verify-reset-otp`

```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response:**

Success:

```json
{
  "message": "OTP verified successfully."
}
```

Failure:

```json
{
  "detail": "Invalid or expired OTP."
}
```

---

#### 🔄 Reset Password

**POST** `/auth/reset-password`

```json
{
  "email": "user@example.com",
  "new_password": "newSecurePassword123"
}
```

**Response:**

Success:

```json
{
  "message": "Password updated successfully."
}
```

Failure:

```json
{
  "detail": "OTP not verified."
}
```

---

## 🔐 Google OAuth2

#### 1️⃣ Login Redirect

**GET** `/auth/google-login`

**Response:**

```json
{
  "redirect_url": "https://accounts.google.com/o/oauth2/auth?client_id=..."
}
```

---

#### 2️⃣ Google Callback

**GET** `/auth/google-callback?code=xyz&state=abc`

**Response:**

```json
{
  "access_token": "your.jwt.token",
  "refresh_token": "optional.refresh.token",
  "user": {
    "email": "user@gmail.com",
    "name": "User Name"
  }
}
```

---

## 🐙 GitHub OAuth

#### 1️⃣ Login Redirect

**GET** `/auth/github-login`

**Response:**

```json
{
  "redirect_url": "https://github.com/login/oauth/authorize?client_id=..."
}
```

---

#### 2️⃣ GitHub Callback

**GET** `/auth/github-callback?code=xyz`

**Response:**

```json
{
  "access_token": "jwt_token",
  "user": {
    "email": "user@example.com",
    "provider": "github",
    "name": "GitHub User"
  }
}
```

---

## 👤 User Profile

**GET** `/auth/me`

**Response:**

```json
{
  "email": "user@gmail.com",
  "provider": "google",
  "name": "User Name"
}
```

---

## 🔐 Two-Factor Authentication (2FA)

#### 1️⃣ Setup 2FA

**POST** `/auth/2fa/setup`

**Response:**

```json
{
  "secret": "BASE32ENCODEDSECRET",
  "qr_code_url": "otpauth://totp/YourApp:user@example.com?secret=...&issuer=YourApp"
}
```

---

#### 2️⃣ Verify 2FA

**POST** `/auth/2fa/verify`

```json
{
  "code": "123456"
}
```

**Response:**

```json
{
  "message": "2FA setup complete."
}
```

---

#### 3️⃣ Disable 2FA

**POST** `/auth/2fa/disable`

```json
{
  "password": "securePassword123"
}
```

**Response:**

```json
{
  "message": "2FA disabled."
}
```

---

#### 4️⃣ Login with 2FA

**POST** `/auth/2fa/login`

```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "code": "123456"
}
```

**Response:**

```json
{
  "access_token": "jwt_token",
  "token_type": "bearer"
}
```

---

## ⏱️ Rate Limiting

> Middleware enforced (no endpoint). Example error response:

```json
{
  "detail": "Rate limit exceeded. Try again later."
}
```

Applied on:
- `/auth/login`
- `/auth/register`
- OTP endpoints

---

## 🗂️ Admin Panel (Restricted)

#### 🔍 Get All Users

**GET** `/admin/users`

**Response:**

```json
[
  {
    "id": "uuid",
    "email": "user@example.com",
    "provider": "google",
    "is_verified": true,
    "created_at": "2025-06-29T15:00:00Z"
  }
]
```

---

#### 🚫 Block a User

**POST** `/admin/users/{user_id}/block`

```json
{
  "reason": "Suspicious activity"
}
```

**Response:**

```json
{
  "message": "User access revoked"
}
```

---

#### 📝 Update User Info

**PUT** `/admin/users/{user_id}`

```json
{
  "is_verified": false,
  "name": "Updated Name"
}
```

**Response:**

```json
{
  "message": "User info updated"
}
```

---

## 🧱 Database Schema

### 📄 `users`

| Column         | Type     | Description                                    |
| -------------- | -------- | ---------------------------------------------- |
| id             | UUID     | Primary key                                    |
| email          | String   | Unique, shared across all login types          |
| password_hash  | String   | Null for OAuth users                           |
| name           | String   | Display name                                   |
| provider       | Enum     | `email/password`, `google`, `github`           |
| google_sub     | String   | Google user ID (nullable)                      |
| github_id      | String   | GitHub user ID (nullable)                      |
| is_verified    | Boolean  | Whether email/OTP is verified                  |
| created_at     | DateTime | Account creation time                          |

---

### 🧾 `otps`

| Column      | Type     | Description          |
| ----------- | -------- | -------------------- |
| id          | UUID     | Primary key          |
| email       | String   | Associated user      |
| otp         | String   | 6-digit OTP code     |
| expires_at  | DateTime | Expiration timestamp |
| is_used     | Boolean  | OTP usage flag       |

---

## 🛡️ Security Notes

- Passwords are stored as bcrypt hashes.
- Refresh tokens are stored in secure, HTTP-only cookies.
- OTPs expire and are single-use.
- Email and password flows are fully protected via token-based auth.
