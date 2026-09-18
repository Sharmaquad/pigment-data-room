# Pigment Data Room

Internal content-management tool for Pigment Skin & Hair Clinic (Haralur Road, Bangalore). Manages treatments, doctors, locations, FAQs, testimonials, before/after gallery, blog posts, and a treatment-page audit checklist — all backed by Firebase (Firestore + Storage), live-synced across everyone editing at once.

Access is password-gated (ask Rahul or Shreyash for the current password). No setup needed to view/edit — it's a single static page.

## Stack
- Firebase Firestore — content storage, live sync via `onSnapshot`
- Firebase Storage — images (doctor photos, treatment photos, before/after pairs, logo)
- Firebase Anonymous Auth — gates access behind the shared password (client hashes the entered password and compares against a stored hash; on match, signs in anonymously so Firestore/Storage security rules — which require `request.auth != null` — allow reads/writes)

## Local rules deploy
```
firebase deploy --only firestore:rules,storage:rules --project pigment-data-room
```
