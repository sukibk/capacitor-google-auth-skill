# Implementation Reference

## Table of Contents
- [Google Cloud Console Setup](#google-cloud-console-setup)
- [Supabase Dashboard Setup](#supabase-dashboard-setup)
- [Environment Variables](#environment-variables)
- [iOS Info.plist Config](#ios-infoplist-config)
- [Plugin Initialization Code](#plugin-initialization-code)
- [Platform-Aware Sign-In Code](#platform-aware-sign-in-code)
- [Sync Commands](#sync-commands)

## Google Cloud Console Setup

1. Go to Google Cloud Console → APIs & Credentials
2. Create TWO OAuth 2.0 Client IDs:
   - **Web application** — likely already exists if Google OAuth works on web
   - **iOS** — create new with Application type "iOS". Set Bundle ID to match the Xcode project. App Store ID and Team ID can be left blank during development.

## Supabase Dashboard Setup

If the project uses Supabase as the auth backend:

1. Authentication → Providers → Google
2. **Client IDs field**: add BOTH client IDs comma-separated:
   ```
   WEB_CLIENT_ID.apps.googleusercontent.com,IOS_CLIENT_ID.apps.googleusercontent.com
   ```
3. **Enable "Skip nonce checks"** — required for iOS because you don't have access to the nonce the native SDK used to issue the ID token
4. Authentication → URL Configuration → Redirect URLs: ensure web callback URLs are listed

For other backends (Firebase, custom): register the iOS client ID in whatever mechanism your backend uses to validate Google ID tokens.

## Environment Variables

Create two env vars for the client IDs. Adapt the prefix to the project's framework:

```bash
# Next.js
NEXT_PUBLIC_GOOGLE_WEB_CLIENT_ID=xxx.apps.googleusercontent.com
NEXT_PUBLIC_GOOGLE_IOS_CLIENT_ID=xxx.apps.googleusercontent.com

# Vite / Vue / React (Vite)
VITE_GOOGLE_WEB_CLIENT_ID=xxx.apps.googleusercontent.com
VITE_GOOGLE_IOS_CLIENT_ID=xxx.apps.googleusercontent.com

# Angular
NG_APP_GOOGLE_WEB_CLIENT_ID=xxx.apps.googleusercontent.com
NG_APP_GOOGLE_IOS_CLIENT_ID=xxx.apps.googleusercontent.com
```

Set in `.env.local` and your deployment platform (Vercel, Netlify, etc.).

## iOS Info.plist Config

Add the **reversed iOS client ID** as a URL scheme. The Google Sign-In SDK requires this to handle the auth callback:

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>com.googleusercontent.apps.YOUR_IOS_CLIENT_ID</string>
    </array>
  </dict>
</array>
```

Replace `YOUR_IOS_CLIENT_ID` with the actual iOS client ID (the part before `.apps.googleusercontent.com`).

## Plugin Initialization Code

Run once at app startup (e.g., in a Capacitor init file, root layout, or app component):

```typescript
const { SocialLogin } = await import('@capgo/capacitor-social-login')
await SocialLogin.initialize({
  google: {
    iOSClientId: process.env.NEXT_PUBLIC_GOOGLE_IOS_CLIENT_ID,  // adapt env var name
    webClientId: process.env.NEXT_PUBLIC_GOOGLE_WEB_CLIENT_ID,   // adapt env var name
  },
})
```

## Platform-Aware Sign-In Code

The core pattern — detect native vs web at runtime, then branch:

```typescript
// Detect if running in a native Capacitor shell
const cap = (window as any).Capacitor
const isNative = typeof cap?.isNativePlatform === 'function' && cap.isNativePlatform()

if (isNative) {
  // Native path: use the Google Sign-In SDK directly
  const { SocialLogin } = await import('@capgo/capacitor-social-login')
  const res = await SocialLogin.login({
    provider: 'google',
    options: { scopes: ['email', 'profile'] },
  })

  // Type narrowing — idToken only exists on the Online response variant
  const result = res.result
  if (!result || !('idToken' in result) || !result.idToken) {
    throw new Error('No ID token returned from Google Sign-In')
  }

  // Exchange the ID token with your backend
  // Supabase example:
  const { data, error } = await supabase.auth.signInWithIdToken({
    provider: 'google',
    token: result.idToken,
  })

  // Firebase example:
  // const credential = GoogleAuthProvider.credential(result.idToken)
  // await signInWithCredential(auth, credential)

} else {
  // Web path: standard OAuth redirect
  // Supabase example:
  await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: { redirectTo: `${window.location.origin}/auth/callback` },
  })
}
```

## Sync Commands

After any native config changes (Info.plist, plugin install, capacitor.config changes):

```bash
npx cap sync ios
npx cap sync android  # if targeting Android
```

Then rebuild in Xcode (iOS) or Android Studio.
