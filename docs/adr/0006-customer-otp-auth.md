# ADR 0006 — Use Mobile + OTP as Primary Customer Authentication

**Status:** Accepted
**Date:** 2026-10-05

## Context
The initial market is Iran-focused and mobile-number authentication is appropriate for the customer experience.

## Decision
Use **mobile + OTP** as the primary customer authentication mechanism.

Email is optional in MVP.

Staff authentication is separate and stronger, with secure password support and future 2FA capability.

## Security Requirements
- OTP expiry;
- one-time usage;
- resend cooldown;
- attempt limiting;
- IP/mobile abuse controls;
- no OTP logging;
- revocable sessions.

## Consequences
The customer flow becomes simpler, but SMS reliability, cost, and abuse protection become important operational concerns.
