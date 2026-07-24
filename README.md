# M. Sarib Randhawa — Ultra-Realistic 3D Portfolio + Business System

A single, self‑contained `index.html` (HTML5 + CSS3 + vanilla ES6 JavaScript) that combines a
premium 3D creative portfolio with a full **User Panel** and **Owner Admin Panel**, plus
authentication and real‑time data sync.

## ✨ What's included

- **Public site** — 3D hero (rotating cube avatar, particle canvas, mouse parallax, typing
  animation, animated counters), glassmorphism sticky nav, 3D flip‑card work grid, interactive
  **skills orbital sphere**, about + contact sections, dark/light toggle, language selector.
- **Auth** — Email/Password, Google login (Firebase) and Owner login. Sign up, log in,
  forgot‑password, session persistence, blocked‑user check.
- **User Panel** — Overview, editable Profile (avatar/info/change password), My Orders table with
  live status, Place‑Order modal (category grid + auto ID + drag‑drop files), Hire‑Me form,
  Settings (delete account / logout).
- **Owner Panel** — Dashboard stats, Site Settings (live edits to the public site), Order
  Management (search/filter/status/update/delete/notify), User Management
  (block/unblock/delete/email/export CSV), Category Management, Owner Credentials.
- **Real‑time sync** — Owner changes (profile, categories, order status, blocking) instantly
  reflect on the public site and user panel.

## 🚀 Running it

**Demo mode (no setup, works immediately):** just open `index.html` in a browser. All data is
stored in `localStorage` and synced in real‑time within the same browser. Default owner login:
**`Sarib` / `@Saribmadni786`**.

**Firebase mode (production):** open `index.html`, scroll to the `CONFIG` block near the top of
the `<script>`, paste your Firebase web config into `FIREBASE_CONFIG`, and set `USE_FIREBASE = true`.
Also create a **Firestore** database with these collections/documents:

```
settings/owner        → { name, cast, exp, desc, phone, email, location, adminUser, adminPass, createdAt, updatedAt }
users/{userId}        → { name, username, email, phone, status, avatar, createdAt, updatedAt }
orders/{orderId}      → { orderId, customer, username, email, phone, category, description, attachments[], status, date, userId, createdAt, updatedAt }
categories/{catId}    → { name, createdAt }
```

> **Important — security model.** In this build the **Owner is NOT a Firebase Auth user**
> (per the spec, owner login compares a password stored in `settings/owner`). That means Owner
> Panel writes hit Firestore *without* `request.auth`. The rules below make the app fully
> functional, but they are **open** — anyone with the Firestore API could write these
> collections. For production, harden this with a Cloud Function / admin SDK that writes on
> behalf of the owner, or make the owner a Firebase Auth user with a custom `admin` claim and
> gate writes on `request.auth.token.admin`.

**Working rules (functional, but open — fine for a prototype):**

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /settings/{doc}     { allow read: if true; allow write: if true; }
    match /categories/{id}    { allow read: if true; allow write: if true; }
    match /users/{uid}        { allow read: if true; allow write: if true; }
    match /orders/{id}        { allow read: if true; allow write: if true; }
  }
}
```

**Hardened rules (recommended for production — requires an admin claim):**

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /settings/{doc}  { allow read: if true; allow write: if request.auth.token.admin == true; }
    match /categories/{id} { allow read: if true; allow write: if request.auth.token.admin == true; }
    match /users/{uid}     { allow read: if true; allow write: if request.auth.token.admin == true
                                              || request.auth.uid == uid; }
    match /orders/{id}     { allow read: if true; allow write: if request.auth.token.admin == true
                                              || request.auth != null; }
  }
}
```

With the hardened rules you'd also make the owner a real Firebase Auth account and grant it the
`admin` custom claim via the Admin SDK (or a callable Cloud Function), and have the Owner Panel
sign in with Firebase Auth instead of the client-side password compare.

> The app loads the Firebase compat SDK from the CDN. When `USE_FIREBASE` is `false` it ignores
> Firebase entirely and runs fully offline in demo mode.

## 🎨 Customizing the public site

Edit the `DEFAULT_SETTINGS`, `DEFAULT_CATEGORIES`, `SKILLS`, and `PROJECTS` constants at the top
of the script, or — once logged in as Owner — use **Site Settings** in the Owner Panel for live
updates (name, title, bio, phone, email, location). The WhatsApp/email buttons are generated
automatically from the owner phone/email.

## 🛠 Tech notes

- Pure HTML/CSS/JS — no build step required.
- Glassmorphism + neumorphism components, CSS 3D transforms, `IntersectionObserver` reveals,
  canvas particle system, debounced scroll/mouse handlers.
- Responsive from 480px → 1280px+.
- Integrations: WhatsApp (`wa.me`) + email (`mailto:`), Google Fonts (Inter), Font Awesome.
