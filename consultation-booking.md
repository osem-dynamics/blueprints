---
title: Consultation Booking (with human review)
summary: A booking page where clients request a slot and you approve, reject, or reschedule - with email confirmations and calendar invites
best_for: Consultants and service firms who vet each request (not instant self-serve)
difficulty: advanced
setup_time: ~2-4 hours
version: 1.0
last_tested: 2026-10-07
tested_with: Claude Opus 4.8 in Claude Code
author: OSEM Dynamics
license: MIT
---

# Consultation Booking (with human review)

## What this builds
A booking flow for your website where a client **requests** a time slot, and **you**
review it - confirming, rejecting, or rescheduling, each with an optional note. On
confirm, the client gets a professional email with a calendar invite (and a video link
for video calls). It's the opposite of an instant self-serve widget: a real person
approves every meeting. This is the pattern running in production on osemdynamics.com.

## Who it's for / not for
- **For:** consultants, agencies, lawyers, advisors - anyone who wants to **vet** who
  gets on their calendar instead of letting anyone grab a slot.
- **Not for:** high-volume instant booking with no review. If you truly want
  anyone-books-anytime, a hosted tool like Cal.com or Calendly is the right call -
  this adds a human-in-the-loop on purpose.

## Prerequisites
- An existing website you can add pages and a small backend to (any modern stack).
- A database (even SQLite is plenty for a single reviewer).
- An email-sending account (e.g. a SendGrid API key) with a **verified sending domain**.
- Optional: a Google account if you want auto-generated calendar events / video links.

---

## Instructions for the AI assistant

> You are setting this up **with** the user in their codebase. Prefer their existing
> stack and conventions. Explain each step. Follow the hard rules at the bottom.

### 1. Ask before building
Ask and wait for answers:
- What's the site's stack (framework, language, database)? Match it.
- One reviewer, or several? (Affects admin auth.)
- Which **email provider**, and is the sending domain verified yet?
- **Phone, video, or both** for consultations? (Drives the email content.)
- Business hours, timezone, and slot length?
- For video: a **single reusable meeting link**, or a **unique link per booking**
  (the latter needs calendar-API access)?

### 2. Decision points
- **Database:** small/single-reviewer → SQLite is fine. Multi-user/high-volume → Postgres.
- **Video link:** simplest → one fixed meeting link in an env var. Proper → generate a
  unique link per booking via the user's calendar API (heavier; only if they want it).
- **Reviewer auth:** one reviewer → a single password + signed session cookie. Several →
  real accounts.

### 3. Build steps (goals, not fixed commands - adapt to their stack)
1. **Data model.** A `bookings` table: contact info, requested date/time, consultation
   type (phone|video), `status` (pending|confirmed|rejected|rescheduled), an admin note,
   responded-at, and created-at. **Slot-holding rule:** pending, confirmed, AND
   rescheduled all *hold* the slot; only rejected frees it. Your "is this slot free?"
   query must exclude rejected rows - not just confirmed ones.
2. **Public request form.** Collects contact info, a date/time from available slots, and
   **requires phone vs. video**. On submit, create a `pending` row. The success message
   must say "request sent - we'll confirm shortly," **never** "booked/confirmed," since
   nothing is confirmed yet. Add a honeypot field + basic rate limiting against spam.
3. **Availability.** Show only free, future slots within business hours, in the business
   timezone. Convert local time → UTC **DST-aware** (never hardcode a fixed offset).
4. **Admin review queue** (behind auth): tabs for pending / confirmed / rejected /
   rescheduled, with a pending-count badge. Each pending row gets **Confirm**,
   **Reject**, and **Reschedule** (a slot picker that excludes the booking's own current
   slot, so it never conflicts with itself).
5. **Emails** on the reviewer's decision: confirm, reject, reschedule. Reschedule emails
   are worded as **final** ("your new time is X"), not a re-proposal. Send from a verified
   domain address. Attach an `.ics` calendar invite and an "Add to Google Calendar" link,
   both built from **one shared function** so they never drift.

### 4. Verify
- Submit a request → it appears as `pending`; the slot disappears from availability.
- Confirm it → the client email arrives with the right details + a working calendar
  invite; the slot stays held.
- Reschedule → the client gets the new time and the calendar invite updates.
- Reject → the slot frees up again.
- Confirm a **real email actually lands in an inbox** and the links in it work (see
  troubleshooting - this is where people get burned).

### 5. Hard rules
- **Never store API keys or the admin password in code.** Use env vars / a secrets file.
- **Never delete the user's data** without asking; status changes are reversible, deletes are not.
- **Gate every admin action** behind auth - not just the admin page, the actions too.
- **If a step fails twice, stop and ask.**

---

## Troubleshooting
- **Emails land in spam / links show a cert error:** SendGrid (and similar) rewrite every
  link to a tracking subdomain that has no valid SSL unless you do extra DNS. **Disable
  click + open tracking per message** so links stay untouched - then test by clicking a
  link in a *real* inbox, not just a preview tool.
- **"Slot already taken" on reschedule:** make sure the availability check **excludes the
  booking's own row**, or it conflicts with itself.
- **Times off by an hour half the year:** you hardcoded a UTC offset. Convert through a
  timezone-aware utility instead.

## Going further
- Add a one-time "Connect Google" flow to auto-create calendar events + a unique video
  link per booking.
- Add an "add to my calendar" button for the reviewer.

> **Need this customized, secured, or running in production?**
> OSEM Dynamics builds and maintains these for real businesses. osemdynamics.com/contact
