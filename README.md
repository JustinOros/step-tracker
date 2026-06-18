# Step Tracker

A mobile-friendly step tracking app for friend groups. Compare daily, weekly, and monthly steps with people you know.

Built with vanilla JavaScript, Firebase, and hosted free on GitHub Pages. No native app required — steps sync automatically from Apple Health or Android Health via a free bridge app.

![License](https://img.shields.io/badge/license-MIT-blue)
![Firebase](https://img.shields.io/badge/backend-Firebase-orange)
![GitHub Pages](https://img.shields.io/badge/hosting-GitHub%20Pages-green)

---

## Features

- **Passwordless sign-in** — magic link sent to email, no password needed
- **Admin approval** — new registrations require administrator approval before access is granted
- **Auto step sync** — bridge apps on iPhone and Android POST steps automatically via webhook
- **Manual entry** — log steps manually if preferred
- **Friend system** — add friends by email, approve or deny requests
- **Step comparisons** — view steps by Today, Week, Month, Year, or All Time
- **Profiles** — display name and avatar URL per user
- **Mobile-first** — designed for use in a mobile browser, works as a PWA

---

## How It Works

```
iPhone/Android Health App
        ↓
  Bridge App (free)
        ↓
  Webhook URL (Firebase Cloud Function)
        ↓
  Firestore Database
        ↓
  Step Tracker Web App (GitHub Pages)
```

---

## Tech Stack

| Layer | Technology | Cost |
|---|---|---|
| Frontend | Vanilla JS, HTML, CSS | Free |
| Hosting | GitHub Pages | Free |
| Auth | Firebase Authentication (magic link) | Free |
| Database | Firebase Firestore | Free tier |
| Backend | Firebase Cloud Functions | Free tier |

---

## Setup Your Own Instance

### Prerequisites

- A [Firebase](https://firebase.google.com) account
- A [GitHub](https://github.com) account
- [Node.js](https://nodejs.org) installed (LTS version)

---

### 1. Create a Firebase Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → name it → disable Analytics → **Create project**

---

### 2. Enable Passwordless Email Auth

1. Firebase Console → **Authentication** → **Get started**
2. Click **Email/Password** → enable **Email link (passwordless sign-in)** → Save
3. **Settings → Authorized domains** → add `yourusername.github.io`

---

### 3. Create Firestore Database

1. Firebase Console → **Firestore Database** → **Create database**
2. Choose **Start in test mode** → pick a region → **Done**
3. Click the **Rules** tab and paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /profiles/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    match /friendRequests/{reqId} {
      allow read: if request.auth != null &&
        (resource.data.fromUid == request.auth.uid ||
         resource.data.toUid == request.auth.uid);
      allow create: if request.auth != null &&
        request.resource.data.fromUid == request.auth.uid;
      allow update: if request.auth != null &&
        resource.data.toUid == request.auth.uid;
    }
  }
}
```

Click **Publish**.

---

### 4. Deploy Cloud Functions

```bash
npm install -g firebase-tools
firebase login
mkdir step-tracker-fn && cd step-tracker-fn
firebase init functions
# Choose: JavaScript, No ESLint, Yes to install dependencies
```

Copy the contents of `functions/index.js` from this repo into your `functions/index.js`.

Install nodemailer:
```bash
cd functions && npm install nodemailer
```

Set your secrets (Firebase will prompt for each value):
```bash
firebase functions:secrets:set ADMIN_EMAIL
firebase functions:secrets:set ADMIN_SECRET
firebase functions:secrets:set APP_URL
firebase functions:secrets:set GMAIL_USER
firebase functions:secrets:set GMAIL_PASS
```

> **GMAIL_PASS** must be a Gmail App Password, not your regular password.
> Generate one at [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)

Deploy:
```bash
cd .. && firebase deploy --only functions
```

Copy the function URLs shown after deployment.

---

### 5. Configure the App

Copy `config.example.js` to `config.js` and fill in your values:

```javascript
window.STEPMATES_CONFIG = {
  firebase: {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.firebasestorage.app",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  },
  webhookBase: "https://YOUR_WEBHOOK_URL.a.run.app",
  adminNotifyUrl: "https://us-central1-YOUR_PROJECT.cloudfunctions.net/adminNotify",
  adminSecret: "YOUR_ADMIN_SECRET",
  appUrl: "https://yourusername.github.io/step-tracker"
};
```

Get your Firebase config from: Firebase Console → gear icon → Project Settings → Your Apps → Add app → Web.

---

### 6. Publish to GitHub Pages

1. Create a GitHub repo named `step-tracker`
2. Rename `stepmates.html` → `index.html`
3. Upload `index.html`, `config.js`, and `.gitignore`
4. Go to repo **Settings → Pages → Deploy from branch → main / root → Save**
5. Your app will be live at `https://yourusername.github.io/step-tracker`

---

## Bridge App Setup

Share these instructions with your friends after they register.

### iPhone — Health Auto Export
- Download: [App Store](https://apps.apple.com/us/app/health-auto-export-json-csv/id1115567069)
- Automations → New Automation → Webhook → paste your personal webhook URL
- Data types: **Step Count**
- Schedule: Daily

### Android — HC Webhook
- Download: [GitHub Releases](https://github.com/mcnaveen/health-connect-webhook/releases) (free, open source)
- Add Webhook → paste your personal webhook URL
- Enable **Steps** → set sync interval

Each user gets their own unique webhook URL shown in the app after sign-in.

---

## Admin Approval Flow

1. New user enters email → receives magic sign-in link
2. After clicking the link, their account is created with `approved: false`
3. Administrator receives an email with **Approve** and **Deny** buttons
4. User sees a "Waiting for approval" screen until approved
5. Once approved, they proceed to onboarding

---

## Free Tier Limits

| Service | Free Allowance | Expected Usage |
|---|---|---|
| Firebase Auth | Unlimited | Fine |
| Firestore reads | 50,000/day | Fine for small groups |
| Firestore writes | 20,000/day | Fine |
| Cloud Functions | 2M calls/month | Fine (a few calls/day) |
| GitHub Pages | Unlimited | Fine |

**Expected monthly cost: $0**

---

## Roadmap

- [ ] Step contests with start/end dates
- [ ] Weekly recap emails
- [ ] Streak tracking
- [ ] Push notifications when a friend passes you
- [ ] Leaderboard with rank change indicators

---

## License

MIT — do whatever you want with it.
