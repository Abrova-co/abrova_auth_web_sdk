## [0.5.1] - 2026-11-23

- Ready-made sign-in screen: new option `sendVerificationEmail`. Set it to `false` when your project doesn't require a
  confirmed e-mail and you don't want people to get the confirm e-mail after creating an account; they are signed in at
  once. A project that requires a confirmed e-mail still sends it. Default `true`, as before.
  ```js
  await openSignIn(auth, { sendVerificationEmail: false });
  ```
- Ready-made sign-in screen: an eye button on the password fields shows or hides what was typed ("Show password",
  in Persian too). Hidden again on every new screen.

## [0.5.0] - 2026-11-16

- `resendVerification({ email, password })`: when your project requires a confirmed e-mail, someone who can't sign in
  yet (`EMAIL_NOT_VERIFIED`) gets the confirmation link again. The e-mail and password prove the account; a wrong pair
  fails like signing in. Nothing is sent to an account that is already confirmed.
- `continueUrl` on `signUpWithEmail`, `sendEmailVerification`, `sendPasswordReset` and `resendVerification`, like
  Firebase's continue URL: Abrova's own confirm and reset pages show a "Continue" button to it when they are done. It
  must be one of your project's link addresses.
- `sendEmailVerification`: `redirectUrl` is optional; without it the link opens Abrova's own confirm page.
- Sign-in screens: "Send again" when signing in fails because the e-mail isn't confirmed yet, and a `continueUrl`
  option.
- Sign-in screens: all languages are kept in one table.

## [0.4.0] - 2026-11-08

- Ready-made sign-in screens, like FirebaseUI, for any framework (or none): `import { openSignIn } from
  '@abrova/auth-web/ui'`, then `const result = await openSignIn(auth)`. It opens a dialog (the whole screen on
  phones) and answers `{ userId }` once the user signed in, or null when they closed it. `mountSignIn(element, auth)`
  puts the same form on any page.
  - Shows only the methods turned on in the Console: e-mail and password (sign in, create account, forgot password),
    magic link, Telegram, Google, Apple and guest.
  - English and Persian (right to left), from the browser's language or `languageCode`. The e-mails Abrova sends
    follow the screen's language, unless you set `auth.languageCode` yourself.
  - Its own styles in a shadow root, so your page's CSS cannot break it. Change the look with `color`, `appName`,
    `logoUrl` or CSS variables such as `--abrova-accent`; dark mode follows the system.
  - Google and Apple: pass a function that gets an ID token for the given hashed nonce (Google Identity Services,
    Sign in with Apple JS). Magic link: pass `magicLinkRedirectUrl`.
  - "Continue as guest" shows only with `allowGuest: true`: every guest is a new user and counts toward your plan.
  - No new dependencies.
- `sendPasswordReset`: `redirectUrl` is optional; without it the link opens Abrova's own reset page.
- `baseUrl`: the Abrova Auth server the client talks to.
- `isAbrovaAuthError` also recognises an error from another copy of the SDK (for example the CommonJS and ESM builds
  in one app).

## [0.3.0] - 2026-11-07

- `languageCode` and `useDeviceLanguage()`, like Firebase: the language of the e-mails Abrova sends this user
  (confirm e-mail, password reset, magic link), for example `auth.languageCode = 'fa'`. Unset, the project's template
  language from the Console applies. Also a `languageCode` option in the config.

## [0.2.0] - 2026-11-05

- `getTokenResult({ forceRefresh })`: the access token with its decoded fields, like Firebase's
  `getIdTokenResult()`. `claims` holds the custom claims your server set with `setCustomUserClaims`, at the top level
  (for example `plan`, `role`), plus `sub`, `iat`, `exp`, `auth_time` and `amr`; also `expirationTime`,
  `issuedAtTime`, `authTime` and `signInProvider`. For showing things in the app; access is still decided on the
  server. Pass `{ forceRefresh: true }` right after your server changed the claims.
- `decodeTokenPayload(token)` helper.

## [0.1.1] - 2026-10-20

- Sends `X-Abrova-SDK: auth-web/0.1.1` with every request so Abrova can show which SDK versions are in use. No behaviour
  change.

## [0.1.0] - 2026-10-16

- Initial release: anonymous, Apple, Google, e-mail + password, magic link and Telegram sign-in; linking a method to
  the signed-in user; `getMe`, `signOut`, `signOutEverywhere`, `deleteAccount`.
- Sessions refresh themselves (one request for concurrent callers); rotated refresh tokens are saved.
- Works with the other Abrova SDKs: `() => auth.getToken()` for Abrova Sync, Push and A/B Testing (null while signed
  out), `auth.tokenProvider` for Abrova Chat (rejects while signed out).
- Memory storage by default; opt-in `localStorageStorage()`.
- Example app in `example/`.
