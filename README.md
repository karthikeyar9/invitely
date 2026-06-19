# Invitely ✉️

A clean, modern invitation site. Fill in your event details, pick a theme,
and get a single link you can text or email to friends.

It runs in one of two modes, automatically:

- **Link-only mode** (default, zero setup): the whole invitation is encoded in
  the share link. No server, no database. Guests RSVP by tapping a button that
  opens a pre-filled email or text to you.
- **Firebase mode** (5-minute setup): events are saved to Firestore, links are
  short (`#e=abc123`), and guests RSVP **on the page** with their name. Everyone
  sees a live tally ("🎉 12 going") and you don't have to count text messages.

## Run it locally

```bash
cd invitely
python3 -m http.server 8642
# open http://localhost:8642
```

## Enable Firebase mode (RSVP tracking)

1. Go to https://console.firebase.google.com → **Add project** (any name,
   Analytics not needed — free Spark plan is plenty).
2. In the project: **Build → Firestore Database → Create database** →
   Start in **production mode** → pick a region near you.
3. Open the **Rules** tab and replace the rules with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /events/{eventId} {
         allow read, create: if true;
         allow update, delete: if false;
         match /rsvps/{rsvpId} {
           allow read: if true;
           allow create: if request.resource.data.name is string
                         && request.resource.data.name.size() > 0
                         && request.resource.data.name.size() <= 60
                         && request.resource.data.going is bool;
           allow update, delete: if false;
         }
       }
     }
   }
   ```

   (Anyone can create an event or RSVP, nobody can edit or delete — invites
   are immutable once sent, like a paper invitation.)

4. Get your web app config (this is the step people get lost on):
   1. Click the **gear icon ⚙** next to "Project Overview" (top-left) →
      **Project settings**.
   2. Stay on the **General** tab and scroll all the way down to the
      **Your apps** section. It will say *"There are no apps in your project"*
      — that's expected.
   3. Click the **`</>`** icon (the Web one, next to the iOS/Android icons).
   4. Give the app any nickname (e.g. "invitely"), **leave "Firebase Hosting"
      unchecked** (we set that up separately below), and click **Register app**.
   5. You'll see a code snippet containing `const firebaseConfig = { apiKey:
      "...", authDomain: "...", ... }`. Copy just that `{ ... }` object —
      ignore the `npm install` and `import` lines around it.
   6. If you clicked past it: Project settings → General → Your apps →
      select the app → **SDK setup and configuration** → choose **Config**.
5. Paste it into `firebase-config.js` in this folder so it reads
   `window.FIREBASE_CONFIG = { apiKey: "...", ... };` (see the example there).

That's it — the site detects the config and switches modes. The config values
are safe to publish; access is controlled by the rules above.

## Put it online (so friends can open your links)

**Recommended: Firebase Hosting** — free, fast, HTTPS included, and it lives in
the same console as your database. A `firebase.json` is already in this folder,
so from the `invitely` directory just run:

```bash
npm install -g firebase-tools
firebase login
firebase use --add        # pick your project from the list, alias "default"
firebase deploy --only hosting
```

The deploy prints your live URL (`https://<your-project>.web.app`). Repeat
`firebase deploy --only hosting` whenever you change a file.

Any other static host also works if you prefer:

- **Netlify Drop**: drag the `invitely` folder onto https://app.netlify.com/drop
- **GitHub Pages** or **Vercel**.

Create invitations from the live URL (not localhost) so links work for everyone.

## Features

- 12 designed themes: Confetti, Midnight, Garden, Ocean, Sunset, Minimal,
  Floral, Blush, Gatsby, Sage, Fiesta, Picnic — with script typography,
  stationery-style frames, and patterned backgrounds
- Upload your own photo or design as the invitation background (auto-resized
  and compressed in the browser; best with Firebase mode, since photo data is
  too large for nice link-only links)
- Live preview while you type
- In-page RSVP with live "who's coming" tally (Firebase mode) or
  RSVP-by-email/text (link-only mode)
- Guests can change their answer; the latest response per name wins
- "Add to calendar" (Google Calendar) and "Open in Maps" links
- "Share via text" button to send the invite by SMS
- Works on phones; no accounts required for you or your guests
