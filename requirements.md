# Requirements – Airbnb Clone Backend

This document specifies technical and functional requirements for three core features: **Authentication**, **Property Management**, and **Booking**. A fourth section outlines **Payments** integration basics.

---
## 1) Authentication & Authorization

**API Base:** `/api/v1/auth`  
**Tokens:** JWT (access ~15m, refresh ~7d).  
**Password Hashing:** bcrypt (≥ 12 rounds).

### Endpoints
- `POST /register` → body: `{ email, password, name }` → 201 Created; sends verification email (optional).
- `POST /login` → body: `{ email, password }` → 200 OK `{ accessToken, refreshToken }`.
- `POST /token/refresh` → body: `{ refreshToken }` → 200 OK `{ accessToken }`.
- `POST /logout` → invalidates refresh token (server-side blacklist / rotating tokens).
- `GET /me` (auth) → 200 OK profile.
- `PATCH /me` (auth) → update profile fields.
- `POST /password/forgot` → email with reset link/token.
- `POST /password/reset` → body: `{ token, newPassword }`.

### Validation
- Email must be RFC compliant and unique.
- Password ≥ 8 chars, 1 letter, 1 number (configurable).
- Rate-limit: login 5/min/IP + account; password reset 3/h/email.

### Security
- JWT in Authorization header `Bearer <token>`.
- Refresh token rotation, revoke on logout/reset.
- Brute-force protection & device lockout on repeated failures.
- Audit log for auth events.

---
## 2) Property (Listing) Management

**API Base:** `/api/v1/listings`

### Endpoints
- `POST /` (host) → create listing.
- `GET /:id` → fetch listing by id.
- `PATCH /:id` (host owner or admin) → update.
- `DELETE /:id` (soft delete/archive).
- `GET /` → search/query with params: `q, lat, lng, radius, checkIn, checkOut, guests, priceMin, priceMax, type, amenities[]`.
- `POST /:id/photos` (host) → upload photos (multipart).
- `DELETE /:id/photos/:photoId` (host/admin).
- `GET /:id/availability` → returns available dates for a window.
- `PUT /:id/calendar/blocks` (host) → set blackout dates.
- `POST /:id/reviews` (guest with completed stay).

### Data Model (relational sketch)
- `users(id, role, email, password_hash, name, phone, created_at)`
- `listings(id, host_id, title, description, type, address, lat, lng, capacity, base_price, currency, cleaning_fee, status, created_at)`
- `listing_photos(id, listing_id, url, position)`
- `amenities(id, name)`; `listing_amenities(listing_id, amenity_id)`
- `calendar_blocks(id, listing_id, start_date, end_date)`
- `reviews(id, listing_id, author_id, rating, comment, created_at)`

### Validation
- Title 1..120 chars; description ≤ 5000.
- Base price ≥ 0; currency ISO-4217.
- Geolocation within valid ranges.
- Host must own the listing to mutate.

---
## 3) Booking & Reservations

**API Base:** `/api/v1/bookings`

### Quote & Availability
- `GET /quote?listingId&checkIn&checkOut&guests` → `{ nights, nightlyPrice, fees, taxes, total }`
- Reject if any date intersects `calendar_blocks` or existing confirmed booking.

### Booking Flow
- `POST /` (guest) → body: `{ listingId, checkIn, checkOut, guests, paymentMethodId }`  
  Response: `201 Created { bookingId, status: "pending_payment", clientSecret }` (Stripe intent).
- `POST /:id/cancel` (guest/host/admin by policy)  
  Refunds calculated by policy and days to check-in.

### States
- `pending_payment` → `confirmed` (payment succeeded) → `completed` (after checkout)  
  or → `cancelled`.

### Data Model
- `bookings(id, listing_id, guest_id, check_in, check_out, guests, subtotal, fees, taxes, total, status, created_at)`

### Validation & Rules
- Date range valid and checkIn < checkOut.
- Guests ≤ listing.capacity.
- Min/max nights per listing.
- One active booking per listing per date.

---
## 4) Payments (Stripe)

**API Base:** `/api/v1/payments`

- Use Stripe **Payment Intents** for SCA-ready flows.
- Use Stripe **Connect** for host payouts; platform retains fee.
- Webhooks:
  - `payment_intent.succeeded` → mark booking `confirmed`
  - `charge.refunded` → update booking, create ledger entry
- Idempotency keys on payment endpoints.
- Store only Stripe IDs; never store raw PAN data.

### Non-Functional
- P95 latency ≤ 300ms for read endpoints; ≤ 700ms for write under nominal load.
- 99.9% uptime target; healthcheck at `/healthz`.
- Observability: request IDs, structured logs, metrics.
