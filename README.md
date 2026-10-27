<p align="center">
  <a href="https://abrova.ir">
    <img src="https://abrova.ir/icon.svg" alt="Abrova" width="120" />
  </a>
</p>

<h3 align="center">Abrova Auth Web SDK</h3>

<p align="center">
  Hosted sign-in for your web app's users. The SDK keeps the session fresh and gives the other Abrova SDKs a user token.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-5+-3178C6?logo=typescript&logoColor=white" alt="TypeScript 5+" />
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="MIT License" />
</p>

---

## Features

- **Six sign-in methods**: anonymous, Apple, Google, e-mail + password, magic link and Telegram
- **Upgrade guests**: `linkApple`, `linkGoogle` and `linkPassword` keep the user id
- **Fresh tokens**: the access token is renewed before it expires; concurrent calls share one renewal
- **One user for every Abrova product**: hand the token to Abrova Sync, Chat, Push and A/B Testing
- **Client-safe**: uses the public API key only
- **Any framework**: plain TypeScript for React, Vue, Svelte, Next.js client code, or none
- **Memory by default**: nothing is stored unless you opt in
- **Zero dependencies**, TypeScript declarations included

## Installation

The package is published on [Abrova Center](https://center.abrova.ir). Add this line to your project's `.npmrc`:

```ini
@abrova:registry=https://center.abrova.ir/api/packages/abrova/npm/
```

Then install:

```bash
npm install @abrova/auth-web
# or
yarn add @abrova/auth-web
```

Requires a browser with `fetch` and `crypto.getRandomValues` (every current browser). Abrova Auth has no server SDK.

## Quick start

Turn on Abrova Auth and the sign-in methods you want in [Abrova Console](https://console.abrova.ir), then:

```typescript
import { createAbrovaAuth } from '@abrova/auth-web';

const auth = createAbrovaAuth({ apiKey: 'ab_live_YOUR_PUBLIC_KEY' });

auth.onAuthStateChanged((userId) => {
  console.log(userId ? `signed in as ${userId}` : 'signed out');
});

await auth.signInAnonymously();
```

Show only the methods your project has turned on:

```typescript
const { methods } = await auth.getMethods(); // e.g. ['anonymous', 'google', 'password']
```

## Use with other Abrova products

The signed-in user's access token is the standard Abrova user token, so the other Abrova SDKs act as that user and
Sync security rules see it as `auth.uid`. No backend needed.

- **Abrova Sync, Push and A/B Testing** accept a provider that returns `null` while nobody is signed in: pass
  `() => auth.getToken()`.
- **Abrova Chat** needs a provider that always returns a token: pass `auth.tokenProvider`, which rejects with
  `UNAUTHENTICATED` while signed out, and create the Chat client after sign-in.

```typescript
import { AbrovaSync } from '@abrova/sync-web';
import AbrovaPush from '@abrova/push-web';
import { AbrovaAbTesting } from 'abrova-ab-testing-nextjs';
import { AbrovaChatClient } from '@abrova/web-chat';

// Abrova Sync, Push and A/B Testing: null while signed out
const sync = new AbrovaSync({ apiKey: 'ab_live_YOUR_PUBLIC_KEY', tokenProvider: () => auth.getToken() });
const push = new AbrovaPush({ apiKey: 'ab_live_YOUR_PUBLIC_KEY', tokenProvider: () => auth.getToken() });
await AbrovaAbTesting.init({ apiKey: 'ab_live_YOUR_PUBLIC_KEY', tokenProvider: () => auth.getToken() });

// Abrova Chat: auth.tokenProvider rejects with UNAUTHENTICATED while signed out
const chat = new AbrovaChatClient({
  apiKey: 'ab_live_YOUR_PUBLIC_KEY',
  userId: auth.currentUserId!,
  tokenProvider: auth.tokenProvider,
});
```

After a sign-in, a sign-out or a switch to another user, ask the other SDKs to fetch a new token (for example
`sync.refreshUserToken()`), ideally from `onAuthStateChanged`. More:
[Use with other Abrova products](https://abrova.ir/cloud/abrova-auth/docs/other-products).

## Sign-in methods

```typescript
// E-mail + password
await auth.signUpWithEmail({ email, password, redirectUrl: 'https://app.example.com/verify' });
await auth.signInWithEmail({ email, password });
await auth.sendPasswordReset({ email, redirectUrl: 'https://app.example.com/reset' });
await auth.resetPassword({ token, password });          // token from the link's ?token=
await auth.verifyEmail({ token });

// Magic link
await auth.sendMagicLink({ email, redirectUrl: 'https://app.example.com/magic' });
await auth.signInWithMagicLink({ token: new URL(location.href).searchParams.get('token')! });

// Telegram
const { sent, botUrl } = await auth.requestTelegramCode({ phone: '+37491234567' });
await auth.verifyTelegramCode({ phone: '+37491234567', code: '123456' });
```

`redirectUrl` must start with one of the redirect URLs allowed in your project's Abrova Auth settings.

### Apple and Google

Run the provider's own sign-in with a nonce from the SDK, then hand its ID token to Abrova:

```typescript
const { rawNonce, hashedNonce } = await auth.createNonce();
// Give hashedNonce to Google Identity Services / Sign in with Apple JS as the `nonce`.
const idToken = /* the ID token the provider returned */;
await auth.signInWithGoogle({ idToken, rawNonce });
await auth.signInWithApple({ idToken, rawNonce });
```

### Upgrading an anonymous user

Linking keeps the user id, so the user's data stays theirs:

```typescript
await auth.linkGoogle({ idToken, rawNonce });
await auth.linkApple({ idToken, rawNonce });
await auth.linkPassword({ email, password });
```

If that Apple or Google account already belongs to another user, the call fails with `PROVIDER_ALREADY_LINKED`.

## Account

```typescript
const me = await auth.getMe();     // { id, anonymous, emailVerified, createdAt, providers }
await auth.signOut();              // this device
await auth.signOutEverywhere();    // every device
await auth.deleteAccount();        // deletes the user, then signs out
```

## Errors

Every failure is a `AbrovaAuthError` with a `code`:

```typescript
import { isAbrovaAuthError } from '@abrova/auth-web';

try {
  await auth.signInWithEmail({ email, password });
} catch (e) {
  if (isAbrovaAuthError(e) && e.code === 'INVALID_CREDENTIALS') showError('Wrong e-mail or password');
  if (isAbrovaAuthError(e) && e.code === 'RATE_LIMITED') retryIn(e.retryAfterSec);
}
```

Common codes: `INVALID_CREDENTIALS`, `EMAIL_TAKEN`, `WEAK_PASSWORD`, `EMAIL_NOT_VERIFIED`, `PROVIDER_ALREADY_LINKED`,
`METHOD_DISABLED`, `RATE_LIMITED`, `MAU_LIMIT`, `PRODUCT_DISABLED`, `NETWORK_ERROR`.

## Sessions

- The access token lives in memory and is refreshed 2 minutes before it expires. Calls made at the same time share
  one refresh.
- A revoked or expired session signs the user out (`onAuthStateChanged(null)`). A network error does not.

### Staying signed in after a reload

By default nothing is stored, so the user signs in again after a page reload. To keep the session, opt in to
localStorage:

```typescript
import { createAbrovaAuth, localStorageStorage } from '@abrova/auth-web';

const auth = createAbrovaAuth({ apiKey: 'ab_live_YOUR_PUBLIC_KEY', storage: localStorageStorage() });
```

> **Warning:** any script running on your page can read localStorage. An XSS bug or a compromised dependency could
> steal the refresh token and with it the user's session. Use this only with a strict Content Security Policy and no
> untrusted scripts. The SDK never sets cookies.

You can also pass your own storage (`{ get, set, remove }`, all async).

## Example app

[`example/`](example) is a small Vite page with anonymous sign-in, e-mail sign-up and sign-in, sign-out, account
deletion and the current user id. See [example/README.md](example/README.md) to run it.

## Documentation

- [Abrova Auth Docs](https://abrova.ir/cloud/abrova-auth/docs)
- [Web SDK reference](https://abrova.ir/cloud/abrova-auth/docs/sdks-web)
- [Use with other Abrova products](https://abrova.ir/cloud/abrova-auth/docs/other-products)
- [Abrova Console](https://console.abrova.ir)

## Support

- **Documentation**: [Abrova Auth Docs](https://abrova.ir/cloud/abrova-auth/docs)
- **Issues**: [GitHub Issues](https://github.com/Abrova-co/abrova_auth_web_sdk/issues)
- **Email**: support@abrova.ir

## License

MIT License - see [LICENSE](LICENSE) for details.
