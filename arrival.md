# Arrival

This document owns the seconds before the product is a product: the tab the page opens in, the colour behind the
status bar, the first frame that paints, the icon on the home screen, the screens that take a person from a link
to an account, and the first session of an account that follows nobody. The feeling it protects is that Surf was
already there. Nothing is assembled in front of the person, nothing snaps into its real colour, nothing asks for
anything before it has given anything, and the first thing a new account sees is content rather than an
explanation of what content would be.

## Owns and defers

Owns: the document identity (title, favicon, theme colour, declared colour scheme), what lands on `<html>` before
the first frame, the manifest and the installed posture, sign-in and sign-up and the invite link, verification and
MFA entry, onboarding as a flow, the first ten seconds of a new account, and the rule that nothing is asked in the
first session until an act has earned it.

Defers: theme palettes and the runtime theme switch to [colour-and-effects]; safe areas, postures and breakpoints
to [layout]; bones, holds and the four states to [loading-and-states]; boot budgets, the boot document and
prefetch policy to [speed]; keyboards, autofill tokens, autofocus and validation placement to [input-and-forms];
what a permission prompt looks like, when it is asked and where denial leads to [permissions-and-trust]; Back,
Escape and history within a flow to [navigation-and-history]; unread in the title, favicon and PWA badge, toasts
and interruption priority to [feedback]; offline, session expiry and returning after hours to [resilience]; every
string to [copy-and-voice]; curves and durations to [motion].

## Rules

### The document

1. **One title, one template.** The root sets `title.default` to the product name for the host and
   `title.template` to the page "on" the name, in the reader's language (`app/layout.tsx`, `generateMetadata`),
   with no glyph between them `[craft §10a]`. A route names itself through `localizedMetadata(key)` so the tab, the
   history entry and the bookmark are in the reader's language. A route whose tab reads only the product name is a
   defect.
2. **The mark answers per host, never per route.** `app/icon.tsx` and `app/apple-icon.tsx` resolve
   `resolveAppMode(host)` and serve that product's file from `BRAND_ICONS` or `MESSENGER_ICONS`. The tab icon is
   512, the touch icon is 180 (`TAB_ICON_SIZE`, `APPLE_ICON_SIZE`). One mark for the life of the product: a
   favicon that changes is a different app in the tab strip.
3. **Browser chrome wears the applied theme.** `themeColor` in the `viewport` export carries one colour per OS
   scheme from `THEME_COLOR_METADATA`, which is the pre-hydration answer. Once a theme is applied,
   `applyThemeChrome` (`lib/theme/chrome.ts`), the one writer, sets that theme's `metaColor` on a tag of its own
   placed first in the head with no media query, so it outranks the static pair without editing tags the
   framework re-renders. It runs inside `ThemeProvider`'s `applyThemeToDocument`, beside `data-theme`, and
   leaves the applied theme in the `surf-theme-plate` cookie, from which a native shell paints its splash.
4. **`color-scheme` is declared by the theme, not inferred.** Every theme block in `app/globals.css` sets
   `color-scheme` to its `ThemeScheme`, so native scrollbars, form widgets and the overscroll gutter match the
   palette. A theme block that omits it renders a dark page with light native controls.
5. **The first frame is already correct.** `buildThemeBootScript()` runs blocking, as the first node in `<body>`,
   and applies everything React would otherwise apply in an effect: `perf-lite`, `reduce-motion`,
   `data-font-size`, `data-theme`, `data-theme-scheme`, and the empty `touchstart` listener that WebKit needs
   before it will paint `:active`. Any value that changes the first frame belongs in that script and nowhere else.
6. **The boot script cannot take the page down.** It reads storage in independent `try` blocks, one per concern,
   because `localStorage` throws rather than returning null in a partitioned context, and an escape from a
   blocking inline script is a white page. `__tests__/theme/boot-script.test.ts` pins every branch.
6a. **A browser below the floor is told so.** `buildFloorProbeScript()` runs ahead of the theme script and sends a
   browser without `color-mix` or one whose parser rejects the app's syntax to `/offline.html#update`, the static
   plate in the nine languages, instead of a white screen. A `Function` the CSP forbids is not a verdict.
   `__tests__/boot/floor-probe.test.ts` pins it, and `__tests__/pwa/the-offline-page-speaks-the-apps-language.test.ts` pins
   the plate's words and colours against the locale files and the theme registry.
7. **No hydration warning is suppressed beyond the document.** `<html>` and `<body>` carry
   `suppressHydrationWarning` because the boot script writes to both; nothing below them may.

### Install and standalone

8. **One manifest per host.** `app/manifest.ts` resolves the mode and answers with that product's name, short
   name, description, icons, `start_url` from `MODE_LANDING`, shortcuts, and `orientation` (`any` on every
   host, because a phone is turned sideways for a video and every route already works rotated [platform]). Installing from the messenger host opens the messenger.
9. **The icon set is complete.** 192, 512 and a separate `maskable512` that is inset for Android's launcher mask.
   A maskable icon is never offered as `any`; the glyph would float small inside its own safe zone.
10. **The install prompt is caught before React exists.** `buildInstallBootScript()` stores the event at
    `INSTALL_PROMPT_SLOT` and calls `preventDefault()` so Chromium's mini-infobar never races the product's own
    offer. A listener added in an effect misses the event, and the event is not replayed.
11. **Install is a destination, not an interruption.** `/install` is the whole offer: one tile, one line, and
    either the one-tap button or the honest manual steps for that browser (`InstallMethod`: `ios`, `macSafari`,
    `android`, `chromium`, `unsupported`). It never waits for a prompt that may never fire, and nothing anywhere nags.
    The one offer inside a flow is the terminal onboarding step (34): a step, never a layer, skippable, and
    only where it can be acted on — a touch device that is not already standalone.
12. **Standalone is the same product, not a mode.** `viewportFit: 'cover'` and
    `interactiveWidget: 'resizes-content'` with `appleWebApp.statusBarStyle: 'black-translucent'`, content
    clearing the insets [layout], and `AppViewportHeight` publishing `--app-vh` from `visualViewport` because
    `100svh` ignores the keyboard in an iOS PWA.
13. **The scale is never capped.** No `maximumScale`, no `userScalable: false`; capping fails WCAG 2.2 SC 1.4.4
    on every page [accessibility].
14. **A launch image exists for every device that asks for one.** `(not yet built)`: no
    `apple-touch-startup-image` is declared, so an installed iOS app opens on a white flash before the first (not yet built)
    paint. It needs a generated set keyed to `THEME_PLATE`, one per device class, emitted beside the touch icon.
15. **A new build announces itself once and waits** `[resilience §31]`. An offer not yet taken still leaves the
    newer worker installed for the next load.
16. **The worker registers on idle and never serves stale code online.** `useServiceWorker` registers inside
    `useIdleEffect` because the worker's value is entirely for the next navigation; `public/sw.js` is
    network-first for assets, network-only for `/api/`, and its caches are the offline net only.

### Sign in and sign up

17. **The fastest way in is first, and a way in that cannot work is not offered.** The entry screen offers,
    in this order: Surf's own session where another surface holds it (`SurfHandoffButton`, "Continue with
    Surf" wherever it is offered: the Surf app's account on Messenger, in its app and on its website, and in the studio on a phone, the browser's in the Surf app, seeded from
    the request by `entryOffers`), passkey, Google, Apple (`AppleButton`, Google's own control at Google's own
    size, as App Review 4.8 asks), then email behind its own button (`EntryChoice`), which opens the form in their
    place as a step Back returns from (`ENTRY_METHOD_QUERY`); with nothing above it the form is the screen. Inside an app a provider is offered only by the route its shell has (`ProviderRoute`): Google's
    native id token, or the system sheet where the app has no Google client yet, since Google refuses a webview, and Apple natively or through the system sheet. Passkey renders only where the
    platform has one (`usePasskeysAvailable`, `isPasskeyHost`); when it does not, Google moves up. Both
    entry screens use that one order. Inside a browser embedded in another app (`embeddedBrowser`) Google
    is not offered at all: `GoogleButton` renders the one action that leads somewhere — the page handed to
    the real browser on Android, the address copied to paste on iOS — and the email form below it is
    untouched. The browser is read off the request's own user agent and seeded through
    `EmbeddedBrowserProvider`, so the first paint is already right rather than withdrawing a control after
    hydration; a Chrome Custom Tab and an `SFSafariViewController` carry the host browser's user agent and
    are never flagged. The passkey offer is seeded the same way, from the request's host through
    `PasskeyHostProvider`, so it is in the first paint rather than pushing the form down after hydration;
    inside an embedded browser, whose WebAuthn is the host app's to grant, it waits for the client's answer.
18. **Zero fields where the platform can answer.** A returning person signs in without typing: the passkey
    assertion seats the session and `finishSignIn` only has the landing left to do. `(not yet built)`:
    conditional mediation, so a passkey is offered in the email field's own autofill sheet rather than behind a
    button. It needs `mediation: 'conditional'` on the `startAuthentication` call and the `webauthn` token added (not yet built)
    to the email field's `autocomplete` [input-and-forms].
19. **The entry screen paints its real controls.** The first screen a person ever sees renders its heading,
    provider buttons and fields at their final shape; it never stands bones in for its own UI
    [loading-and-states].
19a. **The entry screens are one frosted card over today's posts.** `GlassAuthLayout` with `showVisual` lays
    `EntryBackdrop` on the plain canvas: today's public posts as their own colours, painted from blurhashes alone
    (`readEntryBackdrop` keeps only approved, safe posts by adult accounts in good standing), in narrow tilted columns
    that alternately rise and sink and dissolve into the canvas. Messenger borrows no posts: the same tilt and drift
    carry wordless message bubbles in short runs from either side, the person's own in `MESSENGER_BRAND`'s purple
    (`ThreadBackdrop`). One `GLASS_CARD_CLEAR`, centred at every width,
    carries the mark large and the form beneath it, the form's heading visually hidden since its controls name the
    screen; it holds still, since a moving blur re-rasterises every frame.
20. **Joining needs no invite, and an invite never stands in the way.** `/signup` opens for anyone. An
    invite is a link, never a code to type: `inviteUrl` builds `/join/:code`, `JoinClient` carries the
    code on to signup in the address and in `COOKIE_KEYS.referralCode`, and `useAttachCarriedInvite`
    credits it to its sender as onboarding opens, as it does the code a Play install carried. A link
    followed by an account already signed in attaches at once (`useFollowInvite`). A code that will not
    spend (unknown, used up, its owner suspended) costs its sender a credit and the person following it
    nothing: no screen refuses it, asks for another, or names the failure. The retired `/enter-referral`
    address redirects to `/signup` with its `invite` carried over.
21. **A refusal names itself before it is spent, and a second press is not a second attempt.** A sign-in that
    is about to be paused warns first, and once paused states the wait in seconds. A failure from the auth
    provider is mapped to copy about what to do next, never a generic banner `[loading-and-states]`. One
    attempt runs at a time: the pressed control keeps the spinner and every other one goes inert. A bound the
    server enforces is graded in the field as it is typed and holds the submit closed `[input-and-forms]`. Every
    advisory on the screen is `ErrorAlert` at a different tone, never a second look-alike.
22. **Verification is a code, typed or pasted, and every hand-off resolves on one origin.** The emailed link
    opens a Continue page whose one press finishes the step; opening the link changes nothing, and a link that
    would sign in a different account than the one signed in asks "Switch to @y?" first. An app handoff is the
    same decision. Both paths exist because a person reads the mail on the other device as often as on this
    one. When a hand-off fails, the entry screen names the reason as copy about what to do next.
23. **Auth screens have no query cache.** `app/(auth)/**` renders outside `QueryClientProvider`; a hook that
    reaches for the cache blanks the page and the route still answers 200.
24. **Adding an account is a utility, not a marketing moment.** With `?addAccount=true` the entry screen drops
    the aura for the plain card, shows a back control, and blocks re-adding the account already signed in.

### Onboarding

25. **Chrome holds still; only the step moves.** The progress line, the step count and the back control stay put
    across every step. The progress bar scales on transform and is one continuous shape, never a row of dots.
    The card is one height per posture (`GlassAuthLayout` with `anchor="start"`), so a step taller than it
    scrolls inside it under the fixed Continue, and `scroll-edge-fade` says so; the page behind never scrolls.
26. **Eight steps, nine where the install offer lands, and only four of them can refuse.** Age, name and
    username, at least `MIN_TOPICS` interests, and the human check gate the Continue button
    (`canProceedFromStep`, keyed by step); avatar, background, algorithm and accounts pass on Continue, and on
    the interests step Continue names how many are still needed. Nothing in the flow is mandatory that the
    product can run without. The three profile steps each show `ProfileCardPreview`, the card as others will see
    it, built from the answers so far; one merged step does not fit the fixed card on a phone. The algorithm step
    is the feed panel's feel and time with `FeedPreview` ranking on the answers held in the flow
    (`firstSessionPrefs`, which is also what finishing writes), and Never show folded under them; a preview with
    no posts to rank collapses rather than framing nothing. The human check opens the camera as its step arrives
    and asks every turn and both distances, so the ring fills all the way round; a device with no camera skips
    the step itself [permissions-and-trust §2]. Skipping asks once and names what stays closed: going live.
    Once the check settles its stage is gone, never left transparent over the page: the step says how it
    ended, and where it is the last step, Continue carries the finish and retries a refused one. Where the
    account's jurisdiction (`JURISDICTION_RULES`) draws more than a floor, two more steps follow the profile and
    refuse until they are met: the guardian step, below `guardianRequiredUnder`, takes the guardian's invite code
    through `GuardianCodeForm`, and the age check step, wherever `assurance` is beyond self-declaration, asks the
    store's signal first and a reviewer otherwise. Both are shown from the rule and the answer, never from the
    live status, so neither vanishes under the person answering it.
27. **Back unwinds one step.** One `OverlayHistoryLevel` per completed step, so the platform back gesture leaves
    a step rather than the flow [navigation-and-history].
28. **A reload costs nothing.** Every serialisable answer is written to `STORAGE_KEYS.onboardingProgress` on
    change and restored after mount, never in a lazy initialiser, because the server renders step 0 and
    initialising elsewhere is a hydration mismatch. `clearOnboardingProgress()` runs on finish.
29. **Onboarding collects; it does not explain.** A step is a question with its controls: the question is
    `StepHeading`, centred, and the controls answer it below. No paragraph describes a feature the person has
    not used yet, and no step exists only to be read [copy-and-voice].
30. **The flow adapts to the product it is in.** The messenger has no feed to rank, so its signups never see the
    algorithm step; the list is filtered by step key, never by index.

### The first ten seconds

31. **The first screen after onboarding is content.** The ranked feed reads `network_scope` as `all`, so an
    account that follows nobody still lands on posts. A follow graph is never a precondition for a feed.
32. **Zero follows is a designed state with exactly one action.** When the feed genuinely has nothing, the
    `EmptyState` names it in one line and offers one destination, Explore, which is the action that fills it.
    Never two actions, never a wall of suggestions [loading-and-states].
33. **Nothing is asked until an act earns it.** No permission prompt, hard or soft, fires from a timer, a page
    load or a session count. Each one is attached to the act that needs it, and the rules for which act, which
    explanation and which denial path are in [permissions-and-trust].
34. **One interruption at a time, and none in the first session.** The update offer, the install offer, the
    notification card and any upsell are all interruptions; the first session shows none of them, and after it
    they queue by the priority in [feedback] rather than stacking. The skippable install step that ENDS
    onboarding (11) is the one exception: a step the person is already in is not an interruption of it.
35. **A new account is never shown the test suite.** Suggested accounts, the first feed and every people list
    exclude `is_bot` rows.
36. **A visitor's write is a way in, never a refusal.** Any write a signed-out visitor presses — like, repost,
    save, reply, follow — opens `GuestActionSheet` instead of running: the thing it was pressed on with the
    action already drawn, "Sign in to like @maya's post", Sign in and Sign up. Nothing lights up
    and falls back, and no toast refuses. Both carry `next` with the pending `act`, which
    `GuestActionResumer` finishes on the page they return to; joining holds the way back on the device
    (`holdReturnAfterJoin`) because the confirm link can open a new tab. `SignUpBanner`'s links carry `next` too.

## The entry sequence

| Route | What it is | What it can ask for |
|---|---|---|
| `/` | Redirect: carries a grant or a provider refusal (`CALLBACK_PARAMS`) to `app/auth/callback`, else to `/login` | nothing |
| `/join/:code` | An invite link: carries its code to signup, or credits it at once for an account already signed in | nothing |
| `/signup` | Google, Apple, or email and password; open to anyone | email, password |
| `/verify-email` | The emailed code | the code |
| `/login` | Passkey, Google, email and password | credentials |
| `/mfa-verify` | The authenticator code when the session is at aal1 and aal2 is enrolled | the code |
| `/auth/email` | An emailed link's Continue page: one press, or "Switch to @y?" when another account is signed in | the press |
| `/auth/handoff` | Surf's "Sign in to Surf Messenger as @x?": the press mints a one-minute code, never opening the page | the press |
| `/auth/native-return` | Where a browser flow returns to an app: a provider's code goes on to the callback; a handoff code asks "Continue as @x?" | the press |
| `/onboarding` | Eight steps, nine on a touch device (11), plus the guardian and age check steps where the jurisdiction asks (26); the profile row is written at step 2 | age, name, username, interests, human check, a guardian's code, an age check |
| `/install` | The install offer, per browser | nothing |
| `/home` | The ranked feed | nothing, in the first session |

## What lands on `<html>` before the first frame

| Written by | Attribute or class | Source |
|---|---|---|
| `buildThemeBootScript` | `perf-lite` | device memory, hardware concurrency, iOS major, `saveData`, and a shell's Reduce Transparency |
| `buildThemeBootScript` | `high-contrast`, `data-bold-text` | a shell's Increase Contrast and Bold Text (`__SURF_BRIDGE__.a11y`) |
| `buildThemeBootScript` | `reduce-motion` | `STORAGE_KEYS.localSettings` |
| `buildThemeBootScript` | `data-font-size` | `STORAGE_KEYS.localSettings` once the viewer has picked a size, else the shell's text scale, default `DEFAULT_FONT_SIZE` |
| `buildThemeBootScript` | `data-theme`, `data-theme-scheme` | `STORAGE_KEYS.theme`, resolved through `SYSTEM_DARK_QUERY` |
| `RootLayout` | `lang`, `dir` | `resolveDocumentLanguage()` |
| `RootLayout` | the font variable class | `APP_FONT_VARIABLE_CLASS` |
| `buildInstallBootScript` | `window[INSTALL_PROMPT_SLOT]` | `beforeinstallprompt` |
| `AppViewportHeight` | `--app-vh` | `visualViewport.height`, after mount |

## Enforced by

- `__tests__/components/auth/guest-action.test.tsx` and `__tests__/components/layout/sign-up-banner.test.tsx`: rule 36.
- `__tests__/theme/boot-script.test.ts`: 5, 6.
- `__tests__/fitness/browser-chrome-wears-the-applied-theme.test.ts` and `__tests__/theme/theme-chrome.test.ts`: 3.
- `__tests__/pwa/install-boot.test.ts`: 10.
- `__tests__/pwa/install-route.test.ts`: 11.
- `__tests__/onboarding/step-install.test.tsx`: 11 and 34, for the terminal offer and its refusal.
- `__tests__/fitness/user-facing-copy-is-translated.test.ts`: 1, 29.
- `__tests__/fitness/a-people-list-excludes-the-test-accounts.test.ts`: 35.
- `__tests__/fitness/an-empty-state-is-the-primitive.test.ts`: 32.
- `__tests__/fitness/history-state-has-one-owner.test.ts`: 27.
- `__tests__/components/auth/join-client.test.tsx` and `__tests__/hooks/use-referral.test.ts`: 20, for every
  way a followed link goes on and the invite a new account arrived with.
- `__tests__/supabase/signup-is-open-at-the-door.test.ts` and `e2e/invite-links.spec.ts`: 20, for the door
  that stays open and the dead link that still leads through it.
- `__tests__/hooks/use-auth-submit.test.ts`: 21, for the mapping, the report and the one-attempt lock.
- `__tests__/hooks/use-entry-auth.test.tsx`: 21, for a fresh captcha token per attempt; 22, for the origin
  the Google hand-off names and the refusal everywhere else.
- `__tests__/utils/embedded-browser.test.ts`: 17, for the two classes of browser, off real user agents.
- `__tests__/components/auth/google-button.test.tsx`: 17, for the offer, the way out and the seeded read.
- `__tests__/components/auth/passkey-button.test.tsx`: 17, for the passkey offer the server's HTML already
  carries, and the host and the embedded browser it is not promised to.
- `__tests__/domain/auth-redirect-targets.test.ts`: 22, for every address handed to GoTrue.
- `__tests__/domain/entry-failure.test.ts`: 22, for each reason and the copy it is read as.
- `__tests__/api/auth-callback.test.ts`: 20, 22, for where each failure lands and what it files.
- `__tests__/fitness/a-deep-link-get-changes-nothing.test.ts`: 22, for an emailed link that only a press spends.
- `__tests__/app/onboarding-layout-clears-the-way-in.test.tsx`: 20, for the account no invite stands behind.
- `e2e/oauth-entry.spec.ts`: 17, 22, at 390px under an Android Chrome and an in-app user agent.
- `__tests__/validations/password-rules.test.ts`: 21, for the ceiling the schema and the field share.
- `__tests__/theme/theme-contract.test.ts`: 4, for the `color-scheme` every theme block declares and the
  `ThemeScheme` the registry answers with.
- `__tests__/fitness/every-route-names-its-own-tab.test.ts`: 1, for the route that names no tab of its own and
  the title fixed at build rather than resolved per request.
- `__tests__/fitness/the-mark-is-resolved-by-host.test.ts`: 2, for the host read, the mode map and the size each
  mark route declares against the raster it serves.
- `__tests__/fitness/the-first-frame-needs-no-effect.test.ts`: 5, for what a provider stamps on `<html>` after
  hydration that the boot script never applied.
- `__tests__/fitness/one-order-for-the-ways-in.test.ts`: 17, for the order every screen offering a way in
  renders them in.
- `__tests__/fitness/an-entry-screen-paints-its-controls.test.ts`: 19, for the auth route ghost and the entry
  screen that stands one in for itself.
- `__tests__/components/onboarding/step-human-check.test.tsx` and
  `__tests__/components/onboarding/onboarding-flow.test.tsx`: 26, for the settled check and the finish it carries.
- `__tests__/fitness/an-onboarding-step-is-gated-by-its-key.test.ts`: 26, for the gate on Continue deciding by
  key rather than by position.
- `__tests__/fitness/nothing-is-asked-on-a-timer.test.ts`: 33, for the ask raised by a clock, an idle callback
  or a route's own mount, and for the card that leads to one.

**Not yet enforced**

- `an-installed-app-has-a-launch-image` (14): the metadata declares an `apple-touch-startup-image` for every
  device class, and each file exists. Blocked: neither the generated set nor the declaration exists yet.

## Traps

- 5: One shared `try` around the boot script's storage reads let a theme-read failure take the device-tier stamp
  down with it, and the page painted frosted then visibly flattened. One `try` per concern.
- 5: Framer captures the reduced-motion preference when it constructs a component, so a value that only arrives
  at hydration never reaches the initial tree.
- 5: `-webkit-tap-highlight-color: transparent` plus WebKit gating `:active` on the presence of a touch listener
  means zero press feedback on iOS; the boot script adds the empty listener.
- 10: Chromium fires `beforeinstallprompt` long before hydration and never replays it.
- 16: The browser can silently rotate a push subscription, which leaves the device quietly dead;
  `pushsubscriptionchange` re-subscribes and re-registers the endpoint.
- 17: Google answers OAuth inside an app's own WebView with `disallowed_useragent` and shows its own error
  page, so the browser never comes back: 51 of 54 recorded Google hand-offs left and never returned.
- 18: GoTrue answers a wrong password and a Google-only account identically, so the hint offers Google as a
  possibility and never asserts it.
- 21: A password over 72 BYTES is what bcrypt refuses, not 128 characters: the form took it, GoTrue answered
  `validation_failed`, and the generic banner hid both the cause and the fact that it had happened at all.
- 22: A route that answers with a redirect freezes before an unawaited monitoring POST leaves, so every
  callback failure since the sink existed went unrecorded; `deliverError` is awaited after the response.
- 23: A passkey hook that called `useQueryClient` blanked production `/login` for twenty-five minutes, and the
  route still returned 200. Only a browser catches it.
- 28: Restoring onboarding progress in a lazy initialiser is a hydration mismatch: the server renders step 0.
