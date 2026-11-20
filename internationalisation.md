# Internationalisation

Surf ships in nine languages, one of which is written right to left and five of which are not Latin. This
document owns everything that has to be true for a reader in any of them to get the same product a reader in
English gets: the same words, in their language, on the first paint; the same layout, mirrored where reading
direction demands it and left alone where it does not; the same numbers, dates and counts, formed the way their
language forms them. The feeling it protects is that the product was written in the reader's language, not
translated into it.

## Owns and defers

Owns: the nine locales and how one is resolved, the catalogue and the one way a string reaches a reader,
interpolation, pluralisation, number and date and time formatting, the first day of a week, right-to-left
mirroring and what must not mirror, text-expansion budgets, and non-Latin script safety in anything that
normalises, searches or slugifies text.

Defers: the words themselves and the voice they are written in `[copy-and-voice]`; the type ramp, line
breaking, CJK wrapping and truncation `[type]`; the accessible name a control needs at all `[accessibility]`;
logical spacing and radius tokens `[tokens]`; how a layout behaves when a label grows `[layout]`; the locale
picker's own screen `[input-and-forms]`.

## The nine locales

`SUPPORTED_LANGUAGES` in `lib/constants/content.ts` is the roster; `lib/i18n/locales/` holds one JSON file per
code, and the two never disagree.

| Code | Endonym | Direction | Script |
|---|---|---|---|
| `en` | English | ltr | Latin |
| `es` | Español | ltr | Latin |
| `fr` | Français | ltr | Latin |
| `de` | Deutsch | ltr | Latin |
| `ru` | Русский | ltr | Cyrillic |
| `ar` | العربية | **rtl** | Arabic |
| `zh` | 中文 | ltr | Han |
| `ja` | 日本語 | ltr | Han, kana |
| `ko` | 한국어 | ltr | Hangul |

`RTL_LOCALES` in `lib/i18n/rtl.ts` holds exactly `ar`, and `isRtlLocale` tolerates a region subtag and casing
while matching the whole subtag, so `arn` stays left to right.

## Rules

### The catalogue

1. **Every reader-facing string is a key.** `lib/i18n/locales/en.json` is the source of truth and
   `TranslationKey` is derived from it, so a key that does not exist does not compile. A literal in a copy prop,
   a JSX text child, an `aria-label`, a `placeholder`, an `alt` or a `title` is a defect.
2. **A client component reads `useT()`; everything else reads `getLabel()`.** `getLabelOrFallback` and
   `useLabelOrFallback` are for the one case where a static English fallback must beat a missing key.
3. **A label resolves at read time, never at module load.** `activeLocale()` and `getLabel()` are called per
   read; a module-level copy freezes on English because the locale JSON loads after mount.
4. **A group of messages is `localized()` in `lib/constants/messages.ts`.** It returns a Proxy over keys, so a
   consumer writes `SPACE_MESSAGES.created` and the lookup happens in the reader's locale at that moment.
5. **A group that mixes keys with formatters spreads the Proxy.** `...localized({ … })` beside the formatter
   functions, so each half keeps its own type; putting a function inside `localized()` breaks the key contract.
6. **Every key exists in all nine files, with no extras.** A key added to `en.json` alone renders English to
   eight locales, silently, because the store falls back rather than failing.
7. **A translated value is translated.** Key parity says the key exists; it never said the value changed. A full
   English sentence shipped verbatim into another locale is a defect, not a fallback.
8. **A placeholder survives translation.** Every `{param}` in the English value appears, spelled identically, in
   all eight others; `interpolate` renders `{param}` literally when the value is missing, so a dropped
   placeholder is visible in production.
9. **A string outside the catalogue is a sentinel, not copy.** `DECRYPTION_FAILED_PLACEHOLDER` is the pattern:
   it is compared, not read, and it is documented as such where it is declared.
10. **The em dash is banned in every locale file**, in every language `[copy-and-voice]`.

### Resolving the reader's language

11. **The served document already knows the language.** `app/layout.tsx` writes `<html lang dir>` from
    `resolveDocumentLanguage()`; nothing assigns `lang` or `dir` in an effect, because an Arabic reader would
    get a left-to-right English first paint that flips after hydration.
12. **The account's setting wins, then the browser, then English.** `resolveRequestLocale` reads
    `preferred_language` through a signed cookie, falls back to the first shipped locale in `Accept-Language`,
    and to `DEFAULT_LOCALE` on any failure; `useLocaleInit` applies the same order on the client.
13. **The server can resolve a label.** `serverLabel()` and `generateMetadata` give a page title, an email and a
    push notification the reader's language; a hardcoded `metadata.title` ships one language to nine.
14. **Changing the language clears the server's cache.** `clearCachedServerLocale()` runs wherever
    `preferred_language` changes, or the next server render serves a stale `lang` and `dir`.
15. **A BCP-47 tag resolves to its base.** `zh-CN` is `zh`, `fr-CA` is `fr`, `ar-EG` is `ar`, on both the client
    store and the server.

### Direction

16. **The inline axis is logical, always.** `ms`/`me`, `ps`/`pe`, `start`/`end`, `border-s`/`border-e`,
    `rounded-ss`/`rounded-se`/`rounded-es`/`rounded-ee`, `text-start`/`text-end`. `local/no-physical-inline-axis`
    fails a physical spelling; `left-1/2`, `right-1/2` and `inset-x-*` are direction-neutral and exempt.
17. **A directional glyph flips.** A chevron, a caret, a back arrow, a forward arrow and a reply arrow render
    through `DirectionalGlyph` in `components/ui/directional-glyph.tsx`, or `directionalIcon` where a row renders
    whichever glyph it is handed, and both carry `INLINE_FLIP`. A flip, never a rotation: `rotate-180` turns a
    bend-up-left reply arrow upside down. A disclosure caret that opens passes `expanded` and points down in
    either direction; the row's closed chevron points at the edge the reader is travelling toward.
18. **A fill that grows mirrors its origin.** Any `origin-left` on a progress track, a story bar, a poll bar or
    a scrub fill pairs with `rtl:origin-right`; `STORY_TRACK_FILL` in `lib/constants/layout.ts` is the shape to
    copy.
19. **A transform that travels along the inline axis reads its sign from `useInlineDirection()`.**
    `translateX` is physical; a knob, thumb or track that slides toward the inline end negates its travel under
    RTL or it slides out of its own track. A horizontal scroll position is the same kind of number: `scrollLeft`
    runs 0 to −max under RTL, so it is read and written through `inlineScrollOffset`, `scrollToInline` and
    `scrollInlineToCentre` in `lib/utils/inline-scroll.ts`, and a placed popover passes `rtl` to
    `placeAnchoredPopover`, whose `align` is logical.
20. **A keyboard arrow follows the reading direction.** In a roving strip, ArrowRight moves toward the inline
    end; `rovingIndex` in `lib/utils/roving-index.ts` takes `rtl`, and `Tabs`, `ChoiceGroup` and the reaction
    row all step through it.
21. **A gesture mirrors with the layout.** Swipe-back comes from the inline start edge, a swipe-to-archive
    travels toward the inline end, and a paging track advances toward the inline end. A hook signs its travel
    with `useInlineDirection()` and measures an edge with `distanceFromInlineStart`, as `useSwipeBack`,
    `useNavSwipe` and `useSwipeGesture` do.
22. **Media coordinate systems never mirror.** The shorts editor stage, the story canvas, the crop stage and
    the camera HUD position against the media, not against reading order; they are exempt from
    `local/no-physical-inline-axis` by glob in `eslint.config.mjs`, never by a disable comment.
23. **Numbers, code, URLs, handles and timecodes never mirror.** A duration reads `1:05` in every locale, a
    `@handle` keeps its order, and a code block keeps its direction regardless of the surrounding paragraph.
    Text a person wrote takes its own direction: `RichText` renders `dir="auto"` and isolates a URL or an
    address left to right and a handle or tag in its own script, and `DisplayName` and a link card's title and
    description do the same, so English under Arabic keeps its full stop at the end. A post body, a reply, a
    quote and a caption are an `AuthoredBlock`, one `dir="auto"` paragraph with `text-start`, so English under
    Arabic also aligns from the left; the `RichText` inside it drops its own `dir`, because an auto paragraph
    skips every child carrying one. The composer's field and its overlay are `dir="auto"` too, and the `field`
    caret overlay lays out exactly as that textarea does. An @handle outside authored text is `Handle`, never a
    hand-written `@`, so it cannot mirror.

### Numbers, counts and time

24. **A number is formatted for the reader.** `formatCompactNumber`, `formatFullNumber`, `formatUsd`,
    `formatFileSize` and `EMPTY_VALUE` in `lib/utils/format/format-number.ts` are the whole set, and each reads
    `activeLocale()`. `1.2K` is `1.2万` in Japanese and `12 тыс.` in Russian.
25. **`toLocaleString` with no locale argument is a defect.** It resolves to the *runtime's* locale, not the
    reader's, so a German account on an American machine reads American formatting.
26. **A count noun comes from `countOf`.** `countOf(t, n, noun)` with a noun from `COUNT_NOUNS`; `pluralize`
    takes words and is internal to it. A hand-written `${n} posts`, an appended `s`, or a count noun glued to a
    number in JSX ships English into eight locales.
27. **A plural is chosen by the language, not by `n === 1`.** Arabic has six plural categories and Russian
    three; the binary singular/plural pair is only correct for the Latin locales.
    `(not yet built)`: needs `Intl.PluralRules(activeLocale())` behind `countOf` and a category suffix per key
    in every locale file, replacing the `count.x` / `count.xs` pair shape.
28. **Relative time is one format.** `formatRelativeTime` in `lib/utils/format/format-date.ts` is the only
    source of a timestamp on a card, a row or a bubble; a surface never writes its own.
29. **A date is formatted through the date helpers**, each of which passes `activeLocale()`:
    `formatFullDate`, `formatMonthYear`, `formatChartDate`. A hard-coded `'en-US'` or
    `'en-GB'` is a defect, as is an options object written at a call site.
30. **A bare `YYYY-MM-DD` is a calendar day, not an instant.** `toLocalDate` reads it at local midnight,
    because `new Date('2026-09-09')` is UTC midnight and renders as the day before anywhere west of UTC.
31. **A clock reads 12- or 24-hour by locale.** Formatting goes through `Intl` with the locale's own hour cycle;
    `to12Hour` and `formatTime12` in `lib/utils/format/format-time.ts` hard-code AM/PM and are for the pickers
    that edit an explicitly 12-hour value, never for displaying a time.
    `(not yet built)`: needs the display path moved onto `toLocaleTimeString(activeLocale())`.
32. **A week starts where the locale starts it.** A calendar grid's first column is the locale's first weekday,
    not Sunday, and its column headers come from `Intl` in that order.
    `(not yet built)`: needs a `firstWeekdayOf(locale)` helper (`Intl.Locale.prototype.getWeekInfo` where
    available, a table otherwise) read by every calendar grid.
33. **A duration is a length, not a clock read.** `formatDuration` and `formatElapsed` take milliseconds and are
    safe during render; `formatRelativeTime` and `formatTimeLeft` read the clock and are not.

### Composition and expansion

34. **A sentence is one key.** Never two keys concatenated, never a key plus a literal, never a number rendered
    beside a translated noun in JSX; word order is part of a language, and a sentence assembled in JSX is
    assembled in English word order for all nine.
35. **A value that varies is an interpolation.** `t('space.memberCount', { count })`, not
    `{count} {t('ui.members')}`.
36. **A label survives twice its English width.** Measured against the shipped catalogue, short labels expand at
    p95 by 2.20x in German, 2.18x in Russian and 1.82x in Arabic, and the worst single label expands 6.33x
    (`ui.pip`: "PiP" to "Картинка в картинке"). A control whose label is a word or two reserves its widest
    label, wraps, or truncates with the full string reachable; it never clips and never pushes its row.
37. **The surfaces that must survive expansion** without a horizontal scroll or a second line on a phone: the
    tab bar and every tab strip, the page header's title and actions, chip strips, menu-sheet rows, button pairs
    in a dialog footer, settings rows, and every count beside a glyph `[layout]`.
38. **CJK contracts; the layout still holds.** Chinese runs at 0.39x of English, Korean 0.55x and Japanese 0.59x,
    so a fixed width tuned for German leaves a Chinese label adrift in its own control; widths are intrinsic or
    they are the widest real label, never a guess.

### Script safety

39. **A normaliser keeps every script the app ships.** `\p{L}\p{N}` with the `u` flag, never `[a-z0-9]` or `\w`:
    a class-based normaliser folds Arabic, Japanese, Korean, Russian and Chinese to an empty string, so search
    returns nothing and a length guard reports a real name as too short.
40. **Normalisation never strips a mark that carries meaning.** NFKD removes the dakuten from Japanese and
    decomposes Hangul; a safety or search normaliser that runs it erases the word it was checking.
41. **A slug built from a non-Latin name is still usable.** `slugifySpaceName` produces a slug of at least three
    characters for a name in any shipped script, or the feature is closed to five locales.
42. **A label never renders as its key.** A missing key falls back to English and then to the key itself; a raw
    key, an `undefined`, or a layout that shifts when the string lands is a defect
    (`a-label-resolves-when-it-renders`).

## Enforced by

- `__tests__/translation/locale-parity.test.ts` covers rules 6, 7, 8. Asserts the roster is exactly the nine codes,
  every non-English file carries the English key set with no extras and no empty values, no full English
  sentence ships verbatim, and every `{param}` survives.
- `__tests__/fitness/user-facing-copy-is-translated.test.ts` covers rules 1, 6, 10, 11, 13. No hardcoded metadata
  title, no `lang="en"` and no missing `dir` on `<html>`, no English in a copy prop, a JSX text child or an
  `aria-label`, no em dash in a locale file, and something on the server reads `preferred_language`.
- `__tests__/translation/i18n-rtl.test.ts` covers rules 11, 12, 15, 19. Classifies every shipped locale by script,
  drives `useLocaleInit` through account, browser and fallback, and signs `useInlineDirection` per locale.
- `__tests__/translation/i18n-labels.test.ts` and `i18n-interpolate.test.ts` cover rules 2, 3, 8, against the real
  shipped corpus rather than a fixture.
- `__tests__/fitness/i18n-runtime-is-not-mocked.test.ts`: the runtime executes in at least one spec, so the
  fallback chain above is actually exercised.
- `__tests__/fitness/text-handling-is-unicode-safe.test.ts` covers rules 39, 41, over `normalizeText`,
  `searchSettings` and `slugifySpaceName`.
- `local/no-physical-inline-axis` covers rule 16, with the media-coordinate exemption of rule 22.
- `local/no-hardcoded-ui-text` covers rule 1, for attribute copy.
- `__tests__/fitness/a-label-resolves-when-it-renders.test.ts` covers rule 42.
- `__tests__/fitness/a-directional-glyph-mirrors.test.ts` covers rules 17, 18. Censuses every caret and arrow
  that points left or right, the bend, U-turn, arc and elbow families and `ICONS.back`, rendered as a tag or
  passed as a value, and fails any not drawn through `DirectionalGlyph` or `directionalIcon`; it ratchets every
  class list holding `origin-left` without `rtl:origin-right`.
- `__tests__/fitness/a-gesture-mirrors-with-the-reader.test.ts` covers rule 21. Ratchets every file that reads a
  touch's `clientX` without reading the inline direction, against
  `__tests__/fitness/baselines/gesture-blind-to-direction.json`.
- `__tests__/ui/user-text-takes-its-own-direction.test.tsx` covers rule 23 for what a person wrote: `RichText`,
  its tokens, `DisplayName` and the link card.
- `__tests__/fitness/a-sentence-is-one-key.test.ts` covers rules 26, 34, 35. Ratchets a JSX child that renders
  a value set beside one that reads a key with only whitespace between them, and an English plural suffix
  written beside a count.
- `__tests__/fitness/a-date-reads-the-reader-s-locale.test.ts` covers rules 25, 29. Ratchets every
  `toLocaleDateString`, `toLocaleTimeString` and `toLocaleString` outside `lib/utils/format/` whose first
  argument is absent, `undefined` or a literal tag, and fails outright when a formatter stops reading
  `activeLocale()`.
- `__tests__/fitness/a-message-group-spreads-its-proxy.test.ts` covers rule 5. Fails a `localized()` call in
  `lib/constants/messages.ts` handed anything but string keys.

**Not yet enforced**

- Rule 23 for a `@handle` in chrome: about 120 sites write `@{username}` by hand with no isolate, so under
  Arabic the `@` lands after the name. Waits on one handle primitive every site draws through.

- Rule 27, `a-plural-comes-from-the-locale`: fails once `Intl.PluralRules` is in place if a key pair is read by
  an `n === 1` ternary.
- Rule 32, `a-week-starts-where-the-locale-starts-it`: fails a `grid-cols-7` calendar whose first cell offset
  is computed from `getDay()` alone. Waits on the first-weekday helper rule 32 asks for: all three grids are
  Sunday-first today, with nowhere else to move to.
- Rules 36, 37, 38, `a-label-survives-its-longest-locale`: renders the listed surfaces at 390px with the
  German and Arabic catalogues and fails on clipping, a second line or horizontal overflow. Width is a browser
  measurement, so this is `e2e/` work `[definition-of-done]`, not a source-level spec.

## Traps

- **1.** The post overflow menu shipped twelve English entries beside four translated ones in one list, and the
  shorts overflow menu read "Media information" in nine languages: a copy prop is not the only way copy reaches
  a reader, so a JSX text child counts too.
- **11.** Production served `lang="en"` with no `dir` to every request including `Accept-Language: ar`, and the
  document flipped after hydration; screen readers and crawlers were told the wrong language.
- **12.** `preferred_language` was written by settings and onboarding and read by exactly one client hook, so no
  email, push notification, page title or server render ever saw it.
- **16.** The shell had `fixed left-0` on the rail while its gutter was reserved with `ms-`; in Arabic the rail
  stayed left, the column reserved right, and the rail sat on top of the feed. A chat bubble's tail was pinned
  to `rounded-br` and pointed away from its sender.
- **26.** `pluralize` takes words, so every caller that passed an English literal printed English into the other
  eight locales.
- **39.** Settings search returned zero results for any query typed in Arabic, Japanese, Korean, Russian or
  Chinese, a Space named in Japanese could not be created and was told its name was "too short", and a hashtag in
  any of those scripts rendered as dead text.
- **3.** A module-level copy of a label freezes on English: the locale JSON loads after mount, so the read has
  to happen inside the render or the call.
- **7.** Key parity passed while 48 full English sentences shipped verbatim to all eight locales, so a German
  reader read English through the whole shorts editor, the story flow and the DM share menu.
