[README.md](https://github.com/user-attachments/files/31335132/README.md)
# Team Treasury

A single-file web app for tracking player account balances, fees, deposits, and payments for a youth sports team — with a private dashboard for the treasurer and a self-service portal for parents.

No build tools, no server code to maintain. It's one HTML file (`index.html`) that runs entirely in the browser, backed by a free Firebase (Firestore) database.

## What it does

**Treasurer side** (PIN-protected):
- **Dashboard** — outstanding balances, credit on file, this month's activity, and a quick list of who currently owes money
- **Roster** — add/edit players, track parent contact info (up to two parents per player), generate access codes, import players in bulk from a CSV or pasted spreadsheet, export the roster as a PDF
- **Transactions** — log charges and payments one at a time or in bulk across selected players, edit or delete any entry, filter by player/month, export to PDF
- **Statements** — a monthly summary across the whole roster (beginning balance, debits, credits, ending balance), with a per-player drill-down, PDF export, printing, and one-click email to a parent (with the PDF downloaded alongside for you to attach)
- **Settings** — team name, a custom message shown to all parents, treasurer PIN, full JSON backup and restore, and season archiving (freeze a season's records and start the next one clean)

**Parent side** (no account needed — just a name + access code):
- View their player's current balance and a monthly statement, with the same PDF export and print options

## How it's built

- **Frontend:** Plain HTML + React, loaded from CDN (`unpkg`/`cdnjs`) and compiled in-browser with Babel — no `npm install`, no build step. Just edit `index.html` and re-upload it.
- **Database:** [Firebase Firestore](https://firebase.google.com/), using anonymous authentication (no login screen — it happens silently) and open-but-authenticated security rules.
- **Hosting:** [Netlify](https://netlify.com), auto-deploying from this GitHub repo on every push.
- **PDF export:** [jsPDF](https://github.com/parallax/jsPDF) + [jspdf-autotable](https://github.com/simonbengtsson/jsPDF-AutoTable), loaded from CDN.

## Updating the app

1. Make changes to `index.html` (or ask Claude to make them for you).
2. Upload the new version to this repo — **the file must stay named `index.html`**, since that's what Netlify serves as the site's homepage.
3. Netlify redeploys automatically within about a minute of the push.

## Firebase setup (only needed if starting over from scratch)

This repo's `index.html` already has a live Firebase project's config embedded near the top of the file. If you ever need to point it at a *new* Firebase project instead:

1. Create a free project at [console.firebase.google.com](https://console.firebase.google.com).
2. Enable **Firestore Database** (production mode).
3. Enable **Anonymous** sign-in under Authentication → Sign-in method.
4. Set Firestore's rules to:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
5. Register a web app in the project, copy the six config values it gives you, and paste them into the `firebaseConfig` object near the top of `index.html`.

## About the "exposed API key" warning

GitHub's secret scanner may flag the Firebase `apiKey` in this file. **This is a known false positive** — Firebase's client config is designed to be public and is safe to commit. It only identifies which project to talk to; it doesn't grant access on its own. The actual access control is the Firestore security rule above (`request.auth != null`), not the key. See [Firebase's own documentation](https://firebase.google.com/docs/projects/api-keys) for more.

What would be a real problem is a **Firebase Admin SDK service account file** — this project doesn't use one anywhere, and one should never be added to this repo.

## A note on security

This app trades strict security for simplicity, which fits its purpose — internal team bookkeeping, not banking:

- The treasurer PIN and parent access codes are a soft gate, not enterprise-grade authentication.
- Firestore rules require *a* valid (anonymous) session, but don't distinguish "treasurer" from "parent" at the database level — the PIN and access-code checks happen in the app itself.
- Don't enter sensitive data this isn't designed for — no bank account numbers, SSNs, or similar.

## Backups

Download a full backup (Settings → Backup & restore) periodically, and after any large batch of changes. It's a complete, restorable snapshot of every player and transaction — worth keeping a copy somewhere outside this one browser/device (e.g. emailed to yourself or saved to a shared drive).
