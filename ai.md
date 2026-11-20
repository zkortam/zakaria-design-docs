# AI

Surf has dozens of AI features and one way of presenting them. A person learns once what the star means, what
the word "AI" on a post means, what waiting on a model looks like, how a model's words are offered before they
replace their own, what an answer rests on, and what it looks like when AI says no. The feeling this document
protects is trust: nothing a model wrote passes for the person's own, nothing a model guessed passes for proof,
nothing is lost while a model works, and no refusal is a dead end.

## Owns and defers

Owns: the two AI marks and what each means; the ask; which of the waits an AI surface uses; the proposal, its
diff, its accept, its undo and its disclosure; sources, the answered stamp and verdicts; refusals and their one
copy catalogue; the conversation shell; where AI lives on each family of surface.

Defers: the shimmer's band and crossfade to `[signature-motion §24]`; the live activity capsule to
`[signature-motion §30]`; the listening edge to `[signature-motion §39]`; the hold before any loading texture to
`[loading-and-states §15]`; toasts, the undo window and the dock to `[feedback]`; the voice of every sentence to
`[copy-and-voice]`; what a paywall must disclose to `[permissions-and-trust]`; the icon catalogue to
`[components §3]`; the overlay that hosts a conversation to `[components §12]`.

## Rules

### The marks

1. **Asking AI is the star with its verb.** `ICONS.ai` beside the word for what it does: Ask, Rewrite, Draft
   with assistant; in the editor's rail, where every tool is one word for what it holds, AI. A control that stands on its own is `AiInvoke` in `components/ai/ai-mark.tsx`: the glyph
   button with the verb as its name in a toolbar or a header (`glyphOnly`), the quiet button with the verb as
   its text beside words. A row with a recipe of its own (the rail, the feed's action row) draws `AiGlyph` in
   it with the verb as its label; a menu or a nav row takes `ICONS.ai` as its `icon`. The split writes no word
   of its own, so its row is named by what it makes, `Rows`, never the star. Nothing else draws the star, and
   the star means nothing else.
2. **Content made with AI wears the AI mark, never the star; Surf's own intelligence wears none.** `AiBadge`
   draws it, the letters AI in a rounded square at the verified seal's size and stroke, named for its level, at
   one of two levels, read by `aiBadgeLevel` from the signals behind a verdict, never the verdict alone:
   `disclosed` where the author, or the file itself, says so ("Made with AI"), and a game Surf built for a
   profile; `verified` only for Content Credentials whose binding held these bytes and whose signer chains to
   the trust list in `lib/provenance/trust/` ("AI, verified"). An answer, summary, recap, recall, insight,
   drafted note or suggestion Surf writes carries no "AI" word: the surface is Surf's, so the stamp is noise;
   a conversation is the one exception, and says it is AI once (rule 26b).
   No detector's guess is a signal: a label rests on C2PA, IPTC metadata, a generator's own tag or the author's
   word, and the sheet names which, in the words `labelStatement` in `lib/provenance/label.ts` chooses. The
   author's own word outranks everything. A label another file put on an undisclosed item is an `ai_label` case
   its author is notified of and can dispute to a person; an upheld dispute takes the label down for as long as
   the item carries the same media. A manifest signed with Surf's own certificate (`SURF_C2PA_SIGNER`) opens no
   case; a file that only names Surf does. Only the author is offered the dispute, which opens that label's
   case, and only staff the removal, whose undo puts the label back under its old case without telling the
   author again.
2a. **A model's words read in Surf's markdown.** Every answer, summary, recall and finding renders through
    `RichText` (`AiCitedText` and `AnswerText` do), so `*Cars*` never reaches the screen as asterisks.
3. **A card says it once, beside the seal.** `PostAiBadge` stands first in the name's trailing row, right after
   the verified seal, on a text row and over media alike, and opens how it is known; a story's header and a
   game's title carry the same mark in the same place. Never a pill of words, which crowded the name, and never
   among the controls, where it reads as one and crowded them off a phone's edge.
4. **Sparkle, Robot, Brain and MagicWand mean nothing about AI.** They keep only meanings a person could not
   mistake for it: For you, Glow, camera effects, a robot voice, text animation.

### The ask

5. **One field, and chips that fill it.** `AiPrompt`: the field first, a send inside its pill, and under it the
   chips the surface chose for the object in front of the person, never a fixed menu. A chip writes its words
   into the field and asks them; `instruction` carries the model's wording where it differs from the reader's.
   The field never moves: the chips, then the answer, sit under it, so steering keeps the caret and the keyboard.
   Where a route caps the question, the field counts down to it (`CharacterLimitRing`), so it never fails at send.
6. **Send becomes Stop only where the work can really be stopped.** A route that forwards the request's abort
   to its model says so (`stoppable` on `useAiProposal` and `useAiThread`). Elsewhere the server would finish,
   and bill, the answer anyway, so the circle only waits, the object shows the wait instead of a spinner, and
   Keep mine leaves the request behind.
7. **The last ask stays in view.** Quoted above the proposal (`ai.prompt.lastAsked`) in the words the person
   tapped or typed, never the wording the model was sent, so "shorter still" steers from what was asked rather
   than starting over.

### The wait

8. **One wait per surface, chosen by what the person is looking at.** Words or a board changing in place:
   `AiShimmer` [signature-motion §24]; a proposal being written or steered shimmers the words it will change,
   in the proposal's own place (`AiProposal working`), where a rewrite's words stream in plain until the whole
   answer is checked and drawn as the diff. An answer being written: `AiProgressLine`, then the words as they stream
   (`useAiStream`, `streamAi`). A voice being heard: the edge [signature-motion §39]. A job longer than the
   screen: the capsule, through `startLiveActivity` [signature-motion §30]. Never bones, a
   spinner and words at once.
9. **Progress is real or it is absent.** The line before an answer's first word is the step the route reports
   (`{ type: 'progress' }`), shown after the hold [loading-and-states §15]; where the route names none it says
   `ai.working.answering`, never an invented step.
10. **What shimmers is never remounted.** `AiShimmer` crossfades the same node, so a running game keeps its run
    and a field keeps its caret.

### The stream

11. **One protocol, read at the edge.** `text/event-stream`, one JSON object per `data:` line: `progress`,
    `delta`, `sources`, `done`, `error`. `parseAiStreamEvent` validates every event; a body that ends before
    `done` is a failed answer, never a short one; a route still answering in one JSON body is read as `done`.
    A route that checks its words before anyone reads them (`answerAsk`) names the check as its progress and
    sends the words that passed as one `delta`; a draft its checks refused is never streamed. The Studio
    assistant (`answerInDock`) streams each read as progress and hands its figure-checked answer whole in `done`.
12. **A new ask never reads the last one's ending.** Each run is named, and a state from a previous run is never
    shown for the next.

### The proposal

13. **A model's words sit beside the person's before they replace them.** `AiProposal` draws `readableDiff`:
    what was cut struck through, what was added marked, each change one hunk, never colour alone; a screen
    reader hears the proposal whole. A chain's entries are passages of their own, stacked by gap, never joined.
    Where the surface passes `onToggle`, each change is the person's to take or leave by itself
    (`proposalPieces`, `composeProposal`): a tap leaves it out and a dotted rule marks where it was, a run that
    swaps word for word is that many changes, and Use this takes the rest.
14. **Use this leads, Keep mine stands beside it at the same size.** Use this is the primary button and Keep
    mine the secondary, so the offer reads as an offer and declining costs the same reach `[permissions-and-trust]`.
    Retry asks the last instruction again; Edit, at publish only, takes the words into the field to finish by
    hand; Report, beside Retry, files the proposal's words as a report on an AI answer.
15. **A proposal made from words that have moved is void.** `proposalStanding` reads it `stale` the moment the
    source changes, and `entriesStanding` for a proposal over several entries (the publish-time polish); Use
    this leaves, and Retry rebuilds it from the text as it stands. A split into a thread keeps the draft it was
    made from and goes stale the same way. Nothing a model wrote ever overwrites newer words.
16. **Taking it is one undo away, in the toast, on the keyboard and in the platform's gesture.** `useAiProposal`
    writes the words through the field's own edit (`writeFieldText`) with the caret back in it, so the field's
    history holds them: Cmd+Z, a shake, a three-finger swipe or the Edit menu take them back natively and walk
    back through anything typed after. The toast undoes through the same history and withdraws once the person
    writes on (`watchFieldHistory`). Where the platform refuses the edit, the surface's own model applies the
    words and returns the inverse, and `armFieldUndo` gives it Cmd+Z on any layout and the undo gesture while
    the field still holds the proposal. It runs once, however it is asked. Ghost text is taken the same way,
    quietly (`takeAiWords` with no toast): a few words at the caret need no offer, and Cmd+Z takes them back.
17. **Taking it says AI wrote it, and undoing it takes that back.** `onAccepted` receives the `AiAcceptance`,
    which names the tool (`AI_DISCLOSURE_REASON_KEYS`); the surface turns "Made with AI" on with that reason,
    said under its switch, and returns the inverse the undo runs with the words'.
18. **A closed ask leaves nothing behind.** Closing it, posting, or resetting the composer (`dismiss`) abandons
    the request and forgets the ask, the proposal and the refusal, so nothing reopens under a new draft.

### Sources and verdicts

19. **An answer shows what it rests on, and when.** `AiCitedText` turns `[n]` into a link to source n;
    `AiSources` lists each source's title, then its publisher and date apart by a gap, linked only where it gives a safe link, and
    stamps the answer with when it was given, or a finding with when it was checked (`stamp`, read by
    `answeredStamp`), drawn only in the browser from the viewer's own clock. A marker that points at no listed
    source stays plain text. Where sources sit as pills (`SourceChips`, under an @ask reply), each is one line
    naming the publisher or site in at most three words (`chipLabelOf`), never the page's title.
20. **Zero sources is said as zero, and not knowing is an answer.** "No sources found" for a search, "Based only
    on what is here" for an answer from the object alone, said per turn, since a follow-up can rest on the
    object where the first answer searched. Nothing calls a source trusted. An answer that could
    not be verified (`abstained`) says so above its words, in primary ink with what was checked beneath it,
    and keeps Copy and Share: it is an outcome, never drawn as an error and never offered a Retry. Words the
    sources still do not support after the rewrite are withheld, and the line says the sources do not settle it.
21. **A verdict waits for a second check.** `AiVerdict` renders nothing below `VERDICT_MIN_CHECKS`; above it,
    the claim quoted, the ruling, how many checks agree, in words (`countOf`), and who disputes it by name. A
    ruling says what the sources do to the claim, context added among them, and none of them is "false".
22. **Keystroke provenance is ink, not alarm.** Typed, pasted, inserted and AI shares are drawn in the ink tiers
    (`PROVENANCE_GROUP_COLORS`); pasting is ordinary writing. "About this post" gives each answer one inset
    panel: its verdict as the headline, the words as one segmented capsule keyed underneath, the two or three
    figures the verdict rests on (`highlights`), and everything else behind Details. Each picture is shown
    beside its own answer. How the author types is shown to the author alone.

### Refusals

23. **Every refusal is one of the `AI_ERROR_KINDS`, read by one mapper.** `readAiRefusal` reads the route's `aiKind`
    first, then its status, then the network; the person's own Stop is no refusal at all.
24. **A refusal is said in place, from one catalogue.** `AiRefusal` renders inline in the surface that refused,
    never as a corner toast, from `AI_REFUSAL_LINE_KEYS` and `refusalCopy`, never from the route's own text,
    never in the first person.
25. **Each kind carries its one step out.** Crisis: support resources, and nothing to try again; words that
    read as one (`classifyCrisisTopic`) are refused before any request by `useAiProposal`, `useAiThread` and the
    copilot, so no model rewords, continues or answers them: no ghost text is asked for such a draft, and
    opening Rewrite on it shows the support in the suggestion's place; the rewrite route reads it again
    (`detectCrisis`) and answers `crisis` with its topic. Policy: said once. Quota: used of limit in what the allowance counts (`AI_QUOTA_MEASURES`), when it resets, the plan that lifts it and
    Upgrade. Rate: the wait counting down, then Retry; an hourly allowance says the hour's are used up
    (`window: 'hour'`). Tier: shown locked with the plan it comes with (`upgradeOffer`); an entry drawn before
    any ask wears a lock on its star and opens the plan (`AiInvoke locked`), so nothing below its
    plan is offered and then refused. Unavailable, offline and timeout: Retry, and a reference where one was
    filed. Consent: the consent screen itself, `AiConsent` [permissions-and-trust §9a], and once Allow lands the
    same ask goes again. Encrypted and invalid: the reason; a draft over a tool's limit is refused before the request, with the limit (`maxChars`). `refusalCanRetry` decides where Retry appears. A spent allowance
    reads the one catalogue line wherever it is said, a toast included (`refusalCopy`), never a route's own text.

### The conversation

26. **Every conversation with AI is one shell.** `AiThread`: the question as a bubble, the answer in primary ink
    as it streams, its sources and stamp, Copy, Share (the answer's words with the link) and Report (`AiReport`, which files the words shown, since no answer is stored), a refusal in the turn it refused, a stopped answer's
    words kept with Retry, and the ask field under it all. Where the route can write a new answer rather than
    serve a shared one, the last answer offers Regenerate (`regenerable` on `useAiThread`), which asks the same
    question again, fresh. Surfaces add to a turn through `renderWork` and `renderAfter`, never a second shell.
26a. **An agent's turn reads as its work, then what changed.** The question, then `AiSteps` in `renderWork`: a
    row a step, led by the glyph of the tool it ran (`glyphOf`), its title, and its status at the row's end, a
    failure's reason under its title (`reasonOf`); while no step runs, the one `AiProgressLine` under them says
    what the turn is doing. A step that started stays: a plan cut off as it streams keeps the steps it wrote,
    and the turn's next round goes on from them. A reply streams as the answer while it is written, and one cut
    off before any step is written again whole by the next model. The finished turn closes on at most what the
    rows cannot say (the new length, that nothing changed, what the second look fixed and why or that it could
    not look, what no tool could do), then Undo on the rows' columns (`AI_STEP_GAP`). While a turn reads the
    footage or an auto-edit runs, the line names the read under way, the clip it is on and how far it has got,
    then each stage, counted where it counts (`autoEditLine`): no line stands still while the work under it moves.
    Every turn whose edit nothing later touched keeps its Undo, not only the newest (`historyFootprint`).
26b. **A conversation with AI says it is AI, once, under its first answer.** The EU AI Act (Art 50) asks it of
    anything a person talks with: `AiDisclosure` under the first answer of `AiThread`, so the family assistant,
    the Studio assistant and the editor agent say it, and of `AskConversation`. @ask's public replies say it in
    the account's own name, Ask AI (`ASK_DISPLAY_NAME`), rather than a badge. Words a model rewrote and the
    person published keep "Made with AI" through `onAccepted` (rule 17).
27. **A conversation belongs to its object and survives a close.** `useAiThread` keeps one thread per
    surface and object (`aiThreadKey`) for the tab's life, and `useAiThreadParam` carries which is open in the
    URL, so a reload or a shared link reopens it; a shared link opened signed out asks them to sign in first
    and reopens the conversation on return (`AskHost`). Ask is the exception the owner kept: its sheet opens on
    its presets every time and draws `AskConversation`, not `AiThread`, over the same `useAskThread` stream,
    and ends at its answer: no footer and no unverified line, since a fact check opens with its own verdict.
    The editor agent's conversation belongs to its project and is kept in the project's draft (`agentThread`);
    a template applied over the project keeps it, and only a new video starts a new one.
28. **On a phone it is a sheet; on a desk it never covers the object.** `AiThreadSheet` is a bottom sheet under
    the desk breakpoint, with Clear in its header, and a non-modal panel docked at the side above it; a surface
    with a dock of its own hosts `AiThread` there under `AiThreadHeader`, as the family assistant does.

### Where AI lives

29. **One entry per family of surface.** Reading: Ask, the star on a post's action row and the shorts rail, and
    nowhere else on that surface: never a menu row or a header pill beside it. Writing: one
    Rewrite control opening `AiPrompt` under the text, and a "Made with AI" chip once AI touched it: in the meta
    strip of a post, under the text of a reply, and for the entry of a chain that earned it, never the whole
    chain. Making: one AI entry in the editor opening the conversation, each tool once with its lock shown. Studio:
    the describe bar, with runs in the capsule. Settings: one AI page governing every AI feature. Making is
    (not yet built) to this rule.
30. **Every optional AI feature answers to one switch on the AI page.** The switch is read where the feature runs
    (`getSurfVisionEnabled`, `useSurfVisionEnabled`), so off means no request and no entry, and every route that
    runs optional AI for the caller names the switches it answers to (`aiSwitch`), so a stale tab or a direct call
    is refused as `off` before anything is charged, and Studio's schedules read it in SQL. Translating on a tap
    and translating on its own are two switches. The page leads with AI help, those switches read as Off, When I
    ask or Everywhere (`readAiHelp`), with the outcome of the stop under the finger drawn beside it
    (`AiHelpPreview`); a hand-set mix rests between two stops and says it is the person's own, and a stop that would lose a mix offers Undo
    (`aiHelpRestores`). What runs when asked and what runs on its own are two groups; safety checks, what an
    author or a file states about AI, reading a post aloud (how some people read), dictating a search and what
    a post carries for every reader are no switch; a guardian's assistant answers to Family, the describe bar to
    the feed page. A surface made of one feature, with it off, says so and turns it back on (`AiSwitchedOff`).
    What AI keeps about the person is listed by what it shapes, each editable or forgettable (`AiMemorySection`).

### Writing

31. **A writing tool hands the person's words back whole or not at all.** Code, links, mentions, tags and emoji
    reach the model as placeholders (`maskSpans`) and come back exactly once each (`restoreSpans`), or the
    answer is refused and the next model asked; a rewrite stays in the draft's language (`keepsLanguage`). Polish
    names typos and only repairs that pass `editFault` land. A split cuts the author's own sentences where the
    model says (`splitAtBreaks`), so no word is written by it. Ghost text is asked for only where it can be shown
    (`ghostEligible`), is taken by the forward arrow at the draft's end, a tap on the arrow drawn after it or a swipe
    along it, is put away by Escape, never takes Tab, and a tap on its words only places the caret; it names no person, place or
    tag and supplies no opinion (`shapeGhost`).

## Enforced by

- `__tests__/fitness/ai-speaks-one-language.test.ts`: rules 1, 2, 4 and 8, through the star, look-alike-glyph
  and own-loader baselines, and a hard rule that only the badge spends a badge label.
- `__tests__/fitness/one-capsule-per-frame.test.ts`: rule 8's job half, owned by `[signature-motion]`.
- `__tests__/fitness/an-icon-means-one-thing.test.ts`: rule 1's glyph, which only `ICONS.ai` may draw.
- `__tests__/components/ai/ai-mark.test.tsx`, `__tests__/components/post/post-ai-badge.test.tsx`,
  `__tests__/provenance/iptc-and-badge.test.ts`, `__tests__/provenance/content-credentials-round-trip.test.ts` and
  `__tests__/provenance/label-statement.test.ts`: rules 1 to 3.
- `__tests__/components/ai/ai-prompt.test.tsx` and `__tests__/hooks/use-ai-proposal.test.ts`: rules 5 to 7.
- `__tests__/components/ai/ai-working.test.tsx` and `__tests__/components/ai/ai-steps.test.tsx`: rules 8 to 10.
- `__tests__/components/ai/ai-steps.test.tsx`, `__tests__/components/shorts/editor/editor-agent.test.tsx` and
  `__tests__/shorts/editor-agent-turn-e2e.test.ts`: rule 26a, and the editor agent's half of rule 27 with
  `__tests__/hooks/use-editor-restore.test.ts` and `__tests__/shorts/use-shorts-create-flow.test.ts`.
- `__tests__/domain/ai/stream.test.ts` and `__tests__/hooks/use-ai-thread.test.ts`: rules 11 and 12.
- `__tests__/components/ai/ai-proposal.test.tsx`, `__tests__/hooks/use-ai-proposal.test.ts`,
  `__tests__/utils/field-undo.test.tsx`, `__tests__/utils/diff.test.ts` and
  `__tests__/hooks/an-ai-rewrite-is-a-proposal-with-undo.test.ts`, `__tests__/components/compose/rewrite-panel.test.tsx`,
  `__tests__/hooks/use-reply-composer.test.ts` and `__tests__/domain/writing/writing-tools.test.ts`: rules 13 to 18.
- `__tests__/components/ai/ai-sources.test.tsx` and `__tests__/domain/ai/sources-and-proposal.test.ts`: rules 19
  to 21.
- `__tests__/domain/ai/refusal.test.ts`, `__tests__/domain/ai/refusal-copy.test.ts`,
  `__tests__/components/ai/ai-refusal.test.tsx` and `__tests__/hooks/use-thread-copilot.test.ts`: rules 23 to 25.
- `__tests__/domain/writing/writing-tools.test.ts`, `__tests__/components/compose/composer-typing.test.tsx`,
  `__tests__/api/ai-text-routes.test.ts` and `__tests__/live/writing-eval.test.ts`: rule 31.
- `__tests__/settings/ai-settings-contract.test.ts`, `__tests__/components/settings/ai-settings.test.tsx`,
  `__tests__/hooks/ai-help-off-gates.test.tsx` and `__tests__/api/api-handler.test.ts`: rules 29 and 30; `__tests__/domain/error-mapper.test.ts`: rule 25's one wording for a limit.
- `__tests__/components/ai/ai-thread.test.tsx`, `__tests__/domain/ai/thread.test.ts` and
  `__tests__/components/family/family-assistant.test.tsx`: rules 26 to 28.
- `__tests__/fitness/every-ai-surface-discloses-ai.test.ts` and `__tests__/components/ai/ai-thread.test.tsx`: rule
  26b; `__tests__/components/ai/ai-consent.test.tsx`: rule 25's consent step.

**Not yet enforced.**

- `every-ai-surface-is-on-the-primitives`: rules 26 and 29. Fails on a conversation shell, a proposal card or an
  AI entry point outside `components/ai/`; waits on the studio assistant and the analyst moving onto `AiThread`.
  Ask and the editor agent keep their own conversation shells over the thread hooks, by the owner's call.
- `a-verdict-names-its-second-check`: rule 21 at the source, reading every verdict the routes store for a second
  pass before one is served.

## Traps

- **2** The word "AI" was stamped on every summary, recap, recall, insight, drafted note and ghost suggestion Surf
  wrote, a dozen surfaces, until it read as spam and said nothing a person needed; only content made with AI
  keeps a label.
- **2** One detector pass at a fixed confidence was labelled "Likely AI" beside a sheet that called the same
  image a screenshot, and read off the verdict alone, the badge kept saying it after the fix. Every likely post
  on production rested on one classifier signal at 0.87. Detectors no longer label anything, and the level is
  read from the signals.
- **2** A real ChatGPT image read as unlabelled twice over: its AI statement sits on the parent ingredient it
  opened, and OpenAI's certificate had expired with no timestamp, which c2pa-rs reports as Invalid. Hand-written
  manifest shapes passed; `__tests__/provenance/real-files.test.ts` reads the real files.
- **12** A thread's next turn read the previous turn's `refused` state in the render before its own request
  started, and settled itself as refused without asking anything.
- **15** A rewrite in flight overwrote everything typed while it was being written, and could land in the wrong
  entry of a thread.
- **7** A chip sent the model English wording, and that wording was quoted back above the proposal in all nine
  languages, as words the person never typed or tapped.
- **16** A value set in code has no entry on the field's own undo stack, so Cmd+Z did nothing after a rewrite
  was taken; only the toast could take it back, and only for five seconds. The first fix read only the Z key's
  letter, so it did nothing on a Cyrillic, Arabic or Korean layout, and it listened on a field the focus had
  already left with the button that took the rewrite.
- **16** A rewrite taken through the field's own edit reached the composer as an input event, which named it
  ghost text: one span from the first changed word to the last, so a hand-typed post polished by Rewrite read as
  99% AI. The field now carries the tool's own cause, and a rewrite keeps the history of every word it kept.
- **17** Undoing a taken rewrite gave the words back and left "Made with AI" on, so the author's own words
  shipped wearing the badge and were hidden from people who filter labelled posts.
- **18** A refusal held the Rewrite panel open: closing it, and even posting, left the refusal and the field
  under the composer.
- **19** The answered stamp was worked out in the server render, in the server's zone, and mismatched the
  browser's by a minute and an hour, so hydration threw the thread away.
- **20** "Generated from the thread and trusted sources" was shown under a political verdict that cited nothing.
- **20** A fact check of the author's own announcement searched the web, ruled it "unconfirmed" and listed
  macOS 27 and a Trump speech as its sources; the store had filled every rerank seat before a page read live
  was scored. A post that makes no claim is now said to hold nothing to check, with no search and no sources.
- **27** Closing the Ask sheet cleared `?ask=` on the sheet's own history entry, then popped onto one that
  still named it, so X and Escape reopened it at once; `useAiThreadParam` clears it after the pop lands.
- **24** One corner toast said "Couldn't load this" for a tier lock, a quota, an empty draft and being offline.
- **25** A crisis draft sent to Rewrite came back as the model's refusal sentence, offered as a one-tap proposal
  to post in the person's place.
