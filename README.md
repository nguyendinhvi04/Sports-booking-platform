1) System Architecture Overview
The system is organized as a monorepo structure:
client/frontend: Frontend application built with Vue 3 + Vite.
server/backend: API server built with Next.js (App Router) + Prisma + PostgreSQL.
docker + docker-compose*.yml: Container environments for development and production.
docs: System documentation covering architecture, database, APIs, and deployment.
script: Internal testing scripts for API and database operations.
Core Processing Flow:
The frontend sends requests to the API (/api/...) with JWT tokens when authentication is required.
Backend route handlers validate incoming data (Zod), check permissions, and invoke domain-specific services.
Services interact with the database via Prisma and emit real-time events through Socket.IO as needed.
The backend returns a standardized JSON response (success, message, data).

2) Tech Stack
Frontend (client/frontend)
Vue 3, Vue Router, Vite
Axios
Bootstrap + Font Awesome + Lucide
Mapbox GL JS
Socket.IO Client
AI SDK for Vue (@ai-sdk/vue)

Backend (server/backend)
Next.js 16 (App Router configured as API server)
TypeScript + tsx
Prisma ORM + PostgreSQL
Socket.IO + Redis adapter (@socket.io/redis-adapter, ioredis)
JWT, Google Login, Facebook Login
Stripe SDK, Cloudinary, Nodemailer
AI SDK + Google provider (ai, @ai-sdk/google)
Cron jobs (node-cron) for automated business logic
Unit testing with Vitest

3) Business Capabilities
Customer (USER)
Account registration, login, password change, forgot, and reset flows.
Browse clubs and courts, and find nearby sports venues based on current location.
View club details by slug, and inspect available time slots by court and date.
Create bookings, process payments, and track orders by code.
Manage favorite clubs and courts.
Submit post-booking reviews and ratings.
View club announcements, updates, and posts.
Interact with an intelligent AI chatbot.

Field Owner (OWNER)
Onboarding and KYC document submission.
Manage clubs, courts, amenities, opening hours, and pricing tiers.
Manage time slots, lock slots manually, and monitor club bookings.
Create manual walk-in bookings and confirm cash payments on-site.
Manage club customer lists and view consumption history.
Create and manage promotional vouchers and marketing posts.
Handle customer refund requests.

Administrator (ADMIN)
User and field owner account management.
Review and approve KYC profiles.
Review and approve club/court operating statuses.
Monitor overall system statistics and analytics.

4) API Surface (Core Endpoint Groups)
The backend contains 60+ route handlers under server/backend/src/app/api, grouped as follows:
auth/*: login, register, logout, forgot-password, reset-password, social login
users/*: user profiles, password updates
clubs/*, courts/*, bookings/*, reviews/*, favorites/*
vouchers/*, notifications/*, posts/*
payments/*: payment webhooks and payment verification
owner/*: complete suite of field owner management APIs
admin/*: complete suite of administrative operations APIs
chat: AI assistant endpoints
upload: image upload handling

Standardized response format used across all endpoints:
JSON
{
  "success": true,
  "message": "Notification message",
  "data": {}
}
5) Database Schema (Prisma + PostgreSQL)
Schema location: server/backend/prisma/schema.prisma featuring core model groups:
Identity & Auth: User, UserProfile, Session, PasswordReset, OwnerProfile
Club & Court: Club, ClubImage, OpeningHour, Court, CourtImage
Pricing & Slot: CourtPricing, SpecialDatePricing, TimeSlot
Booking & Payment: Booking, BookingItem, BookingService, Payment, BookingStatusHistory
Engagement & Marketing: Review, ReviewImage, Voucher, VoucherUsage, Post, Notification
CRM & Governance: ClubCustomer, AuditLog, SystemConfig, FavoriteClub, FavoriteCourt

Core Business Enums:
Role: USER, OWNER, ADMIN
SportType: FOOTBALL, BADMINTON, TENNIS, PICKLEBALL, BASKETBALL, ...
BookingStatus: PENDING, WAITING_PAYMENT, CONFIRMED, CANCELLED, COMPLETED
PaymentStatus: PENDING, WAITING_PAYMENT, CONFIRMED, CANCELLED, REFUNDED

6) Real-time, Jobs, and Automation
Socket server initialized in server/backend/server.ts.
Domain listeners:
src/modules/booking/booking.listener.ts
src/modules/slot/slot.listener.ts
Automated cron job expire-bookings.job.ts runs every minute to:
Identify expired WAITING_PAYMENT bookings.
Update booking and payment statuses to CANCELLED.
Release associated TimeSlot instances back to AVAILABLE.
Broadcast real-time signals to connected clients.

7) Directory Structure
Plaintext
source/
├── client/
│   └── frontend/                 # Vue 3 application
├── server/
│   └── backend/                  # Next.js API + Prisma + Socket + jobs
├── docs/                         # System documentation
│   ├── _architecture/
│   ├── _api/
│   ├── _database/
│   ├── _deploy/
│   ├── _mac/
│   └── _redis/
├── docker/
│   ├── client.Dockerfile
│   ├── server.Dockerfile
│   └── postgres/init.sql
├── docker-compose.dev.yml
├── docker-compose.yml
└── script/
    └── test/
8) Execution Guide
8.1 Environment Requirements
Node.js: per engine specified in package.json (^20.19.0 or >=22.12.0)
npm
PostgreSQL
Redis

8.2 Docker Execution (Recommended for Fast Onboarding)
Bash
# 1) Create root environment file
cp .env.example .env
# 2) Start development environment
docker compose -f docker-compose.dev.yml up -d
Notes:
docker-compose.dev.yml includes postgres, redis, backend, and frontend.
docker-compose.yml is configured for the production profile.

8.3 Local Execution (Without Docker)
Backend:
Bash
cd server/backend
cp .env.example .env
npm install
npx prisma generate
npx prisma migrate deploy
npm run dev

Frontend:
Bash
cd client/frontend
cp .env.example .env
npm install
npm run dev
9) Critical Environment Variables
Root (.env) - used for Docker Compose
POSTGRES_USER, POSTGRES_PASSWORD
JWT_SECRET, JWT_EXPIRES_IN
GEMINI_API_KEY, STRIPE_SECRET_KEY
Backend (server/backend/.env)
DATABASE_URL
REDIS_URL
JWT_SECRET, JWT_EXPIRES_IN
SMTP_* (forgot/reset password services)
GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
FACEBOOK_APP_ID, FACEBOOK_APP_SECRET
CLOUDINARY_*
GEMINI_API_KEY / GOOGLE_CLOUD_API_KEY
NEXT_PUBLIC_MAPBOX_TOKEN

Frontend (client/frontend/.env)
VITE_API_URL
VITE_SOCKET_URL
VITE_MAPBOX_TOKEN
VITE_GOOGLE_CLIENT_ID
VITE_FACEBOOK_APP_ID

10) Primary Scripts
Backend
npm run dev: launch development server
npm run build: compile Next.js backend
npm run start: run production build
npm run lint: execute code linting
npm run test: run Vitest test suite
npm run seed:config, npm run seed:vouchers, npm run seed:vouchers-test
Frontend
npm run dev
npm run build
npm run preview

11) Detailed Documentation
Architecture: docs/_architecture/system-architecture.md
Database Design: docs/_database/database-design.md
API Overview: docs/_api/api-system.md
Production Deployment: docs/_deploy/deployment.md
macOS Local Setup: docs/_mac/local-setup-mac.md
Windows Redis Setup: docs/_redis/redis-setup-windows.md

12) Testing and Quality Assurance
Chatbot tools testing included under server/backend/src/app/api/chat/*.test.ts.
Additional unit tests are recommended for booking, payment, refund, and owner/admin workflows prior to go-live deployment.

13) Operational Notes
Never commit sensitive information in .env files.
Change JWT_SECRET from default values prior to production deployment.
Enable HTTPS, restrict CORS origins, and set up automated database backup schedules in production environments.

14) License
This project serves as a graduation capstone project and is owned exclusively by the development team.






