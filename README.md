# StepMates — Walk Together

A private step tracking web app that syncs steps from your phone and lets you compete with friends on a leaderboard.

**Live app:** https://justinoros.github.io/step-tracker

---

## Features

- Passwordless email sign-in
- Admin approval for new accounts
- Real-time step sync from Apple Health (iPhone) or Health Connect (Android)
- Friend leaderboard with Today / Week / Month / Year / All Time tabs
- Friend requests by email
- Nicknames for friends (private, only you see them)
- Friend profile pages with average daily steps and last sync date
- Manual step entry
- Daily step goal with progress bar
- Pull-to-refresh
- Mobile-first, installs to home screen as a PWA

---

## How to Join

1. Go to https://justinoros.github.io/step-tracker
2. Enter your email address and click **Continue with Email**
3. Check your email for a sign-in link
4. Choose a display name and complete setup
5. Wait for administrator approval — you'll receive an email with a sign-in link when approved

---

## Syncing Steps

### iPhone
1. Download **Health Auto Export** from the App Store (free tier works)
2. Sign in to StepMates → hamburger menu → **Setup & Sync** → copy your Webhook URL
3. In Health Auto Export → Automations → New Automation → Webhook → paste URL
4. Select **Steps** → set Date Range to **Default** → set schedule to **Daily**

### Android
1. Download the **StepMates Sync** Android app:
   [Download StepMates-Sync-Android.apk](https://github.com/JustinOros/step-tracker/releases/download/v1.1/StepMates-Sync-Android.apk)
2. Install it (allow unknown sources when prompted)
3. Sign in to StepMates in your browser → hamburger menu → **Setup & Sync** → copy your Webhook URL
4. Open the StepMates Sync app → paste your Webhook URL → tap **Sync My Steps**
5. Grant Health Connect permission when prompted
6. Your steps will sync — open the app any time to sync again

> **Note:** Make sure your fitness app (Google Fit, Samsung Health, etc.) is syncing to Health Connect on your Android device.

---

## Tech Stack

- **Frontend:** Vanilla HTML/CSS/JS, hosted on GitHub Pages
- **Backend:** Firebase Firestore + Cloud Functions (Node.js)
- **Auth:** Firebase Authentication (passwordless email link)
- **Android companion:** Kotlin + Health Connect API

---

## Project Structure

```
step-tracker/
  index.html        — Main web app
  config.js         — Firebase config (safe to commit)
  README.md         — This file
```

---

## For Developers

This is a private app — new registrations require admin approval. If you want to run your own instance:

1. Create a Firebase project
2. Enable Firestore and Authentication (Email link sign-in)
3. Copy `config.js` and fill in your Firebase credentials
4. Deploy the Cloud Functions from the `functions/` folder
5. Host `index.html` and `config.js` on GitHub Pages or any static host

### Firestore Collections

| Collection | Purpose |
|---|---|
| `users` | Email, webhook token, approval status |
| `profiles` | Display name, avatar URL |
| `steps` | Step data by date |
| `friends/{uid}/list` | Friend relationships (subcollection) |
| `pendingFriendRequests` | Pending invites for unregistered users |
| `rates` | Rate limiting for magic link emails |

### Cloud Functions

| Function | Purpose |
|---|---|
| `webhook` | Receives step data from bridge apps |
| `sendMagicLink` | Sends branded sign-in email |
| `adminNotify` | Notifies admin of new registration |
| `approveUser` | Approves/denies user, sends approval email |
| `friendRequest` | Sends friend request email, creates friend docs |
