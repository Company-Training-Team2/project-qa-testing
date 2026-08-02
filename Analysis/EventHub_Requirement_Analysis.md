# EventHub — Requirement Analysis

**Prepared by:** QA (Requirement Analysis)

---

## 1. Project Overview

**Product:** EventHub — Event Planning Platform (Web & Mobile)
**Purpose:** A centralized platform for customers to plan events, discover vendors, compare services, submit booking requests, pay for accepted bookings, and manage event budgets.
**Roles:** Customer, Vendor, Admin
**Platforms:** Responsive Web, Mobile (Android & iOS)

### In Scope (MVP)
Authentication · Event Creation · Vendor Search & Filter · Vendor Profile & Work Postings · Booking · Payment · Budget Manager · Favorites · Reviews · Vendor Dashboard · Admin Dashboard

### Out of Scope
Loyalty Program · AI Smart Recommendations · Corporate Events · Birthday Events · Baby Shower · Digital Contracts

---

## 2. Business Flows

### 2.1 Customer Journey
Register / Login (with email verification) → Create Event → Budget Initialized → Search Vendors → View Vendor Profile → Favorite (optional) → Submit Booking Request (PENDING) → Vendor Accepts / Rejects → Pay for Accepted Booking → PAID → Booking Completed → Submit Review

### 2.2 Vendor Journey
Register → Wait for Account Approval → Login → Manage Services / Work Postings → Each Work Posting requires its own Admin Approval (separate from account approval) → Manage Availability Calendar → Receive Booking Requests → Accept / Reject / Cancel → View PAID status & payout per booking → View Statistics

### 2.3 Admin Journey
Login only — no public registration, optional MFA → Review Vendor Account Requests → Approve / Reject → Review Work Posting Requests → Approve / Reject → Manage Users / Categories → Monitor Bookings → Global Payment Ledger → Issue Refunds → Flag Suspicious Transactions → View Payment KPIs

---

## 3. General Business Rules

- One customer can create multiple events.
- Vendor accounts require Admin approval before gaining portal access.
- Each individual Work Posting requires its own Admin approval before appearing publicly, separate from account-level approval.
- Only approved vendors / published postings appear in search results.
- Event date cannot be in the past; event budget must be greater than zero.
- Double booking is not allowed (Vendor blocks dates on an availability calendar).
- Customers can submit reviews only after a booking is COMPLETED.
- Remaining budget is automatically recalculated after each expense.
- Payment can only be initiated when Booking Status = ACCEPTED.
- Failed payment leaves the booking in ACCEPTED status (retry allowed).
- Refunds are an Admin-only manual action in the MVP.
- Admin accounts cannot be created via any public registration flow.

---

## 4. Overall Risks

- Booking conflicts if concurrency isn't handled correctly.
- Budget calculations inconsistent if expenses and payments aren't synchronized (see Open Question on Spent Budget timing).
- Vendors or Work Postings visible before approval if validation is missing (two-tier check: account + posting).
- Invalid event data stored if validations are missing.
- Incorrect booking status transitions across the full chain (Pending → Accepted → Paid → Completed / Rejected).
- Missing authorization checks exposing restricted features.
- Payment gateway integration carries real security/compliance weight; PCI handling should sit with the gateway provider, not be built in-house.
- No defined SLA for Admin review of vendor/posting approval queues could bottleneck the sprint.

---

## 5. Open Questions

1. What is the booking cancellation policy — can Customers cancel a submitted request, and can Vendors cancel an already-accepted booking?
2. Are notifications sent when booking or payment status changes, and through which channel?
3. What is the error-handling strategy across the platform?
4. Are there limits on uploaded vendor images (size, format, count)?
5. Can users edit their own profile after registration?
6. What are the default/available sorting options in search, beyond filters?
7. Can Customers edit event details after creation?
8. Can a Customer have multiple active events at the same time?
9. Does Spent Budget update once (at booking Accepted) or again at Payment confirmation — needs confirmation to avoid double-counting.
10. What exactly can a Vendor access before their account is approved, and before their first Work Posting is approved?
11. Which payment gateway/provider will be integrated?
12. Do partial refunds get reflected back into the Customer's Spent/Remaining Budget?
13. What is the Admin review SLA for vendor accounts and work postings?
14. Where/when is a Vendor's payout bank account collected?
15. Which specific Digital Wallet providers are supported?
16. Is Admin MFA mandatory or optional at launch?
17. What happens to a PAID booking if the Vendor needs to cancel afterward?

---

## 6. Module Analyses

### 6.1 Authentication
**Description:** Secure registration, login, and account management for Customer, Vendor, and Admin roles.

**Functional Requirements**
- FR-01: Customers and Vendors can register (Email/Password, or Social Auth for Customers).
- FR-02: User profile created after successful registration.
- FR-03: Login via email and password.
- FR-04: Forgot Password functionality.
- FR-05: Session persists until logout/expiration.
- FR-06: Redirect to correct dashboard based on role.
- FR-07: Admin access is Login-only via a dedicated page; public registration is disabled.
- FR-08: Optional MFA supported for Admin login.
- FR-09: Email verification required for Customer registration.

**Business Rules**
- BR-01: Email must be unique.
- BR-02: Password must be at least 8 characters.
- BR-03: Vendor accounts require Admin approval before full portal access.
- BR-04: Only authenticated users can access protected features.
- BR-05: Each account is associated with a single role.
- BR-06: Admin accounts cannot be self-registered.

**Acceptance Criteria:** Registration succeeds with valid data · duplicate email is rejected · login redirects correctly per role · Admin login page has no registration path · Customer registration requires email verification.

**Dependencies:** User Database, Authentication Service, Role Management, Profile Management, Admin Approval Process, Email Verification Service, MFA Service (Admin only).

**Assumptions:** Users have valid emails; backend auth service is available; Admin accounts are pre-provisioned outside the app.

**Risks:** Duplicate accounts, unauthorized access, weak passwords, session vulnerabilities, incorrect validation logic; inconsistent MFA enforcement if left optional.

---

### 6.2 Event Creation
**Description:** Allows Customers to create an event that serves as their main planning workspace.

**Functional Requirements**
- FR-01: Customer can enter an event name.
- FR-02: Customer can select the event type.
- FR-03: Customer can select the event date.
- FR-04: Customer can select the event city.
- FR-05: Customer can enter the guest count.
- FR-06: Customer can define the event budget.

**Business Rules**
- BR-01: Budget must be greater than zero.
- BR-02: Event date cannot be in the past.
- BR-03: One user can create multiple events.

**Acceptance Criteria:** Event is saved successfully · event appears on the user's dashboard.

**Dependencies:** Event Database, Authentication, Budget Manager (auto-initializes on event creation).

**Assumptions:** User is authenticated before creating an event; city list is predefined.

**Risks:** Invalid event data stored if validation is missing.

---

### 6.3 Vendor Search & Filter
**Description:** Allows Customers to discover approved vendor work listings based on their preferences.

**Functional Requirements**
- FR-01: Customer can search/filter vendor listings by Category.
- FR-02: Customer can search/filter by City.
- FR-03: Customer can search/filter by Price Range.
- FR-04: Customer can search/filter by Rating.
- FR-05: Multiple filters can be combined in a single search.

**Business Rules**
- BR-01: Only approved vendors and published postings appear in search results.

**Acceptance Criteria:** Results update correctly after applying filters.

**Dependencies:** Vendor Profile / Work Listing data, Category Management.

**Assumptions:** Category list is Admin-managed and stable.

**Risks:** Unapproved listings could leak into search results if the approval check is missing.

---

### 6.4 Vendor Profile & Work Postings
**Description:** Displays a vendor's approved work postings — images, description, pricing, availability calendar, and reviews. Each work posting requires its own Admin approval, separate from the vendor's account-level approval.

**Functional Requirements**
- FR-01: Vendor profile displays images, description, services, pricing, rating, reviews, and availability.
- FR-02: Vendor can submit a new work posting (title, description, category, pricing, images) for Admin review.
- FR-03: A newly submitted posting is set to PENDING APPROVAL status until reviewed.

**Business Rules**
- BR-01: Only approved vendor accounts may submit work postings.
- BR-02: Each posting independently requires its own Admin approval before publishing.
- BR-03: Average rating is calculated automatically.
- BR-04: Rejected postings return to the vendor with moderation notes for resubmission.

**Acceptance Criteria:** Profile is accessible from search only if approved · unapproved postings remain hidden from public search · rejected postings show moderation feedback to the vendor.

**Dependencies:** Admin Approval Queue, File/Image Storage, Category Management, Vendor Account status.

**Assumptions:** Vendors upload their own images; Admin reviews postings within a reasonable time.

**Risks:** Account-level approval and posting-level approval could be conflated in implementation, allowing an approved vendor's postings to skip individual review.

---

### 6.5 Booking
**Description:** Allows Customers to submit booking requests against a vendor's available calendar, track status through completion, including a payment step after acceptance.

**Functional Requirements**
- FR-01: Customer can select an available date and service package.
- FR-02: Customer can add notes and submit a booking request (status set to PENDING).
- FR-03: Vendor can Accept, Reject, or Cancel a booking request.
- FR-04: Once Accepted, the Customer can complete payment for the booking.
- FR-05: On successful payment, booking status becomes PAID.
- FR-06: After the event date, booking status becomes COMPLETED, unlocking the review step.

**Business Rules**
- BR-01: Double booking is not allowed (calendar-based availability).
- BR-02: Booking status updates automatically.
- BR-03: Payment can only be initiated from ACCEPTED status.
- BR-04: A failed payment leaves the booking in ACCEPTED status.
- BR-05: Refunds are an Admin-only action.

**Acceptance Criteria:** Customer can view Pending / Accepted / Paid / Rejected / Completed statuses · a rejected booking frees the date with no budget impact · a failed payment does not change the booking's status.

**Dependencies:** Vendor Availability Calendar, Payment module, Budget Manager, Notification system.

**Assumptions:** Only one accepted booking per date per vendor; the payment gateway is available and responsive.

**Risks:** The extended status chain (Pending → Accepted → Paid → Completed / Rejected) needs precise state-machine handling; concurrency risk remains on the shared calendar.

---

### 6.6 Payment
**Description:** Enables Customers to pay for Accepted bookings through an integrated payment gateway (Credit/Debit Card, Digital Wallet), stores transaction records, and gives Vendors and Admins visibility into payments.

**Functional Requirements**
- FR-01: Customer can initiate payment only when the Booking is ACCEPTED.
- FR-02: System supports Card and Digital Wallet payment methods.
- FR-03: On successful payment, the booking transitions to PAID and a transaction record is saved.
- FR-04: On failed payment, an error is returned and the booking remains ACCEPTED.
- FR-05: Vendor can view PAID status and per-booking / cumulative earnings.
- FR-06: Admin can view a Global Payment Ledger (booking reference, customer, vendor, amount, method, status, timestamp).
- FR-07: Admin can issue full or partial refunds for disputed or cancelled bookings.
- FR-08: Admin can flag suspicious transactions for manual review.
- FR-09: Admin dashboard surfaces payment KPIs (total revenue, refund rate, failed-transaction rate).

**Business Rules**
- BR-01: Payment can only be initiated from ACCEPTED status.
- BR-02: Failed transactions do not change the booking's status.
- BR-03: Refunds are an Admin-only manual action.
- BR-04: Payout to the Vendor's bank account is triggered automatically after the event completion date.

**Acceptance Criteria:** A valid payment moves the booking to PAID · an invalid or declined payment leaves the booking ACCEPTED, shows an error, and allows retry · the transaction is visible to the Customer, Vendor, and Admin according to their respective view · an Admin-issued refund updates the transaction status to REFUNDED.

**Dependencies:** Payment Gateway (provider to be selected), Booking module, Budget Manager, Notification system, Admin Dashboard.

**Assumptions:** A third-party payment gateway will be integrated; PCI-DSS compliance is handled by the gateway provider; refunds are processed manually by the Admin without an automated reversal flow.

**Risks:** Payment gateway integration is a substantial technical scope for the sprint timeline; security risk if card data ever reaches the backend directly instead of being tokenized by the gateway; double-charging risk if retry logic isn't idempotent; interaction with Budget Manager's Spent Budget logic needs confirmation (see Open Questions).

---

### 6.7 Budget Manager
**Description:** Helps users track total, spent, and remaining budget for their event, updated by both booking acceptance and payment confirmation.

**Functional Requirements**
- FR-01: Total budget is set at event creation.
- FR-02: System tracks spending.
- FR-03: System displays remaining budget.
- FR-04: System shows spending by category.

**Business Rules**
- BR-01: Remaining budget is automatically recalculated after each expense.
- BR-02 (pending confirmation): Whether Spent Budget updates at booking Accepted, at Payment confirmation, or both — see Open Questions.

**Acceptance Criteria:** Budget calculations are accurate · remaining budget equals total minus spent at all times.

**Dependencies:** Event Creation (initializes the budget), Booking module, Payment module.

**Assumptions:** All expenses flow through Booking/Payment; there is no manual, off-platform expense entry in the MVP.

**Risks:** Risk of double-counting a booking's cost across Accepted and Paid; the budget must correctly reverse if a booking is rejected after being counted as spent.

---

### 6.8 Favorites
**Description:** Allows Customers to save vendors for later comparison.

**Functional Requirements**
- FR-01: Customer can add a vendor to Favorites.
- FR-02: Customer can remove a vendor from Favorites.

**Business Rules:** None specified.

**Acceptance Criteria:** Favorites list updates instantly.

**Dependencies:** Vendor Profile data, User session.

**Assumptions:** This feature is used by Customers only.

**Risks:** Low — standard data consistency only.

---

### 6.9 Reviews
**Description:** Allows Customers to rate and review completed bookings.

**Functional Requirements**
- FR-01: Customer can give a rating (1–5 stars).
- FR-02: Customer can write a review.

**Business Rules**
- BR-01: Only Customers with a COMPLETED booking may submit a review.

**Acceptance Criteria:** The review appears on the vendor's profile.

**Dependencies:** Booking module (status = COMPLETED), Vendor Profile.

**Assumptions:** One review per completed booking.

**Risks:** A booking that isn't actually completed could be reviewed if the status check is missing.

---

### 6.10 Vendor Dashboard
**Description:** Gives Vendors tools to manage their services, bookings, availability, and payment/payout visibility.

**Functional Requirements**
- FR-01: Vendor can manage services / work postings.
- FR-02: Vendor can manage their availability calendar.
- FR-03: Vendor can accept or reject bookings.
- FR-04: Vendor can view booking statistics.
- FR-05: Vendor can view PAID status per booking and cumulative revenue/payout.

**Business Rules**
- BR-01: Dashboard reflects PAID status immediately once the Customer completes payment.
- BR-02: Payout to the Vendor's bank account triggers automatically after the event completion date.

**Acceptance Criteria:** Dashboard updates after every booking and every payment event.

**Dependencies:** Booking module, Payment module, Availability Calendar.

**Assumptions:** Vendor has a registered bank account on file for payout.

**Risks:** Payout handling is undefined if the Vendor's bank details are missing or invalid at payout time.

---

### 6.11 Admin Dashboard
**Description:** Gives the Admin tools to manage approvals, users, categories, bookings, and platform-wide payment oversight.

**Functional Requirements**
- FR-01: Admin can approve or reject vendor accounts.
- FR-02: Admin can approve or reject individual work postings.
- FR-03: Admin can manage users and categories.
- FR-04: Admin can monitor bookings.
- FR-05: Admin can suspend accounts.
- FR-06: Admin can view the Global Payment Ledger.
- FR-07: Admin can issue refunds.
- FR-08: Admin can flag suspicious transactions.
- FR-09: Admin can view payment KPIs.

**Business Rules**
- BR-01: Changes are reflected immediately.
- BR-02: Refunds are a manual Admin action only.

**Acceptance Criteria:** Changes are reflected immediately · a processed refund updates the transaction status to REFUNDED in the ledger.

**Dependencies:** Payment module, Vendor & Posting approval workflows, User/Category Management, Booking Monitoring.

**Assumptions:** A single Admin role with full access (no tiered permissions specified).

**Risks:** No defined SLA for approval queues; no defined process for in-flight bookings if a Vendor account is suspended mid-cycle.

---

## Next Steps
1. Resolve the Open Questions in Section 5 with the team.
2. Build the Test Plan from Sections 1–5 of this document.
3. Derive Test Scenarios / Cases per module from Section 6, module by module.
