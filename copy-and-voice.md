# Copy and voice

Every word a person reads in Surf comes from one writer. This document owns that writer: what the product sounds
like, how much it says, what it calls things, and what it says when something refuses. The feeling it protects:
a person is never talked at, never apologised to, never sold to mid-task, and never left holding a sentence that
does not tell them what to do next.

## Owns and defers

Owns: the voice, sentence case, person and tense, length, one term per concept, captions and helper prose, error
and refusal copy, empty-state copy, toast grammar, notification and push copy, email copy, and the shape of a
count.

Defers: the message catalogue's mechanics, pluralisation rules, locale, dates, numbers and RTL
`[internationalisation]`; which primitive shows a message `[components]`; when a toast appears, how long it
stays, and undo-versus-confirm `[feedback]`; which state a surface is in `[loading-and-states]`; what a paywall
or permission prompt must disclose `[permissions-and-trust]`; type size, weight and truncation `[type]`;
accessible names and live announcements `[accessibility]`.

## Rules

### One voice

1. **Sentence case, always.** "Follow requests", not "Follow Requests". A proper noun keeps its capitals; a
   feature name is a proper noun only where the product sells it by name. Title case in the middle of a sentence
   reads as a different product's string pasted in.
2. **No first person.** The product has no "we", "our" or "us". It says what happened and what the reader can
   do. "We'll notify you when they go live" is the product talking about itself; "You'll be notified when they
   go live" is the reader's own screen.
3. **No exclamation marks.** An exclamation is the product raising its voice. A moment worth celebrating earns
   it with motion and space `[motion]`, never with punctuation.
4. **No "please", no "simply", no "just".** "Please" apologises for an instruction the reader did not ask for.
   "Simply" and "just" claim the thing is easy, which is only ever said when it is not.
5. **Short enough to read in a sheet.** Past 110 characters a string is a paragraph on a 390px screen. Cut it to
   the sentence that changes what the reader does.
6. **One term per concept, everywhere.** Retry or Try again, not both. Go home or Go back, not both. The same
   word for the same action in the row, the menu, the toast and the empty state.
7. **Present tense, and the reader is the subject.** "Only followers can see your posts", not "Your posts will
   only be visible to followers".
8. **No jargon the reader did not choose.** A term the product invented is defined the first time it is used or
   it is not used.

### Copy says it once

9. **A label is the whole row.** Icon, label, chevron: a settings row, a list row, a toggle and a menu item alike.
   No caption underneath repeating the label in other words. "Follow requests" needs no "Pending follow requests"
   beneath it, and a page title needs no subtitle saying what the page lists. An empty state holds one line
   `[loading-and-states §30a]`.
10. **A description exists only when it names something the label cannot.** What the control actually changes,
    what the reader will lose, who will see it. "Fine-tune the flavor of posts you see" names nothing; "Hide
    posts from accounts marked as automated" names exactly what happens.
11. **No helper prose.** A toggle needs no sentence. A field needs a placeholder, not a caption
    `[input-and-forms]`. A dialog has a title and controls, not an introduction. Onboarding does not have
    paragraphs.
12. **One string, one key.** A sentence that appears under two keys is two sentences the day one of them is
    edited. Where a word is genuinely shared, it is one key read from both places.
13. **Never a sentence assembled from parts.** No `Failed to {verb} user`, no label concatenated with a value to
    make a clause. Word order is not the same in nine languages `[internationalisation]`.

### Errors, absence and refusal

14. **An error names what refused it.** "Couldn't save: the name is taken", never "Something went wrong".
    `surfaceError` and `describeError` in `lib/domain/errors/` are the only place a caught failure becomes
    words, and the only place allowed to say it does not know: `error.reported` carries a reference the reader
    can quote.
15. **Absence is not failure.** A missing post, profile or id is a not-found state that names what is missing:
    "This account doesn't exist". Only a refused request is an error. The two never share a screen or a word.
16. **A refusal names the thing, not the layer.** No status codes, no table names, no driver strings. A caught
    `error.message` is untranslated and names columns; it never reaches a reader.
17. **Every gate names itself and offers the way through.** A lock, limit, paywall or permission wall says what
    it is and carries the one action that opens it `[permissions-and-trust]`. A dead end is a defect.
18. **A rollback says what was refused, not that something was refused.** "Couldn't post: you're offline", not
    "Failed".

### Empty states

19. **An empty state is a title, one line and at most one action.** `EmptyState` is the primitive
    `[components]`. The line says what will be here, not that nothing is.
20. **An empty state that is really an error says so.** "Something went wrong. Tap to retry." under an empty
    illustration is a shrug wearing an empty state's clothes; it belongs to rule 14.
21. **An empty state never sells.** It points at the one action that fills it, in the reader's own words.

### Toasts

22. **A toast is an object and a past participle.** "Post deleted". "List created". "Members added". No subject,
    no full stop, no adverb, no exclamation. `success.mfaEnabled` reading "2FA enabled successfully" says
    "successfully" twice: once in the word, once in the fact that a toast appeared.
23. **A toast reports; it never asks.** A question, an offer or a promise is a dialog, a row or nothing at all.
    "Coming soon" is a promise in a toast and should be a disabled control with the reason one tap away.
24. **A toast that offers undo says what happened, then Undo.** "Post deleted", and the Undo action beside it. The
    window is drawn, never written `[feedback §22]`.
25. **A failure toast is not in the success namespace.** A key that begins `success.` and reads "Failed to …" is
    two mistakes: the key lies and the sentence is assembled.

### Notifications and push

26. **Every notification type has its own copy.** `lib/email/templates/notification.ts` is the single source for
    both push and email, keyed by `notification_type`. A type without its own entry falls back to "New activity
    on Surf", which is honest; falling back to another type's sentence is not, and a follow request once read
    "liked your post".
27. **Lock-screen copy says that something happened, never what.** A message mention, a safety alert and a Radar
    match name the actor and the surface and nothing else. The in-app row carries the detail.
28. **A push title is the subject line.** `notificationPushText` returns the same subject and body the email
    uses, so a person who gets both does not read two different sentences about one event.
29. **A batch says how many, not a list.** One notification keeps its own subject; more than one says the count
    `[internationalisation]`.

### Email

30. **One card, one layout.** `emailLayout` in `lib/email/templates/layout.ts` is every transactional email: the
    wordmark, a title, the body, at most one action inside the card, and only small print outside it.
31. **The preheader says the thing.** It is the line the inbox shows beside the subject. Never "view this email
    in your browser".
32. **A code says how long it lasts.** `emailCode` prints the code and "Expires in N minutes" beneath it, from
    `OTP_EXPIRY_MINUTES`, which is the same number pushed to the auth config.
33. **One action per email.** A column of buttons is a page, not a message. A stack of items uses `emailItem`,
    where the whole row is the link.
34. **A security alert carries the facts and the escape hatch.** What happened, where, when, and what to do if it
    was not the reader. It never asks them to confirm anything by replying.
35. **Copy that leaves the app is in the reader's language too** `(not yet built)`. Nothing under `lib/email/`
    reads `preferred_language` and every template string is an inline literal, so an Arabic reader gets an
    English email and an English push. Both move into the catalogue and resolve against the recipient's
    language `[internationalisation]`.

### Counts and values

36. **A count comes from `countOf`.** Never a hand-written plural, never "(s)", never a bare `{count} items`
    baked into a sentence. `COUNT_NOUNS` in `lib/utils/format/format-number.ts` holds the nouns
    `[internationalisation]`.
37. **A missing value is `EMPTY_VALUE`**, the em dash, never "N/A", "none", "0" or an empty cell.
38. **A number in a sentence is a substitution, not a join.** `{count}` sits inside the translated sentence so
    each language can put it where it belongs.

## Before and after

Real strings from `lib/i18n/locales/en.json` that fail the rules above, with what passes.

| Key | Today | Rule | Passes as |
|---|---|---|---|
| `empty.feed.description` | Something went wrong. Tap to retry. | 14, 20 | Couldn't load your feed. Tap to retry |
| `error.reported` | Something on our side failed. It has been reported. Reference {reference}. | 2, 14 | Surf couldn't finish that. Reference {reference} |
| `settings.list.errorDescription` | Failed to load this list. Please try again. | 4 | Couldn't load this list. Tap to retry |
| `live.notifyOnToast` | We'll notify you when they go live | 2, 22 | Notification on |
| `identity.consentTitle` | Before we measure your face | 2 | Before this check reads your face |
| `auth.termsAgreement` | By creating an account, you agree to our | 2, 13 | By creating an account you agree to the Surf terms |
| `msg.nextgen.autoTuneSuggestion` | Based on your activity, we suggest adjusting your feed: | 2, 5, 7 | Your reading suggests these changes |
| `success.posted` | Posted! | 3, 22 | Posted |
| `ui.copied` | Copied! | 3, 22 | Copied |
| `auth.passwordSet` | Password set! You can now sign in with email and password. | 3, 5, 11 | Password set |
| `success.mfaEnabled` | 2FA enabled successfully | 22 | Two-factor on |
| `success.failedToBlock` | Failed to {verb} user | 13, 25 | One key per action: `error.failedToBlock`, `error.failedToMute` |
| `settings.nav.followRequestsDesc` | Pending follow requests | 9 | No description; the label is the row |
| `auth.signInSubtitle` | Sign in to continue to Surf | 9, 11 | Removed |
| `settings.feed.contentVibesDesc` | Fine-tune the flavor of posts you see. | 10 | Weights the topics you already read toward the top |
| `chat.crossDomainTransfer.retry` | Try Again | 1, 6 | Retry |
| `settings.restricted.toastEnabled` | Restricted Mode enabled | 1, 22 | Restricted mode on |
| `ui.max-uses-0-unlimited` | Max Uses (0 = unlimited) | 1, 11 | Max uses, with "Unlimited" as the zero state |
| `toast.comingSoon` | Coming soon | 23 | Removed; the control renders disabled with its reason |

## Enforced by

- `__tests__/fitness/copy-reads-as-one-voice.test.ts`: 3 (`copy-exclamation`, 14 keys), 5 (`copy-over-110`,
  32 keys), 6 (`copy-synonym`, 10 keys). All three are ratchets: the baseline may shrink, never grow.
- `__tests__/fitness/an-error-names-what-failed.test.ts`: 14, 16. Fences every shrug key to `lib/domain/errors/`
  and every raw `error.message` away from a reader prop.
- `__tests__/fitness/user-facing-copy-is-translated.test.ts`: 12, 35, and every string that must come from the
  catalogue, including a `getLabelOrFallback` key the catalogue never got, whose English literal nine
  locales then read.
- `__tests__/fitness/an-empty-state-is-the-primitive.test.ts`: 19.
- `__tests__/fitness/a-label-resolves-when-it-renders.test.ts`: 12; a key that renders as its own name is a
  missing string.
- `__tests__/fitness/copy-is-sentence-case.test.ts`: 1 (`copy-title-case`, 85 words across 56 keys). The
  allow-list holds the product and its plans, other companies and their software, a format, a key cap and a
  name a string quotes; a Surf feature is not on it, so "Restricted Mode" and "For You" are debt.
- `__tests__/fitness/copy-has-no-first-person.test.ts`: 2 (`copy-first-person`, 17 keys). "my" and "I" are the
  reader's own voice and stay.
- `__tests__/fitness/copy-does-not-apologise.test.ts`: 4 (`copy-apology`, empty: no "please", "just" or
  "simply" is left).
- `__tests__/fitness/a-count-comes-from-count-of.test.ts`: 36 (`plural-in-parentheses`, empty;
  `count-glued-to-a-noun`, 37 keys).
- `__tests__/fitness/a-string-appears-once.test.ts`: 12 (`copy-said-twice`, 409 keys holding 186 sentences
  between them). A single shared word is vocabulary; a shared sentence is two sentences waiting to disagree.
- `__tests__/fitness/a-toast-is-an-object-and-a-participle.test.ts`: 22 (`toast-ends-in-a-full-stop`,
  `toast-opens-with-a-pronoun` and `toast-carries-an-adverb` — 2, 2 and 1 keys).
- `__tests__/fitness/a-failure-is-not-a-success-key.test.ts`: 25 (`failure-in-the-success-namespace`, one key).

**Not yet enforced.**

- `an-email-string-is-translated`: 35. Fails on a bare string literal in `lib/email/templates/`. Waits on a
  catalogue read that takes a locale: `getLabel` resolves the active one, and a drain holds the recipient's.

## Traps

- **12** `lib/constants/messages.ts` exports a `Proxy` on purpose, so keys resolve at read time in the reader's
  locale. A group that mixes keys with formatters must spread it, or the formatters resolve to `undefined`.
- **14** A real signup failed on a database refusal and reached the reader as "Something went
  wrong. Please try again." Three keys held that sentence and 118 call sites spent them, so the copy could never
  say more than its weakest caller.
- **26** A single fallback template made every notification type read as the most common one: a follow request
  and a go-live push both said "liked your post".
- **27** Push copy lands on a lock screen a stranger can read. Naming the message, the query or the safety event
  is a privacy leak with no undo.
- **36** `pluralize` takes words, not keys, so every caller that passed an English literal printed English into
  the other eight locales.
