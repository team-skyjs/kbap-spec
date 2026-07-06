<!-- SPECKIT START -->
Product: K-Bap (display name) · slug `kbap` (repos kbap-fe/kbap-be, package com.kbap) · codename kfood (this dir)

Active feature: 001-personalized-menu-mvp
- Plan: specs/001-personalized-menu-mvp/plan.md
- Spec: specs/001-personalized-menu-mvp/spec.md
- API contract (SSOT): **BE Swagger** — https://meogo.handev.site/swagger-ui (meeting 2026-07-02). This repo's specs/001-personalized-menu-mvp/contracts/openapi.yaml is reference-only, NOT the contract SSOT.
- Constitution: .specify/memory/constitution.md

Stack: FE = React Native + Expo (TypeScript); BE = Kotlin + Spring Boot + MongoDB
(AWS DocumentDB); API = OpenAPI/REST (contract-first). OCR = on-device ML Kit;
translation = BE server-side. 3-repo polyrepo (spec/fe/be); this is the spec repo (SSOT).
Read the plan for full technical context, project structure, and integration points.
<!-- SPECKIT END -->
