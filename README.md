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
- **Owner Panel** — Dashboard stats, Site Settings (live edits to site text, **owner profile image**, **site logo / favicon**, and **social profile links**), Order
  Management (search/filter/status/update/delete/notify), User Management
  (block/unblock/delete/email/export CSV), **Products & Pricing** (owner-managed services with
  USD prices, optional image/video sample upload OR auto "use custom template" visual per
  service), Owner Credentials.
- **Public Services section** — every owner-defined product is shown on the landing page with its
  price, converted live across multiple currencies (default **USD**), and a "Order Now" button.
- **Real‑time sync** — Owner changes (profile, categories, order status, blocking) instantly
  reflect on the public site and user panel.

## 🚀 Running it

**Demo mode (no setup, works immediately):** just open `index.html` in a browser. All data is
stored in `localStorage` and synced in real‑time within the same browser.

**Owner login (shared screen):** the owner logs in through the *same* "Email or Username"
login field as regular users — there is no separate owner button. Use the credentials stored in
`settings/owner` (default **`sarib` / `sarib123`**). Signing in with the Google account
**`saribshareefbacha@gmail.com`** also routes straight to the Owner Panel.

**Firebase mode (production):** open `index.html`, scroll to the `CONFIG` block near the top of
the `<script>`, paste your Firebase web config into `FIREBASE_CONFIG`, and set `USE_FIREBASE = true`.
Also create a **Firestore** database with these collections/documents:

```
settings/owner        → { name, cast, exp, desc, phone, email, location, adminUser, adminPass, avatar, logo, socials[], createdAt, updatedAt }
users/{userId}        → { name, username, email, phone, status, avatar, createdAt, updatedAt }
orders/{orderId}      → { orderId, customer, username, email, phone, category, description, attachments[], status, date, userId, createdAt, updatedAt }
categories/{catId}    → { name, price, desc, kw, useTemplate, media, mediaType, createdAt }
                        (price is stored in USD; media is a data-URL when uploaded via the panel)
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

Edit the `DEFAULT_SETTINGS`, `DEFAULT_PRODUCTS` (owner services + USD prices), `CURRENCIES`
(exchange rates/symbols), `SKILLS`, and `PROJECTS` constants at the top of the script, or — once
logged in as Owner — use **Site Settings** for live text edits and **Products & Pricing** to add
/ edit services (name, USD price, description, image/video sample upload, or "use custom
template"), **Site Settings** to upload the **owner profile image** and **site logo / favicon** (shown in the navbar + browser tab) and manage **social profile links** (rendered at the bottom of the landing page), and **Owner Credentials** to change the owner username/password. The WhatsApp/email
buttons are generated automatically from the owner phone/email.

The app is resilient: if Firestore reads are blocked by security rules it falls back to the
default data in memory and shows a red toast prompting you to fix the rules (see above), so the
public site still renders.

## 🛠 Tech notes

- Pure HTML/CSS/JS — no build step required.
- Glassmorphism + neumorphism components, CSS 3D transforms, `IntersectionObserver` reveals,
  canvas particle system, debounced scroll/mouse handlers.
- Responsive from 480px → 1280px+.
- Integrations: WhatsApp (`wa.me`) + email (`mailto:`), Google Fonts (Inter), Font Awesome.
