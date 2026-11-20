# Components

One primitive per concept. Every dialog opens the same way, every close is the same control, every
empty state has the same shape, and every menu row reads identically whichever menu opened it. That
sameness is what lets a person stop learning the interface after the first screen: they already know
what a sheet does, so the sixth one costs them nothing. A second shape for a concept is not variety,
it is a second thing to learn and a second thing to keep correct.

## Owns and defers

Owns: the primitive per concept, extension over siblings, the ban on per-feature chrome modules, one
icon per concept, focus shown once, tabs driven from the URL, a page's sections and refinements, lazy
overlays, and the overlay family with what each member owns.

Defers: the token values a primitive reads to `[tokens]`; how a control looks and how a surface is
composed to `[craft]`; the four states a surface can be in, and the ghost that stands in for it, to
`[loading-and-states]`; press, haptics and toasts to `[feedback]`; the choreography of an overlay's
enter and exit to `[motion]`; field keyboards, validation and the composer to `[input-and-forms]`;
names, roles and focus order to `[accessibility]`.

## Rules

1. **Reuse the primitive; if it cannot do the job, extend it.** Never build a sibling. A second
   component for a concept that has one is the drift this document exists to stop.
2. **No per-feature chrome module.** A `*-chrome.ts`, a second `*_CARD`, a third `*_ROW`, a private
   `*-style.ts` beside one screen is a design system for one page. Converge onto the primitive or onto
   a `lib/styles/` recipe `[tokens §6]`.
3. **One icon per concept**, from `lib/constants/icons.ts`, one glyph family per row. The repost loop
   is the repost loop on every surface.
4. **A glyph is sized by its container.** Inside an `IconButton` the size prop on the SVG does
   nothing; the button sets it. Elsewhere the size comes from `ICON_SIZES`, and a glyph leading a row is
   `shrink-0`, so every row's glyph is that size whatever its label runs to.
5. **Inputs show focus once.** A bordered field's focus is one change to its one edge, `FIELD_FOCUS_EDGE`
   (`FIELD_SHELL_FOCUS` on a shell): the edge takes the focus ink and doubles inward, and a refused field
   keeps its red edge while focused, `FIELD_ERROR_EDGE`. A composer, a textarea or any large editing
   surface shows focus with the caret and the placeholder alone. Nothing draws two rings.
6. **A bare control is a defect.** A `<button>` defaults to `type="submit"` and re-implements the
   focus ring, `aria-busy` and disabled handling; a bare `<input>`, `<textarea>` or `<select>`
   re-implements height, radius, ring and error state. Use `Button`, `IconButton`, `UnstyledButton`,
   `Input`, `Textarea`, `Select`, `Field`.
7. **A row of mutually exclusive choices is one primitive.** `ChoiceGroup`, `ChoiceChips`,
   `ChoicePill` or `SegmentedControl`, never a hand-built pill row. Hand-built rows lose the
   `radiogroup` role and the roving tabindex.
8. **A section heading is `SECTION_LABEL`.** A heading styled by hand is a second heading style.
9. **`<Tabs>` is controlled by the URL where the strip is.** `defaultValue` alone eats every deep
   link, and a trigger never overrides its own list's variant.
10. **A lazy overlay waits for its first open.** `next/dynamic` renders its `loading:` fallback even
    at `open={false}`, so the import is guarded on first open. Warm the chunk on intent with
    `usePreloadOnIntent`, and a route with `useRouteWarm`.
11. **A fallback is the shape it precedes.** An anchored menu's fallback is an anchored menu, never a
    centred dialog. `SheetFallback rows={0}` is the fallback for a sheet whose body is pure UI;
    `rows={n}` only when the body is a list coming from the network.
12. **An overlay inherits its behaviour.** `Dialog`, `MenuSheet` and the anchored popover are the only
    overlays. They own the portal to `document.body`, the scrim, the focus trap and restore, Escape, Back,
    the scroll lock, the close control, the grabber, the placement and the exit. A surface that builds its
    own gets whichever of those its author remembered — and `fixed inset-0` is not the window when any
    ancestor holds a transform, a filter or a backdrop-filter.
13. **The primitive sizes its own panel.** A consumer picks a `DIALOG_SIZES` preset, not a `max-w-`
    class and not a `--sheet-h` tier. A phone sheet hugs its content between `--sheet-h-xs` and
    `--sheet-h-full`: the floor keeps a one-line sheet from reading as a sliver, and its content stands at
    the top of the panel, never centred in it. `DialogContent`'s mobile sheet in `components/ui/dialog.tsx`
    carries both; a body that would otherwise be empty while it loads owes a skeleton of the shape it is
    about to be `[loading-and-states §13]`.
14. **One close control.** `CloseButton`, at `sm` in a sheet header, `xs` on a banner, chip or inline
    card, `md` beside a `Button` of the same rung. A hand-drawn X in a button is a second close.
14a. **An overlay has one inset, on every edge.** `overlay` spacing (`--spacing-overlay`: 16px, 20px from
    `sm`) is the distance from a dialog's, sheet's or popover's edge to its content, top, sides and bottom
    alike, and between its title row and its body. `OVERLAY_HEADER` sets the title row on it, giving its
    top up only to a drawn grabber (`data-grabber`), never to a phone's fullscreen dialog that merely slides
    like a sheet, whose header stands on the inset below the safe area; and
    `OVERLAY_HEADER_SLOT` pulls the close and trailing controls out so their glyphs, not their hit boxes,
    land on it; `DialogBody` is the one body and `DialogFooter` closes on it. A row's fill bleeds past the
    inset (`ROW_BLEED`) and a quiet text action is `Button flush`, so every word starts on the same edge.
    Nothing inside a body pads its own sides to fix an edge.
15. **A person's name and its handle stack.** `DisplayName` lays them in a flex column; `inline-flex`
    collapsed the handle onto the name with no gap.
16. **An author on a card is `AuthorLink`,** never a `role="button"` div.
17. **A count and a relative time are `[internationalisation §27]` and `[internationalisation §28]`.** A
    surface never writes its own.
18. **A waveform is `waveformEnvelopePath`.** Anything continuous is one shape; dots and bars are for
    discrete choices.
19. **A tap target belongs on a real control.** `tap-target` paints an absolutely positioned `::after`
    at `--tap-target-min`, so it needs `relative` and it must never sit on a wrapper whose children
    are themselves controls.
20. **Two components never share a name.** A duplicate component name is two answers to one question,
    and the import that resolves to the wrong one is invisible in review.
21. **A signature motion is one primitive.** The morph, the roll, the capsule and the listening edge each exist
    once, in the file the table names; a feature that needs one imports it and passes an identity or a level.
    A second capsule, a second edge glow or a hand-rolled morph is the drift this document exists to stop.
22. **A surface that will be morphed into is warmed on intent** `[speed]`, so it exists when the morph starts.
23. **A fill against a total is `Meter`.** An upload's bytes, a ceiling's usage and a reward's progress are
    one concept, so they are one primitive with one `size` recipe per height and one `tone` per meaning.
    A hand-built track and fill is a second meter, and it arrives without the roles `[accessibility §40]`.
24. **A plot is `ChartFrame`.** Line, bar and stacked charts are marks drawn inside the one frame,
    which owns the scale, the grid, both axes, the hover and the empty state; a chart that draws its
    own axis is a second answer to the same question. The axis counts in steps a person reads
    (`niceScale`), every tick is a multiple of one step, and a tick label is centred on the grid line
    it names — a rounded step that lands a label off its line is the wrong step. An x-axis label is
    dropped, never drawn into its neighbour, and the plot mirrors as a whole under RTL
    `[internationalisation §16]` rather than a property at a time.
25. **One composer offers what the other does.** The feed's inline composer, the modal one and the phone
    deck are three frames over `useThreadComposer` and `useComposerOptions`, not three products: a
    capability one surface can reach — drafts, scheduling, trial, the copilot — is reachable from all.
    Placement may differ with the space each has; the feature set may not. Their secondary options are
    one list, `ComposerMoreMenu`, stated back as one `ComposerMetaStrip`, opening one
    `ComposerOptionSheets`; a frame that writes its own `DropdownMenuItem` has started the second list
    this rule forbids. A phone opens the deck for every create, whatever opened it.
26. **A link a person wrote is `UserLink`.** A post, a reply, a bio, a message, a card or a note source
    opens in a new tab with `nofollow ugc`, so nothing a person posts hands search rank to where it points.
26a. **A credit a licence requires is `CreditLink`.** Map tiles and a catalogue song's deed link to the page
    that states their terms, in a new tab, with no referrer handed to the third party.
27. **A page's sections are `TabStrip`.** Underline tabs over content the page swaps: text only, muted ink,
    the chosen tab in primary ink on one neutral capsule (`bg-overlay`, no accent), which glides between tabs on transform. Every
    tab is one weight, so choosing one moves none of the others. There is no pill tab: `Tabs` draws
    `underline`, and `segmented` only for a two-way switch inside a sheet or a tool. A feed with no sections of
    its own reads its order as its tabs (`FeedSortBar`, the replies lens, a list's feed). The strip owns the
    column's inset: its first capsule rests on the column's edge (`STRIP_GUTTER`) and scrolls out to the row's,
    and the filter's glyph ends on the far edge; a strip set inside an already padded column bleeds by
    `CARD_BLEED.out`, so no page offsets it by hand.
28. **A page's refinements are one `FilterButton`.** Sort, window, format, author scope, a floor and a set of
    kinds sit in one sheet behind one quiet glyph, at the end of the tab row or in the header, and the glyph
    counts what is set. While anything is, `FilterSummary` names it on one muted line under the tabs with a
    Clear. The sheet's rows and its Reset are `[input-and-forms §39]`. A single pick behind the glyph is
    `GlassFilterPopup`, which is that sheet holding one list under the sheet's own title, the chosen row
    marked by its check alone, and a door out of the list (Manage folders) as a row beneath it, never as one
    more choice in it.
28a. **A list page is one structure.** Its title row, then its search row (`SearchInput variant="page"` on the
    column), then its `TabStrip`, every one on the same two edges. What narrows the list, a collection
    included, is a refinement inside the one `FilterButton`; a second picker beside it is a second filter.
29. **Pills are content or tools.** `ChoiceGroup variant="pill"` and `ChoiceChips` draw chips that are
    themselves the content (topics, tags, suggestions) or a tool's controls inside a sheet, never a page's
    chrome.

## The one primitive per concept

| Concept | The one primitive |
|---|---|
| Modal dialog | `components/ui/dialog.tsx`: `Dialog`, `DialogContent`, `DialogCompactHeader`, `DialogBody`, `DialogFormColumns`, `DialogFooter`, `DialogItem` |
| Titled sheet | `components/shared/base-sheet.tsx` |
| Menu, action sheet | `components/ui/menu-sheet.tsx` |
| Anchored menu and popover | `components/ui/dropdown-menu.tsx`: `DropdownMenu`, `DropdownMenuTrigger`, `DropdownMenuContent`, `DropdownMenuItem`, `DropdownMenuSeparator` |
| Long-press context menu | `components/shared/context-menu.tsx` |
| Lazy overlay fallback | `components/ui/sheet-fallback.tsx` |
| Close | `components/ui/close-button.tsx` |
| Sheet grabber | `components/ui/sheet-grabber.tsx` |
| Icon button | `components/ui/icon-button.tsx` |
| Bare button | `components/ui/unstyled-button.tsx` |
| Button | `components/ui/button.tsx`, with `buttonClasses` for a control that must stay an anchor |
| Pressable surface | `components/ui/pressable.tsx` and the `press-*` utilities |
| Plot: line, bar, stacked | `components/dashboard/chart-frame.tsx`, with `AreaChart` and `BarChart` drawing its marks |
| Ring: a whole split into parts | `components/dashboard/donut.tsx`: `Donut`, `DonutSkeleton` |
| Steps toward something a person unlocks | `components/shared/requirement-checklist.tsx`: `RequirementChecklist`, each unmet step opening where it is done |
| A panel's one figure over its chart, and the figures that pick it | `components/dashboard/lead-metric.tsx`: `LeadMetricCard`, `LeadMetricSkeleton`, whose strip is `ChoiceGroup` at `variant="tile"` |
| A secondary panel folded to a row that states its reading | `SettingsFoldRow` in `components/settings/settings-section.tsx`; `DashFold` holds its open state in an analytics card and opens on a link to its `anchor` |
| Analytics window | `components/dashboard/period-filter.tsx`: `PeriodFilter`, the page's one `FilterButton` holding the windows as one ladder, at the end of Learn's sections and of Earn's lead figure, kept in the URL by `PERIOD_PARAM` |
| Input, textarea, select, switch, checkbox, slider, field | `components/ui/` of the same name; every range is `RangeInput` (`components/ui/range-input.tsx`), which takes the pointer and draws no handle `[craft §27]`, with `RangeFill` for the painted part |
| A pinned bar's ground | `chrome-plate` (`MOBILE_STICKY_HEADER`): opaque, so it takes no blur `[colour-and-effects §40]` |
| Auto-growing composer field | `components/ui/auto-size-textarea.tsx` |
| Character-limited field | `components/ui/limited-text-field.tsx` |
| Handle field | `components/user/username-field.tsx`, which previews availability and carries the way through when the name is held |
| Password field | `components/ui/password-input.tsx` |
| Code entry | `components/ui/code-input.tsx` with `CODE_CELL` |
| Money amount | `components/ui/money-input.tsx`, parsing through `lib/domain/money/amount.ts` |
| Search field | `components/shared/search-input.tsx` with `SEARCH_PILL`; `components/shared/page-search-field.tsx` for a list page's field over its tabs |
| Exclusive choice | `components/ui/choice-group.tsx`, `components/shared/segmented-control.tsx` |
| Multi-select chips | `components/ui/chip-select.tsx` |
| Pick from a list | `components/settings/pick-sheet.tsx`: `PickSheet` with `mode="many"` (checks, sorted by name) or `mode="one"` (radios in their own order, closing on select), plus `toggles` for switches that combine with any choice |
| Pick list row | `components/ui/check-row.tsx`, a `checkbox` or a `radio`; a chosen row tints and carries a filled check, never a radio dot, and stands `min-h-12` however short its label |
| A held control's reason | `components/settings/reason-note.tsx`: `ReasonNote`, the one line and the one press that frees it |
| A set of people (close friends, story groups, list members, follow groups) | `components/user/people-picker.tsx`, sheeted with its dirty check by `components/user/people-picker-sheet.tsx` |
| A list's order | `components/shared/sort-button.tsx`, which states the order and opens a `PickSheet` in `mode="one"`; a profile's order is the one `GlassFilterPopup` at the end of its `TabStrip` |
| What is known about an account | `components/user/account-about-sheet.tsx`: `AccountAboutSheet`, behind the profile header's info button [permissions-and-trust §16a] |
| A decision on a person's account or content | `components/moderation/case-notice.tsx`: `CaseNotice`, on `/suspended` and in account status, with `AppealForm` its one way to contest it [permissions-and-trust §25r] |
| A console row checked for bulk triage | `components/admin/row-select-toggle.tsx`: `RowSelectToggle`, which never opens the row it sits in |
| Why a pick's files were not added | `components/compose/media-preview-grid.tsx`: `MediaRefusalNotice`, under the media in every composer |
| A message's actions | `components/message/message-menu.tsx`: `MessageMenu`, the reaction row over every action the message allows, each named; a hold, a right-click, the hover control and the add-reaction chip all open this one |
| A notice in a conversation | `components/message/chat-banner.tsx`: `ChatBanner`, a strip in the flow; a thread's stand in `ChatNotices` (`components/message/chat-notices.tsx`) under its bar, never over the log |
| The message composer's row | `COMPOSER_ROW`, `COMPOSER_PILL` and `COMPOSER_SLOT` in `lib/styles/messenger.ts`: `AttachControl` at the pill's leading edge, `ComposerSendSlot` beside it as the microphone that becomes Send, and `VoiceComposer` in the same shape |
| A page's refinements | `components/shared/filter-button.tsx`: `FilterButton`, `FilterSummary`, and `FilterChecks` for a set inside its sheet |
| Everything set about one person | `components/user/relationship-sheet.tsx`, opened from Following or Requested on the profile |
| A status chip, or an entry in a list the person edits | `components/ui/chip.tsx`; `components/settings/word-list-field.tsx` adds to a list of them |
| A daily time limit over the days it would have held | `components/settings/time-budget.tsx` |
| Two-axis choice | `components/ui/feel-pad.tsx`: one thumb sets two dimensions, snapping to nine named places; the feed's feel and the editor's feel are its two axis sets |
| Tabs | `components/ui/tabs.tsx`: `Tabs` for panels it owns, `TabStrip` for a page's one row over content the page swaps; the chosen tab's and choice's fill is `SlidingFill` in `components/ui/sliding-fill.tsx` |
| Collapsible | `components/ui/collapse.tsx` |
| Skeleton bone | `components/ui/skeleton.tsx`; route ghosts in `components/layout/route-skeleton.tsx` and `page-ghost.tsx` |
| Spinner | `components/ui/spinner.tsx` with `LOADING_PADDING` and `SPINNER_CLASS` |
| Skeleton hold | `usePendingBones` with `SKELETON_DELAY_MS` |
| Toast | `components/ui/toast.tsx` |
| Tooltip | `components/ui/tooltip.tsx` |
| Empty state | `components/shared/empty-state.tsx` |
| Not found | `components/shared/not-found-page.tsx` |
| Error | `components/ui/error-alert.tsx`; the message names what refused |
| Confirmation | `components/shared/confirm-dialog.tsx` |
| Who sees a post | `components/compose/audience-sheet.tsx`, opened from the audience chip and the "Audience" row: reach, who may reply on one track of named stops, and what a reader may do with the post (`PostReception`); shorts use it too, their row stating reach and replies (`audienceSummaryKeys`) |
| A model's words before they replace the author's | `components/ai/ai-proposal.tsx`: `AiProposal`, for on-demand rewrites (`RewritePanel`) and the publish-time review alike `[ai §13]` |
| Asking AI, and what AI made | `components/ai/ai-mark.tsx`: `AiInvoke`, the star with its verb; `AiGlyph`, the star inside a row that carries the verb; and `AiBadge`, the AI mark `[ai §1]` |
| An ask of AI | `components/ai/ai-prompt.tsx`: `AiPrompt`, with the send that becomes Stop in `ComposerSubmit` |
| What an answer rests on | `components/ai/ai-sources.tsx`: `AiCitedText`, `AiSources`, `AiVerdict` |
| AI saying no | `components/ai/ai-refusal.tsx`: `AiRefusal`, read by `readAiRefusal` |
| A conversation with AI | `components/ai/ai-thread.tsx`: `AiThread`, hosted by `AiThreadSheet` or a surface's dock under `AiThreadHeader`; an agent's steps are `AiSteps`, each led by its tool's glyph |
| When a post goes out | `components/compose/schedule-picker.tsx`: named moments, an exact minute one fold down |
| Query state wrapper | `components/shared/query-state-wrapper.tsx` |
| Name and handle | `components/user/display-name.tsx` |
| @ask conversation | `components/ask/ask-conversation.tsx`: `AskConversation`, presets then turns over `useAskThread`; `AskSheet` everywhere, opened from the URL by `components/ask/ask-host.tsx`, and the desktop shorts dock (`components/shorts/shorts-ask.tsx`) |
| @ask's sources, in a conversation and under its public reply | `components/ask/source-chips.tsx`: `SourceChips` |
| Why the feed showed a post | `components/post/why-seeing-sheet.tsx`: `WhySeeingSheet`, the reason with Not for me (undone by the feed row's strip, or a toast on a short) and the feed settings; the only way into Not for me |
| A post's context and readers' notes | `components/context/context-note-list.tsx`: `ContextNoteList`; a note is written in `context-note-sheet.tsx` and rated with `note-rating-buttons.tsx` |
| Surface docked beside a short | `components/shorts/shorts-side-panel.tsx`; comments and @ask are one slot |
| Author link on a card | `AuthorLink` in `components/post/post-card-shared.tsx` |
| Sound control on any player | `components/media/sound-toggle.tsx` |
| Playback speed | `components/media/playback-speed-control.tsx` |
| A player's control strip | `components/media/video-control-strip.tsx` |
| A window that floats over every page | `components/media/floating-window.tsx` |
| Link a person wrote | `UserLink` in `components/ui/user-link.tsx` |
| A credit a licence requires | `CreditLink` in `components/ui/credit-link.tsx` |
| Person row | `components/shared/user-list-item.tsx` |
| Avatar, avatar stack | `components/ui/avatar.tsx`, `avatar-stack.tsx` |
| Image crop | `components/ui/crop-stage.tsx` (`useCropFrame`, `CropStage`, `CropControls`); `components/ui/image-cropper.tsx` is the crop as its own sheet or takeover, and the composer's `components/compose/photo-editor.tsx` carries it as its Crop tab |
| Unread indicator | `components/ui/unread-dot.tsx` |
| Glyph that points along the inline axis | `components/ui/directional-glyph.tsx`: `DirectionalGlyph`, `directionalIcon` `[internationalisation §17]` |
| Relative time | `components/shared/relative-time.tsx`, over `formatRelativeTime` |
| A line of facts: who, how many, when | `components/ui/meta-line.tsx`: `MetaLine`, parted by space `[craft §10a]`; `formatSummary` where the line must stay one string, `formatPhrases` when its parts are product copy |
| Page header | `components/layout/page-header.tsx` with `lib/styles/page-header.ts` |
| Glass surfaces, rows, pills, cards | `lib/styles/glass.ts` |
| Waveform | `lib/audio/waveform-shape.ts` |
| Creation tools | `components/shared/create-tool-rail.tsx`: `CreateToolRail`, the one row (a short's on every width, a story's on a phone) or column (a story's on a desk) of every tool both creation editors are reached through, drawn from `lib/styles/create.ts` (with `CREATE_BAR_GLYPH` and `CREATE_NEXT_BUTTON` for both editors' top bars); `components/shorts/editor/context-toolbar.tsx` gives it the selection's tools; `components/shared/trim-window-panel.tsx` is the one trim window `[media §22c]` |
| A subtree in another theme | `components/ui/theme-scope.tsx`: `ThemeScope`; a dialog opened inside it wears its theme |
| Music part and mix | `components/shared/music-part-picker.tsx` for the part a track plays; `components/shared/music-mix.tsx` for the Original ↔ Music balance over `lib/audio/music-mix.ts`; a track being chosen is heard under the clip through `useClipAudition` |
| Haptics | `haptic()` in `lib/utils/haptics.ts` |
| Pull to refresh | `components/shared/pull-to-refresh.tsx` |
| Count badge | `COUNT_BADGE` with the `text-badge` token |
| Player chrome, floating bar, feed actions, shorts rail | `VIDEO_CONTROL_*`, `FLOATING_BAR_*`, `FLOATING_VIDEO_*`, `FEED_ROW_ACTIONS`, `components/shorts/shorts-rail-style.ts` |
| Shared element identity | `layoutId` on both surfaces, named once in `lib/constants/shared-elements.ts` (not yet built) `[signature-motion §7]` |
| Rolling number | `components/ui/rolling-number.tsx` (not yet built) `[signature-motion §20]` |
| Live activity capsule | `components/layout/live-activity-capsule.tsx`, one per app frame, fed by `startLiveActivity` `[signature-motion §30]` |
| AI listening edge | `components/shared/ai-listening-edge.tsx`, one per screen `[signature-motion §39]` |
| Authored text | `components/ui/rich-text.tsx` for a page, `richTextToLine` and `richTextToPlain` for a string `[type §33]` |
| AI text rewrite | `AiShimmer` in `components/ai/ai-working.tsx`, and `AiProgressLine` before an answer's first word `[signature-motion §24]` |

## The overlay family

Three members. Each is a full contract, not a wrapper, and a consumer that renders one inherits
everything in its row.

**`Dialog` and `DialogContent`.** Portals to `document.body`. Presents as a bottom sheet under 640px,
a centred panel above it, and a right-docked side panel or a fullscreen takeover when asked. Owns: the
scrim and its fade; the CSS presence that keeps the exit mounted for exactly `--ovl-dur`; the
`data-mode`, `data-state` and `data-interactive` attributes the stylesheet animates; the focus trap
and its restore, only while topmost and only when modal; Escape, only while topmost; the scroll lock,
held through the exit so the page behind does not snap while the sheet is still sliding; one history
entry per open dialog through a shared `popstate` coordinator that closes only the topmost; the
nesting ladder, where a covered dialog goes `inert` and `aria-hidden` and each depth steps 100 above
the last; the width, from `DIALOG_SIZES` into `--dialog-max-w`, with `--dialog-h-cap` declared on the
panel beside it; the keyboard, lifting the panel on `translate` through `useKeyboardLift`; and the grabber with
drag-to-dismiss on the phone sheet. Its name comes from `DialogCompactHeader`'s registered title id,
so a visible title is never repeated as an `aria-label`. `nonModal` drops the scrim, the focus claim
and the lock, and keeps Escape, Back, the drag and the close.

**`MenuSheet`.** The one mobile action sheet: scrim at the floating rung, grabber, drag-to-dismiss,
focus containment without an autofocus claim, scroll lock, and a `--sheet-h-lg` ceiling it hugs
upward from. It takes its presence state from the caller, so the caller supplies Escape; the sheet
parks its own history entry while it is open, so Back closes it wherever it is drawn. Nothing else draws a phone action sheet.

**The anchored popover, `DropdownMenuContent`.** On a phone it is a `MenuSheet`; on a fine pointer it
is a portaled fixed panel. Owns: placement from `placeAnchoredPopover`, measured against the viewport
with `POPOVER.edgeMargin`, `POPOVER.gap` and `POPOVER.minDropHeight`, never a pixel cap; a transform
origin at the trigger's corner, not the panel's, so a panel shifted off an edge does not grow from
nowhere; re-placement on scroll and resize, once per frame, capture phase so inner scrollers count;
`--ovl-pop-y`, which points the 4px nudge the way it opened; a scroll lock; Escape with capture and
`stopPropagation`, so a menu inside a dialog closes alone; arrow-key roving for a menu and a focus
trap for a popover; focus restored to the trigger only if focus is still inside the closing menu; a
transparent dismiss layer one rung below the panel, so an outside click closes without activating
what it landed on; and a solid surface, `GLASS_POPOVER_SOLID`, because the open scales and scaling a
blur re-rasterises it. Placement state survives the close, because the node is still mounted for its
exit. A panel that stands beside its trigger rather than under it, a short's More beside its rail, is
placed by `placeBesidePopover` on the same margins: it rises to the trigger or hangs from it, whichever
holds it whole, and past the window's height it scrolls inside, so no row is ever below the fold.
Nothing inside an anchored panel unfolds: it is placed once, at its whole height.

**Results under a field, `AnchoredResults`.** The anchored popover's placement, surface and motion for
results that hang from a field the person is still typing in, such as `SidebarSearch`. It never takes
focus, locks the page or claims Escape, a press inside it never blurs the field, and the arrows walk
it, ArrowUp past the first result handing focus back to the field.

Every menu row is `MENU_ITEM`, whichever member drew it.

## Enforced by

- `__tests__/fitness/an-overlay-inherits-its-behaviour.test.ts`: rules 12 and 13, through the
  hand-built-overlay, sheet-height-consumer and dialog-own-width baselines.
- `__tests__/fitness/a-close-is-one-control.test.ts`: rule 14, through the close-glyph and
  dismiss-button baselines.
- `__tests__/fitness/a-recipe-lives-in-styles.test.ts`: rules 1, 2 and 20, through the chrome-module,
  recipe-export, class-recipe and duplicate-component-name baselines.
- `__tests__/fitness/an-overlay-owns-its-escape.test.ts`: rule 12's second half, the key `MenuSheet`
  leaves to its caller.
- `__tests__/fitness/a-fallback-is-the-shape-it-precedes.test.ts`: rule 11, through the
  fallback-panel-mismatch and rows-with-no-list baselines, and a hard floor under the centred dialog
  in front of an anchored menu.
- `__tests__/fitness/an-icon-means-one-thing.test.ts`: rule 3, through the glyph-outside-the-catalogue
  baseline.
- `__tests__/fitness/a-field-shows-focus-once.test.ts`: rule 5.
- `e2e/a-chart-tick-is-drawn-where-it-is-labelled.spec.ts` and `__tests__/utils/chart-scale.test.ts`:
  rule 24. jsdom lays out no SVG and fires no `ResizeObserver`, so a plot's geometry — label centres
  against grid lines, tick spacing, label collisions — is only ever measured in a real browser.
- `__tests__/fitness/a-tab-lives-where-its-strip-does.test.ts`: rule 9.
- `__tests__/fitness/a-page-has-one-row-of-choices.test.ts`: rules 27, 28 and 29, through the
  pill-row-as-page-chrome baseline: a pill row in a file that draws no sheet of its own.
- `__tests__/components/shared/filter-button.test.tsx`: rule 28's count, sheet, summary and Clear.
- `__tests__/fitness/a-lazy-overlay-waits-for-its-first-open.test.ts` and
  `a-lazy-sheet-waits-to-be-opened.test.ts`: rule 10.
- `__tests__/fitness/a-tap-target-is-a-control-not-a-wrapper.test.ts`: rule 19.
- `__tests__/fitness/the-two-desktop-composers-carry-the-same-features.test.ts`: rule 25, through the
  shared option surfaces each frame renders and the single overflow menu.
- `__tests__/fitness/a-name-and-its-handle-stack.test.ts`: rule 15.
- `__tests__/fitness/an-author-is-a-link.test.ts`: rule 16.
- `__tests__/fitness/a-link-a-person-wrote-passes-no-rank.test.ts`: rule 26.
- `__tests__/fitness/a-count-comes-from-count-of.test.ts`, owned by `[internationalisation]`: rule 17's
  count half.
- `__tests__/fitness/an-empty-state-is-the-primitive.test.ts` and
  `an-error-names-what-failed.test.ts`: the empty and error rows of the table.
- `__tests__/fitness/controls-are-operable.test.ts`: rule 4, plus every control having a name.
- `__tests__/fitness/every-control-has-a-test.test.ts`: a primitive ships with a spec.
- `__tests__/theme/ink-context.test.ts`: a primitive's ink matches the ground it is drawn on.
- `eslint local/no-raw-button`, `local/no-raw-form-control`: rule 6.
- `eslint local/no-hand-rolled-choice-row`: rule 7.
- `eslint local/no-hand-rolled-section-label`: rule 8.
- `eslint local/no-unpositioned-tap-target`: rule 19's positioning half.

## Traps

- Rule 10: `next/dynamic` renders its `loading:` fallback even at `open={false}`, so a lazy sheet
  flashed a modal on every page load; and an unconditional `ssr: false` sheet regenerated the whole
  route tree on load.
- Rule 11: a centred dialog standing in for an anchored menu was the whole reported flash.
- Rule 12: the anchored popover clears its placement state on unmount, not on close. Clearing it on
  close sent the exit to the centre of the screen, because the node is still mounted for its exit.
- Rule 12: a global Escape handler mounted in a layout owns the key on every page. The live stage
  owned Escape with no session, so Escape on any dialog also left the page. A listener exists only
  while its surface is on screen, and the topmost claims the key first.
- Rule 12: a sheet or drawer never buries its own history entry, and a menu close never calls
  `history.back()`. A phone-only `history.back()` on menu close undid the navigation the menu had just
  made, which is why a `DropdownMenuItem` that leaves the page declares `href` and the primitive routes it
  through `closeOverlayThenNavigate`; any other item runs in the tap.
- Rule 12: an exiting overlay stops hit-testing the instant the close begins, through
  `data-interactive='false'`, or a leaving surface swallows the next tap.
- Rule 12: the crop editor's takeover was a hand-built `fixed inset-0`, opened from the inline composer's
  card, whose `backdrop-blur` makes it the containing block for a fixed descendant. At 1440x810 the
  "full-screen" editor was 638x906 at the card's own origin, starting 34px above the top of the window —
  so Apply sat off screen — and at 3840x2160 it was a 698px panel in the middle of an empty desk.
- Rule 13: the shorts panels reserved a tier while rendering a content-height sheet, so five panels
  showed a third of themselves through a 167px window.
- Rule 19: `tap-target` on the on-media author pill drew a 44px `::after` over its own name and
  location buttons, so every phone tap landed on the pill.
- Rule 9: `<Tabs defaultValue>` eats every deep link, and a `TabsTrigger` that sets its own variant
  ignores the list it stands in.
- Rule 27: a bolder chosen tab is a wider tab, so every tab after it slid sideways under the finger.
- Rule 28: Explore's refinements were five pill menus on a second row under its tabs; on a phone "All time"
  and "Any format" wrapped inside their own pills, and the row carried more chrome than the results.
- Rule 5: a bare input kills the global `:focus-visible` outline, which is why the ring lives on the
  shell rather than the field.
- Rule 25: the inline composer shipped without drafts, scheduling, trial mode or the copilot for as
  long as the modal had them, because each toolbar was edited where it was needed and never compared.
  Two surfaces over one hook drift silently; only a test that reads both catches it.
- Rule 20: a component name defined twice is resolved by the import path, and the wrong one looks
  identical in review.
