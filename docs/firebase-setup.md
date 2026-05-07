# Firebase Authentication Setup

## Overview

This guide sets up Firebase for 2-user authentication on your VC platform.

**Setup time: ~15 minutes**

## Step 1: Create Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click "Add Project"
3. Enter project name: `worklife-vc-hub`
4. Disable Google Analytics (not needed)
5. Click "Create Project"

## Step 2: Enable Email/Password Authentication

1. In Firebase Console, go to **Authentication**
2. Click **"Get Started"**
3. Select **"Email/Password"**
4. Enable both:
   - Email/Password
   - Email link sign-in
5. Click **"Save"**

## Step 3: Add Users (2 only)

1. Go to **Users** tab
2. Click **"Add user"**
3. Add your email + strong password
4. Repeat for second user

**Example:**
- User 1: `your-email@domain.com` / `SecurePassword123!`
- User 2: `partner-email@domain.com` / `SecurePassword456!`

## Step 4: Get Firebase Configuration

1. Click **Project Settings** (gear icon)
2. Go to **"Your apps"** section
3. Click **"Web"** app
4. Copy the config object:

```javascript
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "worklife-vc-hub.firebaseapp.com",
  projectId: "worklife-vc-hub",
  storageBucket: "worklife-vc-hub.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123...:web:abc..."
};
```

## Step 5: Get Service Account Key (for backend)

1. Go to **Project Settings** → **Service Accounts**
2. Click **"Generate new private key"**
3. Save the JSON file safely
4. You'll use these values in `.env.local`:
   - `FIREBASE_PRIVATE_KEY` → `private_key` from JSON
   - `FIREBASE_CLIENT_EMAIL` → `client_email` from JSON

## Step 6: Update Environment Variables

In `.env.local`, add:

```env
NEXT_PUBLIC_FIREBASE_API_KEY=AIza...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=worklife-vc-hub.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=worklife-vc-hub
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=worklife-vc-hub.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123456789
NEXT_PUBLIC_FIREBASE_APP_ID=1:123...:web:abc...
FIREBASE_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----\n...
FIREBASE_CLIENT_EMAIL=firebase-adminsdk-abc...@worklife-vc-hub.iam.gserviceaccount.com
```

## Step 7: Set Firebase Security Rules

1. Go to **Firestore Database** (or Realtime Database)
2. Click **"Rules"** tab
3. Replace with:

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null",
    "ventures": {
      "$uid": {
        ".read": "auth.uid == $uid || auth.uid in root.child('admins').child($uid).val()",
        ".write": "auth.uid == $uid"
      }
    }
  }
}
```

4. Click **"Publish"**

## Step 8: Test Authentication

Run locally:

```bash
npm run dev
```

Go to `http://localhost:3000/auth/login`

Test login with one of your 2 users.

## Troubleshooting

**"Authentication not configured"**
- Check `.env.local` has all Firebase keys
- Restart dev server: `npm run dev`

**"User not found"**
- Make sure you added the user in Firebase Console → Users
- Check email spelling matches exactly

**"Too many login attempts"**
- Firebase blocks after 5 failed attempts
- Wait 5 minutes or reset in Console

## Next Steps

→ [Airtable Integration](./airtable-integration.md)
