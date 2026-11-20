# Type

Text is most of the product. This document owns the ramp and what each step is for, weight, tracking, how a line
wraps and where it breaks, what truncates and after how many lines, how numbers and emoji behave, which script
the reader is in, and the viewer's own text size. The feeling it protects: a person reads a post, a name, a
count and a timestamp on one card and never once notices the typography, because every decision was already
made and none of them was made twice.

## Owns and defers

Owns: the ramp's assignment, the 14px floor and its one exception, weight, tracking, leading, wrapping,
breaking, truncation, tabular figures, emoji sizing, mixed script, the font registry, and which layouts must
survive the largest text size.

Defers: the token values themselves and the no-arbitrary-value rule `[tokens]`; ink tiers and which tier a step
takes `[colour-and-effects]`; hierarchy depth and how size, weight and ink move together `[craft §8]`; column
measure and line length `[layout]`; locale, pluralisation, date and number formatting `[internationalisation]`;
what the words say `[copy-and-voice]`; contrast and screen-reader text `[accessibility]`; the caret, the
composer and input keyboards `[input-and-forms]`.

## Rules

### The ramp

1. **Three sizes carry the app.** `text-body` for reading, `text-sm` for secondary, `text-lg` or `text-xl` for a
   title. Everything at `2xl` and above is a page title or a display number. A screen that needs a fourth
   reading size has a hierarchy problem, not a type problem.
2. **14px is the floor.** No reading text renders below it at the default text size. `text-badge` at 11px is the
   one exception: a count badge, and the rail's caption under its glyph on a device that cannot hover
   `[layout §25]`, never running text. The `text-2xs` and `text-xs` aliases were deleted
   because both floored to 14 and 2,036 call sites expressed a three-tier hierarchy that rendered as one size;
   a surviving `text-xs` now resolves to Tailwind's 12px and stops tracking the viewer's text size.
3. **A transform is not a size.** `scale-90` on `text-sm` renders 12.6px, and 11.0px for a reader on the small
   text size, without ever naming a size below the floor. So does an inline `fontSize` in px and an SVG
   `fontSize` attribute. Text is sized by a ramp step, or it is not sized.
4. **`text-base` is the input size, not a second body size.** 16px is load-bearing: iOS Safari zooms the
   viewport when a focused input renders below it, so every text input pairs `pointer-coarse:text-base` with its
   resting size `[input-and-forms]`. Body copy is 15.
5. **Weight does the rest.** 400 body, 500 a label or a count, 600 a title or a person's name, 700 only a page
   title or a number meant to be glanced. A fifth weight is a bug. `--font-weight-*` in `app/globals.css` names
   all four.
6. **Tracking lives in the ramp.** Each step above `sm` carries its own `--text-*--letter-spacing`, tightening
   as the size grows. A `tracking-*` utility on a step that already has tracking applies it twice; on any step
   it overrides a decision already made correctly and does not follow the viewer's text size. The exception is
   a code meant to be read aloud a character at a time, where the wide tracking is the point.
7. **Leading is relative.** `leading-none`, `leading-tight`, `leading-snug`, `leading-normal`, `leading-relaxed`.
   A numeric `leading-6` is a frozen 24px line box around text that grows 12.5% at the largest text size, and
   the glyphs then sit off centre in it.
8. **A size is never a colour and a colour is never a size.** Emphasis is weight and ink, not a step up the
   ramp; a quiet thing is muted ink at the same size, not smaller type.

### Wrapping and breaking

9. **Titles balance, body does not.** A short heading or a centred explanatory line takes `text-balance` so its
   last line is not one orphaned word. Running text takes neither `text-balance` nor `text-pretty`: both are
   layout passes over content whose length the app does not control, and `text-balance` gives up past four lines.
   `text-balance` never sits on the same node as a `line-clamp`, because the balance is computed against a box
   the clamp then cuts.
10. **A line breaks between words.** `break-words` is the default for user text. `break-all` is only for a value
    with no words in it: a URL, an email address, a recovery code, an API key. It never touches a sentence, a
    name or a handle, because it breaks inside a grapheme cluster and can split an emoji from its modifier.
11. **`whitespace-nowrap` is for a value that must not split.** A timecode, a count beside its glyph, a
    timestamp, a tab label. On a button label it is a decision to overflow, and a `flex-1` button that cannot
    wrap overflows in German and at the largest text size.
12. **Mixed script and CJK break on their own rules.** Chinese, Japanese and Korean wrap between characters with
    no space, and Arabic and Hebrew wrap right to left; the app sets no `word-break` on body text and lets the
    browser do it. Any normaliser that touches user text uses `\p{L}\p{N}` with the `u` flag, never `[a-z0-9]`,
    which erases five of the nine locales `[internationalisation]`.

### Truncation

13. **A name truncates, a handle truncates, and they stack.** `DisplayName` is one flex row carrying `truncate`
    on the name and `max-w-full`, with the badge after it; a name and its `@handle` sit in a flex column so the
    handle never joins the name's line box. A surface that renders a name or a handle without `DisplayName`
    renders it inside `truncate` and a `min-w-0` parent, or it overflows its row.
14. **A handle is truncated with its `@`.** The sigil is the first character of the truncated node, never
    stripped and never left outside it, or the ellipsised string is indistinguishable from a display name.
15. **One line, or the clamp ladder.** A single-line value uses `truncate`. A multi-line one uses `line-clamp-2`
    for a card preview or a two-line title, `line-clamp-3` for a body excerpt or a sticker, and `line-clamp-4`
    and above only on a reading surface: one with no "more" control, a feed card's own post, or a story's
    shared text post, where the post IS what was shared and the card itself opens the whole of it. A "more"
    control stands on the clamp's last line and never adds one: overflow is measured after the server paints,
    so the feed card drops from `line-clamp-6` to `line-clamp-5` and "Show more" takes the sixth. A clamp
    inside a box with no height ceiling clamps nothing.
16. **A clamped string carries its full text somewhere.** A `title` attribute on a fine pointer, or the row
    opens the thing. A truncation the reader cannot escape is a dead end.
17. **A truncation width is not a pixel.** A `max-w-[90px]` on a name truncates 12.5% earlier at the largest
    text size and 12.5% later at the smallest. Cap by the column, with `min-w-0`, never by a literal.

### Numbers

18. **Every number that changes uses `tabular-nums`.** Like, repost, comment, bookmark and view counts; live
    viewer counts; timers, durations, countdowns, progress readouts, character counters, scores. Proportional
    figures re-measure on every tick and shove the text beside them.
19. **A count's control is as wide as its widest label.** The number crossfades in place `[motion]`; the box
    does not resize under the finger that is still on it.
20. **Numbers are formatted, never concatenated.** `formatCompactNumber`, `formatFullNumber` and `countOf` from
    `lib/utils/format/`; `EMPTY_VALUE` is the em dash a missing number renders as `[internationalisation]`.

### Emoji

21. **One emoji stack, named once** `(not yet built)`. Emoji are content, and the reader's platform font is the
    right face, but the fallback order still has to be stated or a glyph the app font happens to carry renders
    as a monochrome outline. `EMOJI_FONT_STACK` in `components/ui/emoji-picker-grid.tsx` is the closest thing
    to it and is copied by hand into four other render paths with three different tails. It becomes one
    `--font-emoji` token that every emoji surface and the canvas exporter read. (not yet built)
22. **Emoji sit on the text baseline at the text's size.** They are content, not glyphs from the icon set, and
    they never take a size the ramp does not have.
23. **An emoji-only message steps up, in two steps.** `jumboEmojiSize` returns 48px at three glyphs or fewer and
    30px at six or fewer, both steps on the ramp; past six it stays body text.
24. **Length is counted in graphemes.** `countGraphemes` in `lib/utils/emoji.ts` uses `Intl.Segmenter`, so a ZWJ
    family counts as one. A counter, a limit or a `slice` that works in `String.length` charges a reader eleven
    for one family emoji, and a `slice` at a code-unit boundary can leave a lone surrogate in the field.
24a. **The picker offers only what the device draws.** `canDrawEmoji` in `lib/utils/emoji-support.ts` probes the font on a
    canvas and drops a glyph that comes out as the missing box or falls apart into its parts, so an older Android
    never shows a person a square to send.

### The viewer's text size

25. **`--font-scale` multiplies the whole ramp.** The viewer picks small, default or large in
    `components/settings/display-settings.tsx`; `html[data-font-size]` sets `--font-scale` to 0.875, 1 or 1.125
    and the root `font-size` to 14, 16 or 18px, so rem-based chrome and px-based type move together. Nothing
    branches on the setting. Until the viewer picks a size, a shell's OS text scale chooses it by `TEXT_SCALE_STEPS` (up to 0.9 small,
    1.1 default, 1.2 large, 1.45 larger, 1.75 largest, past that maximum) and follows the OS while the app runs,
    through `effectiveFontSize`. The three steps above large set `--font-scale` to 1.3, 1.5 and 1.75 and leave the
    root at 18px, so the rem chrome (header rows, bars, pills) holds its large geometry and only content grows; a
    token of 24px and up reads `--font-scale-display`, which stops at 1.3. The picker still offers three sizes and
    shows large for a step above it.
26. **The scale lands before first paint.** `FONT_SIZE_ATTR` is stamped on `<html>` by the boot script, not by
    an effect, or every label jumps once on hydration `[arrival]`.
27. **The spacing scale does not move, so every fixed box around text is checked at 1.125 and, on a 360-wide
    phone, at 1.75.** These must not clip
    or overflow: a settings row, a menu row, a button label, a tab strip and its scroll reserve, a count badge,
    a name and handle pair, an on-media author pill, a reaction chip, the composer's counter, a numeral badge,
    a dialog's title and its actions, and every skeleton that reserves a line box.
28. **A preview of a text size is in px.** The picker renders its three options at `--root-font-small`,
    `--root-font-default` and `--root-font-large`; a rem preview rescales with the setting it previews and the
    three options look identical.

### The font registry

29. **One family per theme.** `lib/fonts.ts` is the registry and `--theme-font-sans` is the only place a theme
    names a face. Six themes are Inter; aero is Open Sans, because its original stack named five faces that ship
    on Windows and nowhere else and every phone fell through to Inter.
    Every stack that names Inter names `--font-inter-cyrillic` straight after it, so a Russian line never falls to
    the system face mid-word.
30. **A face is self-hosted and non-blocking.** `next/font/google`, Latin subset, `display: swap`, a real system
    fallback behind it, and `preload: false` on everything but the default family.
31. **The editor's faces are content, not chrome.** `FONT_FAMILIES` in `lib/constants/colors.ts` is the creator's
    palette for text on media, reached through a CSS variable so a chip, the DOM and the canvas exporter render
    the same face. No app surface picks from it.
32. **A family never comes from a data field.** A face a reader can set on their own profile is chosen from
    `FONT_FAMILIES` by id, never stored and applied as a raw `fontFamily` string, or the app renders a family
    it has never seen on a surface everyone else reads.

### Authored text

33. **Authored text is drawn by `RichText`.** A post, a reply, a quote, a caption, a message and every excerpt
    of one carry Surf's markdown, and drawn raw they show the reader its asterisks, underscores, backticks and
    spoilers. `RichText` is the one renderer, and `parseRichText` in `lib/domain/content/rich-text.ts` is the one
    parse behind it and behind every plain projection.
34. **The variant is the context, not a taste.** The full variant where the text is read in place. The preview
    variant for an excerpt inside a link, a button or a clamped row: formatting kept, and no link, control or
    block of its own, so it nests and clamps. The verbatim variant where the source is the point: an edit
    record. The field variant only under a textarea's caret: the source as the field holds it, a link in ink.
35. **A string gets the plain projection.** An attribute, a title, a meta description, a social card, a push,
    an email, a one-line search snippet and a share to another app take `richTextToLine` or `richTextToPlain`,
    never the source. A spoiler becomes `SPOILER_PLAIN_MASK` there, because a string has no reveal.
36. **A block is phrasing content.** A code block and a quote render as a code element and a span drawn as
    blocks, because post text sits in a paragraph and the HTML parser closes a paragraph at a block element, so
    the server and the client would build different trees and hydration would throw one away.

## The ramp

Values are `[tokens]`; this is the assignment. Every step multiplies by `--font-scale`.

| Step | px | Line height | Tracking | Where it is used |
|---|---|---|---|---|
| `text-badge` | 11 | 1 | none | A count badge. Never a word |
| `text-sm` | 14 | 20 | none | Secondary text, meta, row labels, timestamps, names in a list |
| `text-body` | 15 | 22 | −0.005em | Reading text: a post, a message, a paragraph, a menu row |
| `text-base` | 16 | 24 | none | Text inputs, on touch and on desk |
| `text-lg` | 18 | 28 | −0.01em | A page or dialog title |
| `text-xl` | 20 | 28 | −0.01em | A page title with room for it |
| `text-2xl` | 24 | 32 | −0.02em | A hero title |
| `text-3xl` | 30 | 36 | −0.02em | A display number; the second jumbo-emoji step |
| `text-4xl` | 36 | 40 | −0.02em | Display |
| `text-5xl` | 48 | 1 | −0.02em | One glanceable figure; the first jumbo-emoji step |
| `text-7xl` | 72 | 1 | −0.02em | The largest display, one per screen |

`--text-5xl` and `--text-7xl` are curated because both were already in use and were resolving to the framework ramp, which
carries neither the tracking curve nor the viewer's text size. 6xl, 8xl and 9xl are not tokens: a class
naming one of them resolves to Tailwind's default and is frozen against the text size.

## The clamp ladder

| Clamp | Surface |
|---|---|
| `truncate` | A name, a handle, a row label, a filename, a single-line title |
| `line-clamp-2` | A card preview, a two-line title, a notification body, a search result, a toast description |
| `line-clamp-3` | A body excerpt, a quoted post, a story sticker, a poll option |
| `line-clamp-4` and above | A reading surface: no "more" control, a feed card's own post, whose "Show more" takes the clamp's last line, or a story's shared text post, whose card opens the whole post |

## The font registry

| Variable | Family | Preloaded | Used by |
|---|---|---|---|
| `--font-inter` | Inter | yes | light, dark, glass, retro, celestial, glow, sapphire; the editor's "Modern" |
| `--font-inter-cyrillic` | Inter, Cyrillic subset | no | after `--font-inter` in every stack |
| `--font-humanist` | Open Sans | no | aero, through `--theme-font-sans` |
| `--font-round` | Poppins | no | Editor: "Round" |
| `--font-serif-display` | Playfair Display | no | Editor: "Classic" |
| `--font-anton` | Anton | no | Editor: "Strong"; `DISPLAY_FONT_STACK`; caption presets |
| `--font-bebas` | Bebas Neue | no | Editor: "Heavy" |
| `--font-typewriter` | Space Mono | no | Editor: "Type" |
| `--font-marker` | Caveat | no | Editor: "Hand" |
| `--font-script` | Dancing Script | no | Editor: "Script" |

`--font-mono` is `ui-monospace` first and no theme overrides it. `APP_FONT_VARIABLE_CLASS` puts every variable
on `<html>`; `FONT_VAR_FAMILIES` resolves eight of them for the canvas exporter, and omits `--font-humanist`
because a theme's face never enters an export.

## Enforced by

- `__tests__/fitness/a-text-size-is-not-a-colour.test.ts`: 8.
- `__tests__/fitness/a-name-and-its-handle-stack.test.ts`: 13.
- `__tests__/fitness/arbitrary-value-duplicates-a-token.test.ts`: 1, 2, 6, 17, where a size, a tracking value or
  a text width is written as an arbitrary class.
- `__tests__/fitness/text-handling-is-unicode-safe.test.ts`: 12.
- `__tests__/fitness/a-label-resolves-when-it-renders.test.ts`: 27, for the shift a late string causes.
- `__tests__/fitness/a-size-comes-from-the-ramp.test.ts`: 2, 3. The ramp is read out of `app/globals.css`, so a
  step added there is enforced the same hour.
- `__tests__/fitness/tracking-comes-from-the-ramp.test.ts`: 6. The one exception is a widening on a code read
  aloud a character at a time; the spec names those four surfaces and allows nothing else.
- `__tests__/fitness/a-line-box-is-relative.test.ts`: 7.
- `__tests__/fitness/a-sentence-does-not-break-all.test.ts`: 10.
- `__tests__/fitness/a-handle-truncates-with-its-sigil.test.ts`: 13, 14.
- `__tests__/fitness/a-changing-number-is-tabular.test.ts`: 18, for a count or a formatted number drawn as an
  element's own text.
- `__tests__/components/post/feed-list-item.test.tsx`: 15, for the feed card laying out as many body lines once
  its overflow is measured as it did before.
- `__tests__/fitness/a-limit-counts-graphemes.test.ts`: 24.
- `__tests__/fitness/one-family-per-theme.test.ts`: 29, 32.
- `__tests__/fitness/authored-text-is-drawn-by-the-renderer.test.ts`: 33, 35, for an authored field drawn as an
  element's own text or read out of an attribute.
- `__tests__/ui/rich-text-variants.test.tsx`: 34, 36, including a server render hydrated in place.
- `__tests__/security/user-text-and-link-previews-never-become-markup.test.tsx`: 33, every variant against every
  payload a prober has posted.

**Not yet enforced.**

- `one-emoji-stack`: 21. Fails on any emoji font stack string that is not the single token. Waits on the token
  rule 21 asks for `(not yet built)`.
- `the-largest-text-size-still-fits`: 27. A `ui:probe` pass at `data-font-size="large"` that reports any element
  whose scroll height exceeds its client height. It belongs in `e2e/`: jsdom computes no layout, so no fitness
  spec can measure a box `[enforcement §2]`.

## Traps

- **2** `cn()` drops a custom `text-*` size it was never told about, so the count badge silently rendered at the
  inherited size instead of 11px. Use the token; never a one-off.
- **6** A `tracking-*` set at a call site does not scale with `--font-scale`, so it tightens as the reader's text
  grows and the largest step reads cramped.
- **13** `display: inline-flex` on a name puts the `@handle` in the same line box with no gap; the name is a flex
  row and the pair is a flex column.
- **15** A clamp inside a box with no height ceiling clamps nothing: the box grows to the content and the
  ellipsis never appears.
- **15** "Show more" as a row of its own appeared only once hydration had measured the clamp, so it grew every
  clipped card the server painted and pushed every card below it down: the `/home` card shift.
- **25** `--font-scale` moves the px ramp and the root `font-size` moves the rem chrome. Setting only one leaves
  the header taller and the text unchanged, or the reverse.
- **27** A line ghost sits in the real text's line box, and the line box moves with the text size; a ghost
  measured at the default step shifts the page for a reader on the largest one `[loading-and-states]`.
- **29** An OS font stack is not a font choice. Aero's original `Segoe UI, Frutiger, …` shipped on none of macOS,
  iOS or Android, so every mobile reader silently got Inter and the theme had no face of its own.
- **34** An anchor inside an anchor is split by the HTML parser. A mention in a quoted post sat inside the quote's
  own link, and the browser closed the outer link at the inner one; an excerpt inside a link is a preview.
- **36** A code block drawn as a pre element inside the post's paragraph closed the paragraph in the HTML parser,
  so the server's tree and the client's disagreed for any post carrying one.
