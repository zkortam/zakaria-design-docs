# Principles

These are the root commitments the rest of this directory expands. Each one explains why a family of rules
exists; none of them is a rule, and none of them can be checked against a screenshot. When a rule and a
principle disagree, the principle is what the rule was trying to say, and the rule is wrong. When a situation
has no rule yet, the principle decides, and the rule is written in the same commit. Read this page first and
then the document that owns the change.

## Owns and defers

Owns: the reasons. Why the product is standardized, why it moves the way it moves, why a state is designed
before it is reached, why a person is never left wondering what happened.

Defers: every rule. Each principle below names the document that holds its rules, and that document is where
a number, a primitive, a behaviour or a prohibition lives.

## The principles

1. **The person's time is the unit of quality.** Quality is measured in the seconds they wait, the decisions
   they are forced to make and the moments they wonder what happened, not in pixels alone; a screen that is
   beautiful and slow has failed. `[speed]`, `[loading-and-states]`

2. **Everything earns its place, and the person's content is the only loud thing.** The interface is water around a
   person's photos, words and people; chrome is quiet, restraint beats decoration, and anything that could be
   removed without loss is removed. `[craft]`, `[colour-and-effects]`

3. **One way to do each thing.** One primitive per concept, one token per value, one recipe per repeated class
   string, one motion ladder; a second shape for a concept that already has one is drift, and drift is what
   makes a product feel assembled instead of designed. `[components]`, `[tokens]`

4. **Everything flows into the thing beside it.** No state cuts to the next: a surface arrives from where it
   belongs and leaves back toward it, so a person always knows what came from what. `[motion]`

5. **A tap is answered in the frame it lands.** Intent is acknowledged before the network is consulted, the
   result is optimistic, and a refusal rolls back visibly and says what refused. `[feedback]`, `[speed]`

6. **Nothing typed is ever lost.** A draft survives navigation, refresh, a backgrounded tab and a crash; the
   product is responsible for a person's words from the first keystroke, not from the moment they press send.
   `[resilience]`

7. **Undo over confirmation.** A reversible act happens immediately and offers a way back for a named window;
   a dialog that asks "are you sure" about something we could simply undo spends the person's attention to
   save ourselves the work. Only the irreversible is confirmed. `[feedback]`

8. **Absence is not failure.** A thing that does not exist, a list with nothing in it and a request that was
   refused are three different states with three different words, and they never borrow each other's tone.
   `[loading-and-states]`

9. **Every gate has a way through.** A lock, a limit, a paywall or a denied permission names itself and offers
   the one action that opens it; a dead end is a defect, not a state. `[permissions-and-trust]`

10. **The product degrades and never disappears.** On a slow device, a poor connection, a conserved-data
    setting or an old phone, effects are shed and budgets shrink, but every feature still works.
    `[speed]`, `[resilience]`

11. **Who can see this is answered before it is asked.** Audience, visibility and encryption are stated where
    the person is acting, in their words, not discovered afterwards in a settings page. `[permissions-and-trust]`

12. **Defaults so good that settings are unnecessary.** A setting exists only where two reasonable people
    genuinely differ; everything else is a decision we made so they do not have to. `[input-and-forms]`

13. **Every state has a URL, and Back leaves one thing.** What is on screen is addressable, refreshable and
    shareable, and one press of Back or Escape undoes exactly one layer of it. `[navigation-and-history]`

14. **Copy says it once, in one voice.** No sentence restates its label, no screen introduces itself, and
    every string reads like the string beside it. `[copy-and-voice]`

15. **Accessibility is the design, not a pass over it.** A name, a role, a hit area, a focus treatment and a
    contrast ratio are part of what a control is, decided when it is built and never retrofitted.
    `[accessibility]`

16. **Every theme and every language is a first-class citizen.** Eight themes and nine locales are the
    product, not variants of it; a screen that only reads well in light English is unfinished.
    `[colour-and-effects]`, `[internationalisation]`

17. **Nothing built for a test reaches a person.** Seed accounts, bot content and fixture data are excluded at
    the source, in every list and every feed, not filtered at the last surface that remembered.
    `[definition-of-done]`

18. **Whatever becomes something else travels there.** A photo becomes the bubble, a card becomes its page, a
    ring becomes the story, a state that is still happening lives in one capsule: one element, one identity,
    one spring, so the person never sees a thing vanish and another appear. `[motion]`
19. **The browser is the proof.** A change is not done because it compiles or because a spec is green; it is
    done when it was seen on a phone and a desktop, in light and dark, driven the way a person drives it.
    `[definition-of-done]`
20. **Minimal is designed, never stripped.** Many values are one object the person moves directly, the way the
    feed's feel pad holds in one draggable point what could have been ten sliders. A wall of rows, switches or
    prose is the lazy answer and deleting the capability is the bare one; the design is the arrangement where
    nothing is missing and nothing more could go. Quiet is never static `[motion]`. `[craft]`
21. **One door per action.** Two controls in one view that do the same thing make the person choose between
    identical doors, and the second is clutter the first already paid for. `[craft]`

## Enforced by

Nothing here. A principle is not testable, which is the point of the distinction: the topical documents carry
every rule and every test, and `enforcement.md` says how a rule and its test arrive together.
`design-docs-cite-what-exists` holds the citations on this page to documents that exist.

## Traps

A principle has no trap. A trap is the mechanism behind a broken rule, so it lives with the rule, in that
rule's document or in `traps.md`.
