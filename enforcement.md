# Enforcement

This document owns how the rules in this directory are held to. A design contract that lives only in prose is a
contract nobody reads twice: it drifts the first week someone is in a hurry, and the drift is invisible until a
person meets it. The fitness suite is these documents compiled into checks that run before a push, so the rule
and the code cannot disagree for longer than one commit.

## Owns and defers

Owns: which tests are the design subset and what each compiles; the ratchet model and how a baseline behaves;
how a rule and its test land together; how a rule is deleted; how the one-home rule is enforced; the citation
test that keeps every identifier in these documents real.

Defers: what any individual rule says to the document that owns it; the browser evidence a change needs to
`[definition-of-done]`; the mechanism behind a past failure to `[traps]` and to `docs/engineering-traps.md`;
which suite can see what to `docs/TESTING.md`.

## Rules

### The suite

1. **The design subset is the compiled form.** Part of `__tests__/fitness/` compiles rules from these
   documents; the rest guard migrations, security, billing, the API surface and the build machine, and belong
   to `CodeCheckList.md`. How many specs exist is the tree's to say, not this document's (rule 3); the table
   below is the whole design mapping, one row per test.
2. **A fitness test reads source, never a browser.** jsdom computes no layout, so a rule about a pixel, a
   duration, a colour or a hit area cannot be checked here. Those rules are held by `e2e/`, listed in
   `[definition-of-done]`, and a fitness test that pretends otherwise is a test that cannot fail.
3. **Every push runs them.** `.githooks/pre-push` runs `npm run test:fitness` on every push, whatever the change
   touched, because these specs read the repo with the file system instead of importing it and so nothing links
   them to the code they guard. `npm run test:fitness:list` is the one definition of the set and derives it from
   the tree, so a new spec is in the gate the moment it is written.
4. **A test is named as a claim.** Lower case, hyphenated, a sentence that is true while the suite is green:
   `a-close-is-one-control`, `an-error-names-what-failed`. The file is that name plus the spec suffix. A name
   that describes the mechanism rather than the promise dates the moment the mechanism changes.
5. **A spec asserts that it swept.** Every sweep asserts its file set is non-empty before asserting over it. A
   glob that matches nothing passes silently, and the git index does not expand braces, so one brace pattern is
   enough to make a whole rule vacuous.
6. **Tracked files only.** `gitFiles` reads the git index, so an agent's scratch copy in a worktree is never
   what a rule is measured against.
7. **Internal surfaces are out of scope.** `NOT_READER_FACING` excludes staff tooling, the mockup studio, the
   admin app and API handlers from the reader-facing rules. A rule that must also hold there says so by reading
   `NOT_A_MEMBER_SURFACE`, which exempts only what nobody opens: `controls-are-operable` does, because a
   moderator taking an irreversible action needs the control named as much as a member does.

### The ratchet

8. **A ratchet records the debt that predates its rule.** `ratchet` in `__tests__/fitness/design-baseline.ts`
   fails on any offender the committed baseline in `__tests__/fitness/baselines/` does not already carry, so a
   new offence is red on the commit that writes it while the existing ones stay visible and countable.
9. **A baseline prunes only.** Regeneration drops entries that are fixed and never absorbs a new one. The file
   can shrink and can never grow to make a red run green, which is the only property that makes a ratchet worth
   having.
10. **A baseline is regenerated, never edited.** By its own spec, one worker at a time, from a clean tree:

    ```
    UPDATE_BASELINE=1 NODE_ENV=test npx vitest run __tests__/fitness/<spec>.test.ts --maxWorkers=1
    ```

11. **A ratchet's message names the fix.** The failure text says what the rule is, lists the new offenders, and
    points at the primitive or the shared home that already solves it. A reader who has never opened the spec
    fixes the code from the failure alone.

### Adding, deleting, and one home

12. **A rule and its test land in one commit.** A rule added here without its test is unenforced from the moment
    it is written; a test added without its rule is a check nobody can appeal. When the rule cannot be tested at
    source level, the same commit adds the browser check in `e2e/` instead.
13. **A rule with no test names the test it wants.** It goes under **Not yet enforced** in its document with a
    name in the house style and one line saying what the test would read. That line is the specification the
    test is written from.
14. **Deleting a test deletes its rule.** Or replaces the test in the same commit. A rule whose test was removed
    is a sentence with nothing behind it, and it will be found untrue later by a person rather than by a run.
15. **Renaming a primitive edits its document.** In the same commit.
    `design-docs-cite-what-exists` fails the suite when a document names a file, export, token, utility or test
    that the repo does not have, so a rename cannot quietly leave the contract describing a symbol that is gone.
16. **One rule, one home.** A rule lives in exactly one document; a document that needs a rule owned elsewhere
    cites it as `[motion §12]` and does not restate it. When the same rule is found in two documents, the one
    that does not own the concern is deleted in the same commit, and `README.md` decides which that is. The
    index is the only page allowed to repeat anything.
17. **A phase-two violation is recorded, not excused.** Code that breaks a rule written after it goes into the
    rule's baseline, not into a carve-out inside the rule. A carve-out is permanent and invisible; a baseline
    entry is a debt with a number on it.

### The citation test

18. **Every backticked identifier resolves.** `design-docs-cite-what-exists` reads every document in
    `design-docs/` and `docs/engineering-traps.md`, extracts every token in backticks outside a fenced block,
    and fails unless it is one of: a path that exists in the repo, a route path in `lib/constants/routes.ts`, a
    fitness test name, a custom property declared in `app/globals.css` or `styles/`, a class or utility declared
    there, a Tailwind utility backed by a declared token, an export in `lib/`, `components/` or `app/`, a script
    in `package.json`, or a cross-reference to a document in this directory.
19. **The two allow-lists stay small.** HTML and ARIA attributes and CSS properties are real things a rule must
    sometimes name, so each is matched against a named list inside the spec. Adding to either list is a
    decision, taken in the spec, not a way past a failure.
20. **A rule for something that does not exist is marked.** A rule the code cannot yet satisfy is written anyway
    and carries `(not yet built)` on its line, naming what would have to exist. The citation test skips tokens
    on a marked line, so an unbuilt symbol can be named exactly once, in the rule that asks for it.
21. **A proposed test name is exempt only where it is proposed.** Under **Not yet enforced**, a hyphenated name
    is a specification for a test that has not been written. Anywhere else, a name in that shape must be a spec
    on disk.
22. **A §-reference names a rule the document still numbers.** `[motion §12]` claims that document numbers a
    rule 12; a renumbering edits every citing document in the same commit, or the citation test fails on the
    number the owning document no longer carries.

## The design subset

Each row is one spec and the document whose rules it compiles. The rule numbers are held by the owning
document; this table is the index from test to owner.

| Test | Document | What it holds |
|---|---|---|
| `a-baseline-only-shrinks` | `[enforcement]` | A committed baseline never gains a line |
| `a-close-is-one-control` | `[components]` | One close control, one size per context |
| `a-document-follows-the-skeleton` | `[enforcement]` | Every document is read in one shape, and claims only tests that exist |
| `a-field-shows-focus-once` | `[components]` | One focus treatment per field, never two rings |
| `a-ghost-is-written-once` | `[loading-and-states]` | A route ghost has one definition, so it cannot drift from itself |
| `a-handle-keeps-its-order` | `[internationalisation]` | Every @handle is drawn through `Handle`, isolated left to right |
| `a-header-control-is-one-size` | `[craft]` | Every header control is one box and one glyph at rest weight |
| `a-label-resolves-when-it-renders` | `[internationalisation]` | No key name, no undefined, no shift when a string lands |
| `a-lazy-overlay-waits-for-its-first-open` | `[components]` | A lazy overlay mounts nothing until it is opened |
| `a-lazy-sheet-waits-to-be-opened` | `[components]` | The same, for sheets that render a fallback while closed |
| `a-media-frame-keeps-its-shape` | `[media]` | A short, a story and the camera stage keep 9:16 on every window |
| `a-meta-line-is-parted-by-space` | `[craft]` | Facts on one line are parted by space; no dot, bullet or pipe between them |
| `a-name-and-its-handle-stack` | `[components]` | The person's name and handle are one primitive, stacked |
| `a-page-has-one-row-of-choices` | `[components]` | Sections are tabs, refinements sit behind one filter, pills are never chrome |
| `a-people-list-excludes-the-test-accounts` | `[copy-and-voice]` | Nothing from the test suite reaches a person |
| `a-pinned-tag-lands-inside-its-image` | `[media]` | A tag's coordinates are bounded by the image that carries them |
| `a-press-reads-one-ladder` | `[feedback]` | One press ladder, no hand-written scale |
| `a-recipe-lives-in-styles` | `[tokens]` | One home for a shared recipe; no per-feature design system |
| `a-route-skeleton-must-server-render` | `[loading-and-states]` | A route ghost arrives in the HTML, not after hydration |
| `a-rule-has-one-home` | `[enforcement]` | No rule is opened by two documents |
| `a-side-inset-is-logical` | `[layout]` | A side inset follows the writing direction |
| `a-stateful-input-is-mounted-once` | `[layout]` | A composer is mounted once and moved by layout, never twinned |
| `a-surface-is-not-a-wall` | `[craft]` | A crowd of switches or sliders is one object, one ladder or a default |
| `a-surface-shows-one-state-at-a-time` | `[loading-and-states]` | Loading, empty, error and content never share a screen |
| `a-tab-lives-where-its-strip-does` | `[navigation-and-history]` | The URL controls the tab, so a deep link survives |
| `a-tap-target-is-a-control-not-a-wrapper` | `[accessibility]` | The hit area belongs to the control, never its container |
| `a-text-size-is-not-a-colour` | `[craft]` | Size and ink are separate axes of hierarchy |
| `a-way-in-survives-a-phone` | `[layout]` | The way into a feature is reachable below the desktop breakpoint |
| `accent-never-colours-a-word` | `[colour-and-effects]` | Words the product wrote are never accent ink |
| `an-edge-element-pays-its-inset` | `[layout]` | An element fixed to a screen edge pays its inset or bleeds as background |
| `an-edge-is-drawn-once` | `[craft]` | One edge at rest, never a ring beside a border |
| `an-empty-state-is-the-primitive` | `[loading-and-states]` | One empty state, not a grey sentence per screen |
| `an-error-names-what-failed` | `[copy-and-voice]` | The error says what refused, and can be quoted |
| `an-inset-holds-at-every-width` | `[layout]` | A wider rung changes the inset's step, never drops it |
| `an-overlay-inherits-its-behaviour` | `[components]` | Focus trap, Escape, scrim, scroll lock and history come from the primitive |
| `arbitrary-value-duplicates-a-token` | `[tokens]` | No arbitrary value where a rung already holds the number |
| `ai-speaks-one-language` | `[ai]` | One AI mark per meaning, one way to wait on AI, one badge for what AI made |
| `aria-props-use-the-standard-name` | `[accessibility]` | A prop a caller can reach, spelled the way the platform spells it |
| `chrome-never-prefetches-on-sight` | `[speed]` | Chrome warms on intent; a bare `next/link` prefetches on sight |
| `controls-are-operable` | `[accessibility]` | Every control has a name, a role and a hit area |
| `copy-reads-as-one-voice` | `[copy-and-voice]` | One voice across the key set, ratcheted on length, exclamation and synonym |
| `every-control-has-a-test` | `[enforcement]` | A control no spec presses is a control first pressed in production |
| `every-css-var-resolves` | `[tokens]` | A property is declared where it is read, or the whole declaration is dropped |
| `every-feature-has-a-way-in` | `[layout]` | A finished feature nothing links to is indistinguishable from one never built |
| `framer-ships-once` | `[motion]` | The animation features ship once, for the whole app |
| `history-state-has-one-owner` | `[navigation-and-history]` | One writer per history entry, so the URL and the screen agree |
| `invalidation-reaches-a-reader` | `[speed]` | An invalidation aimed at a key nothing reads refreshes nothing |
| `nothing-fails-silently` | `[feedback]` | An action that fails says so |
| `nothing-outranks-the-gate` | `[tokens]` | The layering contract: nothing the app renders covers the gate or the toast |
| `one-capsule-per-frame` | `[signature-motion]` | One live activity capsule in the app frame, fed through its two doors |
| `one-door-per-action` | `[craft]` | An empty state never repeats its header's action |
| `route-javascript-stays-under-budget` | `[speed]` | The shared shell and per-route budgets, read from the build's manifests |
| `the-keyboard-is-read-once` | `[input-and-forms]` | One listener reads the visible viewport and the keyboard |
| `the-size-class-is-one-variant` | `[layout]` | Which design a window gets is its size class, asked one way |
| `the-settings-ghost-names-the-page-it-stands-in-for` | `[loading-and-states]` | A ghost that draws a real title says what the page will say |
| `user-facing-copy-is-translated` | `[internationalisation]` | Every string a person reads goes through the key set |
| `design-docs-cite-what-exists` | `[enforcement]` | Every identifier these documents name exists in the repo |

The other twenty-eight specs in `__tests__/fitness/` are the code contract, not this one: migration uniqueness
and ownership, RLS and guardian floors, CSRF adoption, API envelope and route bounds, query-option and
invalidation hazards, the coverage and comment ratchets, the release gate, the studio and proxy contracts, and
`vercel-build-stays-on-the-free-machine`. They are governed by `CodeCheckList.md`.

## Enforced by

- `__tests__/fitness/design-docs-cite-what-exists.test.ts` — rules 15, 18, 19, 20, 21, 22.
- `__tests__/fitness/every-control-has-a-test.test.ts` — rule 12, from the other side: it fails when a control
  ships with no spec that presses it.
- `.githooks/pre-push` — rule 3.
- `__tests__/fitness/design-baseline.ts` — rules 5, 6, 7, 8, 9, 10, 11, shared by every ratchet.
- `__tests__/fitness/a-rule-has-one-home.test.ts` — rule 16. Groups every numbered rule in these documents by
  its bold lead and ratchets each lead two documents share; `README.md` is exempt, being the index.
- `__tests__/fitness/a-document-follows-the-skeleton.test.ts` — rules 12, 13, 14. One ratchet on a document
  missing **Owns and defers**, **Rules**, **Enforced by** or **Traps**, or carrying them out of order; one on an
  **Enforced by** bullet whose first name is no spec under `__tests__/` or `e2e/`. Names under **Not yet
  enforced** are specifications, not claims, and are read by neither.
- `__tests__/fitness/a-baseline-only-shrinks.test.ts` — rule 9. Reads each baseline back out of the last commit
  and fails on an entry the working tree carries and that copy did not. A baseline the last commit does not have
  is newly seeded and passes.

**Not yet enforced.** Nothing: every rule above names a test that exists.

## Traps

- **Rule 5.** The git index does not expand braces, so a brace glob matches nothing and the sweep passes over an
  empty set while reporting green.
- **Rule 6.** A worktree holds a stale copy of the whole repo. A sweep that walks the file system instead of the
  index measures a checkout nobody ships.
- **Rule 9.** A baseline regenerated from a red run absorbs the failure. Regeneration prunes only, and a
  regeneration commit that adds a line to a baseline is the bug.
- **Rule 15.** A rename that leaves the document behind is invisible until a reader follows the name and finds
  nothing, and then trusts the next name less.
- **Rule 17.** A carve-out inside a rule reads as part of the rule forever. The next author extends it.
