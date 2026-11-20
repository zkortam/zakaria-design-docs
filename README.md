# Surf Design Standard

> Public copy of the Surf design standard. File paths, component names and test names refer to the Surf
> codebase, which is private. `permissions-and-trust.md` and the security internals of a few other documents
> are left out of this copy.

This directory is the binding contract for everything a person sees, touches, hears or waits on. Open the
document for the system you are changing before you change it, and hold the change to every rule there.
`CodeCheckList.md` governs the code; this governs the experience. `definition-of-done.md` is what "done" means
for anything visual, and `enforcement.md` is how the fitness suite in `__tests__/fitness/` compiles these rules
into tests.

## The standard we are held to

Surf feels the way the best Apple software feels: calm, continuous, premium, and so coherent that no single
screen draws attention to itself. Every surface is obviously the same product. Nothing snaps, flashes, jumps,
stutters or glitches. Nothing is rough. Nothing is decoration. A person never notices the interface; they notice
their content and their people, and the interface is water around it.

That standard is measured in the person's time and attention, not in pixels alone: every second they wait, every
decision they are forced to make, every moment they wonder "what just happened", "who can see this", "did that go
through". The product is a sequence, not a screen: arrive, sign in, the first ten seconds, type, wait, get
interrupted, lose signal, come back three hours later, switch devices, make a mistake, undo it, leave. Each
document below owns part of that sequence.

Minimal here is designed, never stripped. Clean, polished and quiet, but never bare, boring or static: when a
surface has many values, the answer is one object the person moves directly, the way the feed's feel pad holds in
one draggable point what could have been ten sliders. Nothing lazy survives review: no wall of rows, switches or
prose standing in for a design; no second button beside the first doing the same thing; no accent ink on words the
product wrote; no stacked edges, excess pills or clutter; nothing inconsistent with the screen beside it; no state
that changes without a transition. Every one of these is a rule with a test, named in the documents below.

The three tests for any pixel, motion, sound or word:

1. **Does it earn its place?** If removing it loses nothing, remove it.
2. **Is it the one way we do this?** If a second shape for the same concept exists anywhere, one of them is wrong.
3. **Does it flow?** Every state arrives from, and leaves into, the state beside it without a cut.

## Which document to open

| You are changing | Open |
|---|---|
| Anything at all | `principles.md`, then the row below that matches |
| A token, a value, a duration, an easing, a z-index, a size | `tokens.md` |
| A page's shape, a column, a gutter, the header, safe areas, a breakpoint | `layout.md` |
| A component, an overlay, a primitive, an icon | `components.md` |
| Spacing, hierarchy, radius, glyph alignment, a surface, a control's look | `craft.md` |
| Text: size, weight, wrapping, truncation, emoji, text scale | `type.md` |
| Colour, ink, accent, a theme, a blur, a shadow, a sheen, a scrim | `colour-and-effects.md` |
| A transition, an animation, a gesture, a spring, reduced motion | `motion.md` |
| A morph, a zoom, a fling, a roll, the live activity capsule, the AI listening edge | `signature-motion.md` |
| A skeleton, an empty state, an error, a not-found, any state the person can land in | `loading-and-states.md` |
| Offline, reconnection, background, drafts, session, multi-device | `resilience.md` |
| Boot, navigation cost, data fetching, budgets, device tiers, measurement | `speed.md` |
| A field, a keyboard, a form, validation, paste, the composer, a setting | `input-and-forms.md` |
| Press, haptics, sound, a toast, a badge, undo, confirm, an interruption | `feedback.md` |
| A URL, Back, Escape, a tab, scroll position, a deep link, a shortcut | `navigation-and-history.md` |
| First paint, install, sign-in, sign-up, the first session | `arrival.md` |
| Cursor, scrollbar, selection, long-press, landscape, window width, share | `platform.md` |
| A permission prompt, audience, privacy, a paywall, deletion, trust | `permissions-and-trust.md` (withheld from this public copy) |
| An image, a video, audio, capture, playback, picture-in-picture | `media.md` |
| A name, a role, a landmark, focus, contrast, a live announcement | `accessibility.md` |
| A string in another language, RTL, a date, a number, a plural | `internationalisation.md` |
| Anything AI: its mark, an ask, a wait, a proposal, a source, a refusal, a conversation | `ai.md` |
| A native shell, a bridge method, a store artifact, the system call screen, a system permission dialog | `shells.md` |
| Any user-facing string | `copy-and-voice.md` |
| Shipping the change | `definition-of-done.md` |
| Adding or removing a rule or a test | `enforcement.md` |
| Something that once broke on production and must not again | `traps.md` |
| Something that broke the build, the harness or the deploy | `docs/engineering-traps.md` |

## One rule, one home

Each rule lives in exactly one document. A document that needs a rule owned elsewhere cites it as `[motion §12]`
and does not restate it. This index is the only page that repeats anything. When a rule is found in two places,
one of them is deleted in the same commit. When a primitive, token or test named in a document changes, the
document changes in the same commit; `design-docs-cite-what-exists` fails the suite when it does not.
