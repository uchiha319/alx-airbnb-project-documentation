Airbnb Clone – Features & Functionalities

This document lists the minimum viable scope for an Airbnb-like backend. It is organized by capability area.

## 1) Authentication & Accounts
- Email/password signup & login (JWT-based sessions)
- Password reset (email token) and change password
- Social login (optional: Google OAuth)
- Profile management (name, avatar, phone, government ID status)
- Roles & permissions: **Guest**, **Host**, **Admin**
- Device/session management (logout from all devices)

## 2) Property (Listing) Management
- Create/read/update/archive listings
- Listing fields: title, description, photos, type, amenities, rules, location (lat/lng + address), base price, currency, capacity, availability
- Photo upload & management
- Pricing: nightly/base price, cleaning fee, discount rules (weekly/monthly), dynamic pricing (optional)
- Calendar availability (blackout dates, min/max nights)
- Search & filters: location + date range + guests, type, price range, amenities
- Reviews & ratings for listings (and for hosts by guests)

## 3) Booking & Reservations
- Quote calculation (nights × price + fees + taxes - discounts)
- Request-to-book and instant-book flows
- Pre-authorization and confirmation after payment
- Cancellation policies and refunds
- Booking lifecycle: pending → confirmed → completed/cancelled
- Messaging between guest and host (optional MVP: booking-level conversation thread)

## 4) Payments
- Stripe integration (Payment Intents) for card payments
- Payouts to hosts (Stripe Connect)
- Wallet/ledger entries for platform fee, host earnings, guest charges
- Refunds & partial refunds; security deposit (optional)
- Currencies & FX (optional via Stripe)

## 5) Compliance & Safety
- KYC for hosts (via Stripe identity) – optional for MVP
- Government ID verification (optional provider)
- Content moderation (image/description), abuse reporting
- Audit logs for sensitive actions

## 6) Admin & Ops
- Admin portal endpoints: user management, listing takedown, refunds, disputes
- Metrics & health: bookings/day, GMV, cancellations, uptime, error rates
- System emails/notifications (transactional & alerts)

## 7) Platform/Infra
- RESTful API, versioned: `/api/v1`
- Tech (suggested): Node.js + Express, PostgreSQL, Redis (caching, rate limiting), S3-compatible storage for images
- AuthN/Z: JWT access + refresh tokens, RBAC middleware
- Observability: structured logs, tracing, error reporting
