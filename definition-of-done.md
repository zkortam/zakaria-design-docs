# Definition of done

This document owns the moment a visual change is finished. Everything else here describes what the product must
be; this describes the evidence that it is, on the surfaces a person actually holds. The feeling it protects is
the one nobody notices: that every screen was seen by somebody before it was seen by them, at their width, in
their theme, at their text size, on their connection.

## Owns and defers

Owns: the per-change checklist, the order it runs in, the reference set a change is judged against, the tools
that produce the evidence and what each one proves, and the weekly cadence that keeps the product done after it
ships.

Defers: what a rule says to `[principles]` and the document that holds it; the contact-sheet standard and the
reference pages' composition to `[craft]`; the budgets a throttled scroll is measured against to `[speed]`; how
a rule becomes a test to `[enforcement]`; the traps that shipped to `[traps]`.

## Rules

A change is done when every rule below is true of it. Rules 1 to 12 run before the commit, 13 and 14 before the
push, 15 after the deploy, 16 to 19 every week. Nothing here is satisfied by reading the diff: jsdom returns a
zero box for every element, so the browser is the proof.

### Before the commit

1. **Both postures, seen.** The change is opened at 390 wide with a coarse pointer at device pixel ratio 3, and
   at 1440 wide with a fine pointer. `tools/uiprobe/` drives both widths against a running app and writes
   `tools/uiprobe/shots`; the `mobile` and `android` projects in `playwright.config.ts` drive the same two
   engines the phone estate runs. A change to anything at a screen's edge is also seen on an 820×1180
   coarse-pointer tablet and turned once (`tools/uiprobe/device-fit.probe.ts`), and `e2e/every-inset-is-paid.spec.ts` is
   green on its device classes.
2. **Both schemes, both switches.** The change is seen in light and in dark, with the account's theme
   (`data-theme`, from `THEMES`) and the OS colour scheme (`prefers-color-scheme`) both set to the scheme under
   test. Setting one and not the other renders light and measures nothing.
3. **The tallest and the widest user.** The largest real consumer of the primitive is opened, never the
   smallest: the tallest `DIALOG_SIZES` caller, the longest list, the longest name at the largest text scale,
   the account with the most of whatever the surface counts.
4. **Opened, closed, opened again.** From the real trigger, three times. The exit lands where the entrance
   began, Escape closes the topmost surface only, and Back leaves exactly one thing.
5. **Driven, not visited.** The trigger is clicked, not navigated to. A route visited directly cannot show a
   Suspense flash, a lazy fallback or an unwarmed chunk, which is where those defects live.
6. **At the largest text scale.** The viewer's largest step multiplies the whole ramp through `--font-scale` and
   moves the rem chrome with it. Nothing clips, wraps to a second line of controls, or pushes a row past its
   height.
7. **In the other direction.** Where the change has direction, it is seen in an RTL locale: chevrons, slide
   direction, gesture side and inline padding all mirror. `isRtlLocale` decides the document's `dir`. The
   `rtl` and `rtl-mobile` projects in `playwright.config.ts` sign in an Arabic account through
   `e2e/rtl.setup.ts`, and `tablet` and `tablet-landscape` drive an iPad Pro 11: a coarse pointer at desk
   width, where a press is decided by the pointer and not the width `[layout §2]`.
8. **Under reduced motion.** With the OS preference and the in-app switch (`REDUCE_MOTION_CLASS`) on. Travel is
   gone, presence remains, nothing teleports and nothing hangs waiting for a transition that never ran.
9. **Offline where it sends.** Any change that writes is exercised with the network off: the act is answered in
   the frame, the queue is visible, `components/shared/offline-banner.tsx` says what is happening, and coming
   back online reconciles without a second tap.
10. **Scrolled at throttle.** With the performance panel recording, on a 4x CPU throttle over a throttled
    connection: no long frame, no layout shift after first paint, no off-screen decode, no blur animating.
11. **Judged as a contact sheet.** The new screen is placed beside the reference set in `[craft]` and read as one
    sheet. If it looks like a different product, it is not done. `[craft]` holds the standard the sheet is read
    against.
12. **Copy read aloud.** Every string on the surface is spoken. A sentence that survives being cut is cut, a
    label that restates its row is deleted, and a word that is not in the app's voice is replaced.
13. **The native-impact check answered.** `CodeCheckList.md` asks the three questions; a change that touches a
    shell carries its seam, its generator or its gate in the same commit [shells].

### Before the push

13. **Types checked to completion.** `npm run type-check` exits 0 with a real exit code in the log. The build
    does not check types, so this is the only type gate and a truncated log is a kill, not a pass.
14. **Fitness green, and one rule richer.** `npm run test:fitness` passes, and any rule this change implies is
    added to its document with its test in the same commit. `[enforcement]` owns how.

### After the deploy

15. **The click path driven on production.** The exact sequence a person takes is driven by hand on the deployed
    build, signed in, from the trigger. `npm run smoke:deploy` runs `scripts/smoke-deploy.mjs` first and proves
    the frame is alive; it does not prove the change.

### Every week

16. **A real phone in the hand.** The core walk runs on a real older iPhone and a real mid-range Android:
    `/home`, `/explore`, `/shorts`, `/messages`, `/settings`, compose, send, open a post. Press feedback,
    scroll smoothness and keyboard behaviour exist only on hardware; a CDP touch event never fires `:active`.
    Monthly, an iPad and a foldable, folded and unfolded on Samsung's Remote Test Lab, walk the same path.
17. **A contact sheet of the estate.** Screens from the week's changes are read beside the reference set at both
    widths in dark. `npm run ui:probe` shoots the settings family through
    `tools/uiprobe/settings-shots.probe.ts`, in the theme the signed-in account carries, so put that account in
    dark first; the rest is captured by hand.
18. **Field vitals per route.** Read what real sessions measured, per route, against the ceilings in `[speed]`,
    and open anything that crossed one. What is read today: `app/layout.tsx` mounts Vercel Speed Insights in
    production only, so Largest Contentful Paint, Interaction to Next Paint and Cumulative Layout Shift arrive
    per route in the Vercel dashboard and nowhere the repo can see; `reportClientPerformance` adds four sampled
    events (`ClientPerformanceEvent`) at one in ten sessions, with navigation latency from
    `components/shared/navigation-performance.tsx`. What must exist: a per-route p75 read the repo can assert
    against, so a route that crosses a ceiling fails a check instead of waiting to be noticed `(not yet built)`.
19. **Rage taps and dead clicks.** Read the taps that repeated and the taps that did nothing, weekly, and treat
    each as a defect report. Nothing collects either signal today: no repeated-tap or no-op-tap instrumentation
    exists, and the analytics events are four named successes, so a tap that never resolved is invisible
    `(not yet built)`.

## The reference set

The pages a change is judged beside, and the order they are read in, are `[craft]`'s contact-sheet table. This
document does not repeat them; it only says when the sheet is read: at rule 11 for a change, at rule 17 for the
week.

## The tools, and what each proves

| Tool | Proves | Cannot see |
|---|---|---|
| `tools/uiprobe/` | Real Chromium at 390 and 1440 driving the running app; prints geometry, writes screenshots | Nothing: it reports and never asserts, on purpose |
| `e2e/targets.spec.ts` | The hit box a finger actually gets, widened by `tap-target`, and the gap to the next control | Whether the control does anything |
| `e2e/ui-integrity.spec.ts` | Horizontal overflow and post-paint shift, every route, both form factors | Anything behind a trigger |
| `e2e/design-system.spec.ts` | The token and hit-area rules measured in a browser rather than read from source | Motion |
| `e2e/motion-tokens.spec.ts` | The durations and easings the rendered page actually uses, read back against `app/globals.css` | Whether the motion looks right |
| `e2e/ui-primitives.spec.ts` | That every sheet, modal, menu and toast funnels through the primitives | Per-surface choreography |
| `e2e/accessibility.spec.ts` | Axe against a real render over `e2e/fixtures/swept-routes.ts`, ratcheted | Anything a screen reader announces in sequence |
| `e2e/performance.spec.ts` | Fixed budgets on the hot routes | The field |
| `e2e/performance-budget.spec.ts` | A per-route JavaScript and timing ratchet over the whole `ROUTES` set | The field |
| `e2e/visual.spec.ts` | Pixel snapshots of the surfaces that must not drift | Whether a drift is an improvement |
| `e2e/resilience.spec.ts` | Service worker, cache storage and reconnection, which jsdom has none of | Real radio behaviour |
| `tools/uiprobe/settings-shots.probe.ts` | A contact sheet of every settings surface in `e2e/fixtures/swept-routes.ts`, at phone and desktop width | It asserts nothing; it exists to be looked at |
| `scripts/smoke-deploy.mjs` | On production, signed in: `/home` loads, the composer types and posts, a post page renders, a desktop dialog keeps a resolved height ceiling, login accepts text, no page errors | Everything the change touched |
| `__tests__/fitness/` | The source-level rules of these documents | Layout, colour, timing, anything with a pixel |

`docs/TESTING.md` holds why the suites are split and what each one structurally cannot see. The rule that
governs all of it: put logic in the unit suite and anything a person perceives in `e2e/`.

## Enforced by

- `__tests__/fitness/design-docs-cite-what-exists.test.ts` — rule 14, for these documents: every identifier a
  rule names resolves to something in the repo.
- `__tests__/fitness/vercel-build-stays-on-the-free-machine.test.ts` — rule 13: the build machine does not
  re-run what the push already ran, so `npm run type-check` stays the local gate.
- `.githooks/pre-push` — rules 13 and 14, on every push, before the change leaves the machine.

**Not yet enforced**

- Rules 1 to 12 are a person's work. They are enforced by the reviewer, and each has a tool above that makes the
  evidence cheap to produce.
- `a-route-holds-its-field-budget` — reads the per-route p75 for Largest Contentful Paint, Interaction to Next
  Paint and Cumulative Layout Shift from a checked-in field export and fails when a route crosses the ceiling
  `[speed]` holds. Rule 18.
- `a-tap-that-repeats-is-a-defect` — reads a rage-tap and dead-click export and fails on any surface above a
  recorded floor. Rule 19.

## Traps

- **Rule 2.** A dark sweep that sets only the OS scheme, or only the account theme, renders light. Both switches
  or the screenshots lie.
- **Rule 5.** A `goto` cannot see a Suspense flash. Only a click can, because the flash is the chunk arriving
  after the tap.
- **Rule 1.** CDP touch events never fire `:active`, even on a bare page, so no Playwright sweep can prove press
  feedback. A transparent `-webkit-tap-highlight-color` plus WebKit gating `:active` on a touch listener means
  zero feedback on iOS while every press class looks complete on the desktop screenshot.
- **Rule 1.** jsdom returns a zero box for every element, so a unit test asserting a 44px target compares
  zero against forty-four and passes.
- **Rule 10.** Two browser sweeps on one machine turn 2% dropped frames into 30%. One sweep at a time, on a
  quiet machine.
- **Rule 11.** `scrollWidth` is inflated by `tap-target`'s pseudo-element and by any descendant scroller, so it
  is unusable as an overflow signal. Judge the visible box, or judge the screenshot.
- **Rule 11.** React streams the real subtree inside a hidden element, so a DOM count reports rows nobody can
  see. Measure whether the skeleton appeared, not how many rows exist.
- **Rule 13.** `next.config.ts` turns type checking off in the build, which prints "Skipping validation of
  types" and ships a broken import. A zero-byte type-check log with no exit code is a kill, not a pass.
- **Rule 15.** The app subdomain redirects, and a probe that carries cookies to it measures the login page.
