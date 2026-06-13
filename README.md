# Daily Planner

A personal daily planner web app with a weekly calendar view, priorities sidebar, recurring events, and cross-device sync via Firebase.

**Live app:** https://daily-planner-eaeed.web.app

---

## How the app works

### Calendar
- Displays a 7-day week view with 30-minute time slots
- Click any empty slot to create an event
- Click an existing event to edit or delete it
- Drag events to move them — the event's start time snaps to the cell you drop on
- Drag the bottom edge of an event to resize it

### Events
- Set a title, date, start/end time, location, notes, color, and reminder
- Mark an event as **All day** to pin it to the day header as a chip
- Set a **Repeat** pattern (daily, weekly, monthly) with an end date
- When editing a recurring event you can choose to update **this occurrence only** or **all occurrences**

### Priorities sidebar
- Add weekly priorities with optional progress targets (e.g. "Run 5 times")
- Track completion with checkboxes or a progress counter
- Priorities are scoped per week

### Themes
- Choose from Professional, Dark, Pink, or Blue via the 🎨 Theme button

---

## Data storage

The app has three independent storage layers. They do not automatically sync with each other — only Firebase syncs across devices.

### 1. Browser localStorage (always on)
Data is saved automatically in your browser's local storage every time you make a change. This requires no setup and works offline.

**Limitations:**
- Tied to one browser on one device
- Cleared if you wipe browser data or use a private/incognito window
- Not accessible from your phone or another computer

### 2. Local folder (optional)
Click **📁 Select Folder** to link a folder on your computer. The app saves a `planner-data.json` file there on every change.

**Use this when you want to:**
- Back up your data to a real file you can copy or restore
- Access your data from any browser on the same computer by selecting the same folder
- Keep a copy independent of browser storage

**Limitations:**
- Still limited to one computer — other devices cannot access the file
- Requires selecting the folder once per browser session (browser security restriction)

### 3. Firebase cloud sync (recommended for multi-device)
Click **☁️ Sign In** and sign in with your Google account. Your data is stored in Firebase Firestore and syncs in real time across all your devices.

**Use this when you want to:**
- Access your planner on both your laptop and phone
- Have changes on one device appear instantly on another
- Keep your data safe even if you clear your browser or switch devices

**How it works:**
- Each Google account gets its own isolated data store — no one else can see your events
- When you sign in on a new device, the app compares local and cloud data and shows a sync review if there are differences
- Every save automatically writes to both localStorage and Firebase
- Signing out stops syncing; your local data remains

---

## Security

### Authentication
- Sign-in uses Google OAuth via Firebase Authentication
- Only signed-in users can read or write data
- Each user can only access their own data — enforced by Firestore security rules

### Firestore security rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null
                   && request.auth.uid == userId
                   && request.resource.size < 500000;
    }
  }
}
```

- **Authentication required** — anonymous access is blocked
- **User isolation** — users can only read/write their own path (`users/{their uid}/...`)
- **Write size limit** — each write is capped at 500KB to prevent quota abuse

### XSS protection
- All user-supplied text rendered into the DOM uses `textContent` or HTML escaping — no raw HTML injection from user data

### Firebase API key
The Firebase config (including API key) is intentionally public in the client code. This is standard Firebase practice — the API key is a project identifier, not a secret. Security is enforced entirely by Firestore rules, not by hiding the key.

---

## Deployment

The app is a single HTML file with no build step.

### Firebase Hosting (primary)
```
firebase deploy --only hosting
```

### GitHub Pages (backup)
```
git push origin main
```
GitHub Pages auto-deploys from the `main` branch to:
https://jennyyecao-coder.github.io/daily-planner

> Note: Firebase sign-in does not work on the GitHub Pages URL due to browser security restrictions (COOP headers). Use the Firebase Hosting URL for sync features.

---

## Firebase free tier limits (Spark plan)

| Resource | Free limit |
|---|---|
| Firestore reads | 50,000 / day |
| Firestore writes | 20,000 / day |
| Firestore storage | 1 GB |
| Hosting bandwidth | 10 GB / month |

Well within limits for personal use or a small group.
