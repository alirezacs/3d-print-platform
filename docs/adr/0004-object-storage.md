# ADR 0004 — Use S3-Compatible Object Storage Behind StorageService

**Status:** Accepted
**Date:** 2026-10-05

## Context
The platform stores product images, customer references, generated images, 3D models, production files, personalization images, and invoice PDFs.

## Decision
Use **S3-compatible object storage** behind an internal `StorageService` abstraction.

Initial direction:
- Local: MinIO
- Staging/Production: S3-compatible provider selected later

## Rules
- private assets are private by default;
- signed/authenticated access is used where appropriate;
- business modules do not use provider SDKs directly;
- PostgreSQL stores file metadata/object keys, not large blobs.

## Consequences
This reduces vendor lock-in and local-disk dependency, but requires explicit lifecycle, access-control, and orphan-cleanup logic.
