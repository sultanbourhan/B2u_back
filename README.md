# B2U Backend API

A RESTful backend API for the **B2U (Best To You)** platform — a business/company directory that allows users to browse companies, view advertisements, leave reviews and comments, and manage company profiles with subscription-based plans.

## Frontend

The frontend for this project is available at:
**[B2U Frontend Repository](https://github.com/sultanbourhan/B2U_front)**

## Tech Stack

- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** MongoDB (via Mongoose)
- **Authentication:** JSON Web Tokens (JWT)
- **Image Processing:** Multer + Sharp
- **Email:** Nodemailer (Gmail SMTP)
- **Validation:** express-validator
- **Security:** cors, hpp, xss-clean, express-mongo-sanitize, express-rate-limit

## Project Structure

```
├── server.js                  # Entry point — Express app setup, middleware, DB connection
├── config.env                 # Environment variables (PORT, DB_URL, JWT secret, email creds)
├── package.json               # Dependencies and scripts
├── ApiError.js                # Custom error class with HTTP status codes
│
├── models/
│   ├── userModels.js          # User schema (name, email, password, role, points, etc.)
│   ├── companyModels.js       # Company schema (info, ratings, reviews, comments, subscription)
│   ├── Company_requestsModels.js  # Pending company registration requests
│   ├── CategoreyModels.js     # Category schema (name, description, image)
│   └── advertisementsModels.js    # Advertisement schema (images, title, likes)
│
├── routes/
│   ├── authRoutes.js          # Auth endpoints (signup, login, password reset, profile)
│   ├── userRoutes.js          # Admin user management endpoints
│   └── companyRoutes.js       # Company, ads, categories, reviews, comments, requests
│
├── services/
│   ├── authServicrs.js        # Auth logic (JWT, email verification, password reset)
│   ├── userServicrs.js        # User CRUD logic + image upload
│   └── companyServicrs.js     # Company CRUD, ads, categories, reviews, comments, requests
│
├── validationResulterror/
│   ├── validationResulte.js   # Validation middleware (returns 400 on errors)
│   ├── v_auth.js              # Auth input validation rules
│   ├── v_user.js              # User input validation rules
│   └── v_company.js           # Company/ads/category input validation rules
│
├── resetEmail.js              # Nodemailer transporter for sending verification emails
├── sindEmailMe.js             # Nodemailer transporter for contact form (sends to admin)
├── subscriptionReminder.js    # Cron job — sends subscription expiry reminders (5 days before)
│
└── image/                     # Uploaded images storage
    ├── user/                  # User profile images
    └── company/               # Company images, logos, ads, category images
```

## Getting Started

### Prerequisites

- Node.js (v16 or higher)
- MongoDB Atlas account (or local MongoDB instance)
- Gmail account with App Password for email sending

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd B2u_back-master
```

2. Install dependencies:
```bash
npm install
```

3. Configure environment variables in `config.env`:
```env
PORT=8000
NODE_ENV=development
DB_URL=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/company?retryWrites=true&w=majority
WJT_SECRET=your-jwt-secret-key
WJT_EXPIRESIN=90d
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password
RECEIVER_EMAIL=admin-email@gmail.com
BASE_URL=localhost:8000
```

4. Start the development server:
```bash
npm run dev
```

The server runs on `http://localhost:8000`.

## API Endpoints

### Authentication — `/api/v2/auth`

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/sign_up` | Register (sends verification code via email) | No |
| POST | `/verify` | Verify email with code and complete registration | No |
| POST | `/login` | Login and receive JWT token | No |
| PUT | `/logout` | Logout (sets user inactive) | Yes |
| GET | `/get_date_my` | Get current user profile | Yes |
| PUT | `/update_date_user_my` | Update current user profile | Yes |
| PUT | `/update_user_password_my` | Change password | Yes |
| POST | `/forgotpassord` | Request password reset code | No |
| POST | `/verifyResetPassword` | Verify reset code | No |
| POST | `/resetPassword` | Set new password | No |
| POST | `/receiveAndSendEmailMe` | Send contact message to admin | No |

### User Management — `/api/v2/user` (Admin only)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/` | Create a new user | Admin |
| GET | `/` | Get all users (paginated, searchable) | Yes |
| GET | `/:id` | Get user by ID | Admin |
| PUT | `/:id` | Update user by ID | Admin |
| DELETE | `/:id` | Delete user by ID | Admin |
| PUT | `/:id/update_password` | Update user password | Admin |

### Company — `/api/v2/company`

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/` | Create company (admin) | Admin |
| GET | `/` | Get all active companies (paginated, filterable, searchable) | No |
| GET | `/get_company_my` | Get current user's company | Yes |
| PUT | `/update_company_my` | Update own company | Yes |
| PUT | `/update_company_id/:id` | Update company by ID | Admin |
| GET | `/get_company_id/:id` | Get company by ID | No |
| DELETE | `/delete_company_id/:id` | Delete company by ID | Admin |
| DELETE | `/delete_company_my` | Delete own company | Yes |

### Advertisements

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/create_company_advertisements_my` | Create ad (1 per 24h limit) | Yes |
| DELETE | `/delete_company_advertisements_my/:id` | Delete own ad | Yes |
| DELETE | `/delete_company_advertisements_admin/:id` | Delete any ad | Admin |
| GET | `/get_company_advertisements_my/:id` | Get ads by company ID | Yes |
| GET | `/get_all_company_advertisements` | Get all ads (filterable by category) | No |
| GET | `/get_all_company_advertisements_id/:id` | Get ads by company | No |
| POST | `/likes_company_advertisements/:id` | Toggle like on ad (+2 points) | Yes |

### Comments & Reviews

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/create_company_comments/:id` | Add comment to company (+6 points) | Yes |
| DELETE | `/delete_company_comments_my/:companyId/:commentId` | Delete own comment (-6 points) | Yes |
| DELETE | `/delete_company_comments_admin/:companyId/:commentId` | Delete any comment | Admin |
| POST | `/create_company_Reviews/:id` | Add/update review (+8 points) | Yes |

### Categories

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/create_Categorey` | Create category | Admin |
| PUT | `/update_Categorey/:id` | Update category | Admin |
| GET | `/get_Categorey` | Get all categories | No |
| GET | `/get_Categorey_id/:id` | Get category by ID | No |
| DELETE | `/delete_Categorey/:id` | Delete category | Admin |
| GET | `/get_Categorey_company/:id` | Get companies in a category | No |

### Company Requests

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/create_Company_requests` | Submit company registration request | Yes |
| PUT | `/update_Company_requests` | Update pending request | Yes |
| GET | `/get_Company_requests` | Get all pending requests | Admin |
| GET | `/get_Company_requests_id/:id` | Get request by ID | Admin |
| GET | `/get_Company_requests_my` | Get own pending request | Yes |
| DELETE | `/delete_Company_requests/:id` | Delete own request | Yes |
| POST | `/Accept_Company_requests_admin/:id` | Approve request → creates company | Admin |
| DELETE | `/delete_Company_requests_admin/:id` | Reject request | Admin |

## Data Models

### User
- `name`, `email`, `phone`, `password` (bcrypt hashed)
- `role`: `user` | `admin` | `employee`
- `profilImage`, `points` (default: 20), `active`, `verificationCode`

### Company
- `name`, `description`, `companyImage`, `logoImage`
- `user` (ref → User), `categorey` (ref → Category)
- `phone`, `email`, `linkedIn`, `facebook`, `instagram`
- `ratingsAverage` (1–5), `ratingsQuantity`, `reviews[]`, `comments[]`
- `subscription`: `{ type, startDate, endDate }` — types: `سنوي` | `ثلاث شهور` | `شهري`
- `Country`, `city`, `street`

### Category
- `name`, `slug`, `description`, `Categoreyimage`

### Advertisement
- `Image[]`, `title`, `description`
- `Company` (ref → Company), `categorey` (ref → Category)
- `likes[]` (ref → User)

## Key Features

- **Email Verification:** Users receive a 6-digit code on signup; account is created only after verification.
- **Password Reset:** Forgot password flow sends a verification code via email.
- **Subscription System:** Companies have subscription plans (yearly, 3-month, monthly) with auto-calculated end dates.
- **Subscription Reminders:** A cron job runs daily at midnight and emails companies whose subscriptions expire within 5 days.
- **Points System:** Users earn/lose points for engagement — likes (+2), comments (+6), reviews (+8).
- **Image Upload:** Profile images, company images, logos, ads, and category images are uploaded via Multer and processed with Sharp (resized, compressed to JPEG).
- **Role-Based Access:** Middleware enforces `admin` and `user` role restrictions on routes.
- **Pagination & Search:** Company listing supports pagination, keyword search, and category filtering.
- **Rate Limiting & Security:** Uses `express-rate-limit`, `hpp`, `xss-clean`, and `express-mongo-sanitize`.

## License

ISC
