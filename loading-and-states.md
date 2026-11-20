# Loading and states

Every surface in the product is in exactly one of four states, and this document owns all four plus the long
tail of states nobody designs until they happen: the post deleted while it is open, the invite already spent,
the file too large, the account suspended. The feeling it protects is that the app is never confused. A person
always knows whether they are waiting, whether there is nothing, or whether something refused them, and which
one it is is never in doubt for a single frame.

## Owns and defers

**Owns.** The four states and their ordering. Ghosts, bones and what may shimmer. The hold before a skeleton
earns its place. Seeded pages and what hydration keeps. The empty state, the error state, the not-found state.
Every long-tail refusal a person can land in, what it says, and the one step out of it.

**Defers.** Why data is slow and what it costs, `[speed]`. What happens with no network at all, `[resilience]`.
The crossfade's duration and curve, `[motion §18]`. The words themselves, `[copy-and-voice]`. Toasts as a
delivery mechanism, `[feedback]`. What a lock may and may not conceal, `[permissions-and-trust]`. Live-region
announcements, `[accessibility]`. Gate copy and upgrade paths, `[permissions-and-trust]`.

## Rules

### The four states, one at a time

1. **Four states, never two at once.** Loading, empty, error, content. `QueryStateWrapper` resolves them in one
   fixed order: locked, then pending, then error, then empty, then the content. A surface that draws its own
   ordering will get it wrong.
2. **A locked surface outranks every other state.** A query the tier does not allow never resolves, so pending
   would hold a skeleton open forever. `lockedFeature` short-circuits to the gate `[permissions-and-trust]`.
3. **Pending is `isPending`, never `isLoading`.** `isLoading` is false for a query that is disabled, gated, or
   already holding cached data, so a surface wired that way paints empty first, then the skeleton, then the
   list: three states in three frames for one load.
4. **An error never falls through to empty.** An outage read as "nothing here" is the most expensive lie the
   product can tell. A failed read always shows a failure with a retry.
5. **A list already on screen keeps its content through a failure.** When there is something to show, the
   failure and its retry go to the foot of the list; only a surface with nothing loaded is replaced.

### Ghosts and bones

6. **A bone stands in for content on its way from a server, and for nothing else.** Never chrome, never a
   form, never a picker, never a menu, never a setting. A lazy panel whose body is pure UI holds its space at
   final size instead `[components §11]`, and every fallback declares which it is rather than inheriting a
   default, because the default is bones.
7. **A ghost is the content's exact shape, so nothing moves on arrival.** A line bone sits inside the real
   text's line box, taken from that size's own line-height token so the viewer's text scale carries through.
   A card ghost is the card's height, a grid ghost has the real column count, a row ghost is built from the
   row's own layout recipe. Layout shift after first paint is zero. A row that has never rendered is held
   the same way: `.feed-item-cv`'s `contain-intrinsic-size` is one `--cv-intrinsic-h` per card height, and a
   card whose frame changes at a breakpoint carries that breakpoint's estimate too — `.feed-item-cv-short` and
   `.feed-item-cv-video` follow their frame to square at `lg` `[media §33]`, while an image row keeps its 4:5
   floor at every width. A kind whose frame is not the image column's gets its own class, never the nearest
   one: a video row estimated at 4:5 was up to twice the height it takes, which shifts every card below it.
7a. **A list ghost's height belongs to its column, never to its row count.** How many rows an account has
    is exactly what the read has not answered yet, so any count typed into a ghost is wrong in one direction
    or the other: `/spaces` reserved twelve rows for an account with five and stood nearly twice the height
    of the list that replaced it. The strip takes what is left of the ghost's own column —
    `flex-1 min-h-0 overflow-hidden`, so it clips rather than promises — and the count is only the ceiling
    that fills it. A list that IS the page under the header takes the screen directly, off
    `--page-header-h`, because there its column is the viewport.
8. **Chrome is drawn, not boned.** The header, the back control, the tab strip and any title the page knows
   before its data are real in the ghost, and the ghost's title is the title the page will show. Back works
   from the ghost, so leaving mid-load lands where leaving after it would.
9. **What a page knows, it draws.** An invite or call landing screen renders every fixed word and holds only
   its two actions until the answer arrives, because only "does this link still redeem" was ever unknown.
10. **A full-screen fallback carries the ground of the surface it precedes.** A capture or playback screen
    waits on a dark frame with the silhouette of its own controls, because a light panel in front of a camera
    about to be black is a flash. A surface whose final shape cannot be predicted ships no fallback at all.
11. **A ghost is written once.** The route's `loading.tsx` boundary and the page render one ghost component. Two
    copies drift, and the person watches one skeleton get replaced by a different skeleton before content lands.
12. **One shimmer.** `skeleton-shimmer` is the only loading texture in the product `[colour-and-effects]`. No
    pulse, no second keyframe, no gradient of a surface's own.
13. **A spinner is for a surface with no shape to promise.** A full-bleed capture or playback frame gets
    `MediaSurfaceSpinner`; an appended page of a list gets `CenteredSpinner` with a `LOADING_PADDING` step.
    Anything that resolves into laid-out content gets bones instead.
14. **A pending surface announces itself once**, at the boundary that is genuinely pending, not on every child
    `[accessibility]`.

### The hold

15. **No bone before the hold ends.** `usePendingBones` waits `SKELETON_DELAY_MS` `[tokens]`, and content that
    lands inside the hold paints with no skeleton having existed.
16. **The hold reserves height without painting.** `state-held` is laid out and invisible, so a query that
    resolves late still has its space held and a query that resolves early costs nothing.
17. **Only a surface that showed bones crosses back.** `state-fade-in` runs on the children of a surface whose
    skeleton was actually seen; a surface that never showed one is untouched at rest, because a crossfade with
    nothing to cross from is a flash.
18. **A navigation waits before it ghosts, and the wait is not a React commit.** Under `NAV_GHOST_GRACE_MS`
    a warm push has already landed and a ghost would only flicker; past `NAV_GHOST_MAX_MS` the ghost gives the
    page back, so a navigation that never resolves cannot leave the app permanently mid-load. The grace is a
    timer that puts `NAV_GHOST_REVEALED_CLASS` on the frame, counted from the press, because React cannot
    re-render the frame while the destination is suspended: a second commit to reveal the ghost lands after
    the wait it was meant to cover. Both sides of the swap are plain inherited `visibility` off that one
    class, never an animation — `.nav-ghost` and `.nav-ghost-outgoing` are `display: contents`, which not
    every engine animates, and a swap half of which never ran painted both pages at once `[traps]`. The ghost
    reaches the frame on the transition lane, so the press never pays for the skeleton's render; it leaves on the
    urgent one, in the commit that lands the page, but a ghost that has been revealed is held for
    `NAV_GHOST_MIN_MS` so a commit just after the reveal never flashes it. A press that moves the page itself, a
    section swipe, asks for no ghost.
19. **The ghost stands over the old page in its own frame, and unmounts nothing.** `PendingRoute` hides the
    outgoing page rather than tearing it down, so its state, its scroll, its height and its playing media all
    survive a navigation that is abandoned. The ghost's box is fixed over the area the shell owns —
    `SIDEBAR_RAIL_INSET`, `TOP_STRIP_OFFSET`, and `RIGHT_PANEL_OFFSET` when the panel on screen is holding
    that gutter — and is sized by its own destination's `pageColumn`, because the frame underneath it belongs
    to the page that is still being read `[layout §7]`. Every navigation runs inside a transition this app owns — `useNavPush`, which
    `useLinkPress` gives every `AppLink` — because that is the only thing that stays pending until the
    destination paints: the router's own pathname and link status both answer the destination in the first
    frame of the wait `[traps]`.

### Seeded pages and hydration

20. **A seeded page has no loading state.** Stateless and public pages render their content on the server; there
    is nothing to wait for and therefore nothing to ghost.
21. **Hydration keeps the server paint.** The seed becomes `placeholderData`; it is never thrown away and
    refetched, and a warm reload never paints later than a cold one. The first client render derives exactly
    what the server did: nothing in it reads the render zone, the runtime's locale or the clock, so a seeded
    list keys its days in UTC and measures age against the query's `dataUpdatedAt`. A re-ranked first page that
    lands over it keeps the rows its first screen showed, in their order, and ranks the rest around them
    (`keepPaintedHead`).
22. **Every route's ghost matches the shell it precedes.** `RouteGhost` is one registry from pathname to the
    destination's real ghost, and each route's `loading.tsx` renders the same component the registry does. A
    route with a shape of its own is in the registry as well as in its own boundary, or its pending frame is a
    generic row list that its boundary then replaces. A boundary that composes its ghost in place (donate,
    billing, the invite landings) is registered as itself; `routeGhostFor` is the registry's one read. Each
    ghost is its own chunk, warmed with its route; only the tab bar's destinations ship with the shell, and
    `PendingRoute` arms the reveal only once the ghost is drawn, so a chunk still on its way leaves the old
    page up. A group boundary reads the destination the press announced rather than `usePathname`, which still answers the route being left: `app/(main)/loading.tsx`
    takes `useNavDestination`, or a full-bleed capture route is ghosted with the shell it is leaving. Explore
    keeps its chrome's boxes and not its chrome: `ExploreGhost` is bones laid out from
    `explore-chrome-layout`'s own tokens, so the one ghost serves both the pending frame and the boundary and
    the real bar lands in the box the bones held.
22a. **A row that may be absent is decided before first paint.** Home's stories row is drawn only while it holds
    something to watch (`storyStripShown`); the server reads that verdict with the page and hands it down,
    the browser's own groups reach the same verdict (`storyGroupsShown`), and the route's ghost follows the
    last verdict it left in `COOKIE_NAMES.storiesShown`, so the feed under it does not move when it goes.

### Empty

23. **An empty state is sized by what holds it.** `EmptyState` at `page` owns a screen; at `section` it is a
    band inside a panel that already has a heading, where the icon tile and the page padding are ceremony
    around nothing. A panel declares its treatment once, and its failure state inherits it.
24. **Empty appears only after data has settled.** Never during the hold, never for a query that is disabled or
    gated, and never where a failure is the real answer `[copy-and-voice §20]`.
25. **Empty is not the same as filtered-to-empty.** A list emptied by a filter says so and offers the clear, not
    the create.

### Error

26. **A null row and a thrown query are two different states.** A row that is not there is not-found; a read
    that threw is an error. A surface that collapses them turns every deleted post into an outage, or every
    outage into a deletion `[copy-and-voice §15]`.
27. **One refusal wins, in a fixed order.** A file the app itself rejected, then one the device will no longer
    read, then connectivity, then an abort, then the auth code, then the HTTP status, then the guard's own
    raised reason, then the SQLSTATE. The first
    that answers is the one the person reads, so a dropped connection never surfaces as a database code
    `[copy-and-voice §14]`. A refusal that is a designed product rule
    reads as that rule, never as `error.notPermitted`, which reads as a broken account.
28. **A failure is filed once.** `describeError` during a render, `surfaceError` where the failure is handled.
    A component that reports as it draws files the same crash on every pass.
29. **The reference is derived where it is shown.** The eight characters come from the failure's own
    fingerprint, so the surface quotes the same reference whether or not the report itself got out.
30. **A failed read always carries a retry the person controls.** No surface leaves a failure with nothing to
    press, and no retry runs on a timer the person cannot see.
30a. **Empty, error and gone are one card.** `EmptyState`'s tile, one line and one action, at `page` or
    `section`: `QueryStateWrapper`'s failure, `ErrorFallback` and `NotFoundPage` all render through it, and
    the one retry verb is `button.retry`. The line is the refusal itself; a surface that hands over no
    `failure` has the newest failed read mounted read for it (`useLatestReadFailure`). A route's crash keeps
    its `PageHeader` and back (`ErrorPage`), because some routes hide the tab bar.
31. **`ErrorAlert` is the inline shape**, in the error tone for a refusal and the warning tone for a condition.
    A full-surface failure uses the boundary's fallback, and a failure caused by an action uses a toast
    `[feedback]`.
32. **Every route group has an error boundary**, and a crash inside the shell replaces the page, not the app.
    A refusal is not a crash: a server render that throws one hands the whole route to the boundary's shrug
    in place of the state that names it, so a server component answers a refused read or write with a
    redirect or a designed state and keeps the throw for what nobody designed (`readJsonObject`, not a
    decoder that raises).
32a. **An app-wide fallback asks for nothing below it.** `ErrorBoundary` at the root and `global-error.tsx`
    draw with every provider the app mounts already gone, so they take their words from `useLocaleStore`
    and their sentence from `describeError`, never from a context. A fallback that cannot render replaces
    the crash the person hit with a blank document, and files its own fault in place of theirs.
32b. **A hold is keyed on what the server already knows.** The request's verified viewer reaches the client
    in the first HTML, so a surface that decides whether to hold a viewer-only row by reading a store
    instead begins holding it a frame after paint and moves the card it sits in. The same answer, a render
    later, is a layout shift.

### Not found

33. **One not-found screen.** `NotFoundPage` names what is missing in the person's words, keeps the header so
    back is never lost, and offers exactly one link to the parent the thing lived in.
34. **Every "gone" surface is a kind, not a copy.** A surface that hand-rolls its own missing-thing copy will
    drift from the kinds that did not. Post, space, sound, conversation and place are kinds like the rest; a
    parent known only at runtime, a post's author, is handed in as `parent`.
35. **Private is never dressed as missing.** A thing that exists but is not for this viewer says so, names the
    one action that would open it, and is held behind its own pending guard so a follower is never shown
    "this account is private" for a frame while the follow state resolves.

### The long tail

36. **Every refusal a person can reach is in the catalogue below.** A state that is not in it is not shipped,
    and a refusal nobody designed reaches the person as a shrug.
37. **A refusal is a surface, not a toast, when the person is stuck in it.** A blocked composer, a suspended
    account, a spent invite and a locked panel each replace what they refuse and carry the one action out
    `[copy-and-voice §17]`. A toast is for a refusal the person can simply try again.
38. **A state that changes underneath a person is noticed.** A block, a deletion, a suspension or a moderation
    decision that lands mid-session reaches the surface through its own signal, not on the next accidental
    refetch. Where only a poll can do it, the poll is named and states why an event will not
    `[speed §30]`.
39. **A refusal a second attempt cannot fix says so once.** A block, a revoked permission or a moderation gate
    answers identically every time; the control does not offer a retry it knows will fail `[resilience §21]`.
40. **A limit's surface shows the number it hit**: the cap in the file's own units, the reset time, the
    accepted formats. A limit stated without its number is a dead end wearing a sentence.
41. **A post that is still finishing says so on itself.** `PublishProgressBar` is the one shape, drawn with
    `Meter` `[components §23]`: the bytes by the byte while they upload, then the product's one loading
    texture while the server's own work is outstanding, then nothing. Both halves read real state — `upload_percent` on the seated row and
    `my_posts_processing` on the server's own queues — so nothing on it is ever a timer `[speed §30]`.
    Attachments upload as they are attached, so the row is seated with the fraction already measured
    rather than with zero: a bar that reads 0% for a publish whose bytes are long gone is a lie too.
    The percent is reported from the transfer's own callback into a ref the composer leaves behind —
    `reportUploadPercent` over `publishingGhostRef` — never from an effect in the surface the tap closes.

42. **A story that is still going up says so on its ring.** The rail is where a story lives, so the ring
    the tap lit is what carries the publish: a grey track the accent arc fills from `upload_percent` on
    the pending row, one arc across however many slides the story has. The composer closes in the tap's
    own frame and nothing is held behind a scrim; a refusal clears the ring and offers the story back.
    The publish can also be taken back: `cancelPendingStory` aborts the signal every byte of it rides
    on, clears the ring and says nothing, because a story the author withdrew is not a failure
    `[speed §31]`. `(not yet built)`: nothing on the ring offers that yet — the rail has no
    long-press surface and the viewer never opens a promised story, so the affordance needs a shape
    before it has a control.

43. **A count a person cannot see all of says how many they cannot.** A reply the viewer may not read —
    a protected author they do not follow, an author the post's author restricted, either side of a block
    or a mute — is counted on the card and dropped from the thread, so a card that said 3 opened on 2 with
    nothing to explain it. Once the last page is in, every comment surface — the sheet, the detail page and
    the shorts panel, at the root and dived into a comment alike — ends with `HiddenRepliesNote`, which says
    `comments.hiddenReplies` and nothing when there is nothing to say, and never leaves a number that does
    not add up. `hiddenReplyCount` is the one derivation. A follow count is the other case: an account no
    list draws for this viewer (a QA account, a deactivated or suspended one) is not counted at all, so
    a profile's Followers and Following read `USER_PROFILE_SELECT`, whose counts are the lists' own.

## The long-tail catalogue

What a person sees, and the one step out. `(not yet built)` marks a state the code does not produce today.

| State | What the person sees | The one next step |
|---|---|---|
| Post deleted while viewing | Not-found copy naming the post, on the post's own surface | Back to the parent |
| Post deleted by you | It leaves the list in the same frame, and a toast holds the undo for `TIMING.undo` | Undo |
| Post edited while replying `(not yet built)` | Nothing today: the reply lands against a version the person never read. Needs an inline notice above the composer naming the edit, with the current text one tap away | Read the new version |
| Quoted post deleted or withheld | The quote keeps its frame and names the post as missing, rather than vanishing from a card that drew it | None |
| Message deleted in a thread | An italic placeholder that keeps the bubble's rhythm, live over realtime | None |
| Backup key not on this device | The Chat backup rows name the device as holding no key, and the switch cannot read "on" for it `[permissions-and-trust §20a]` | Restore here, or Start over |
| Backup key out of date on this device | The same rows name the key as out of date, because the account's wrap names a different one | Restore here, or Start over |
| Backup state not yet read | Every control on the card is held closed while the wrap is read and any legacy blob merged in, so nothing is pressed against a state that has not settled. No copy changes: the card is already at its real shape | None; it settles on its own |
| Backup state could not be read | The card names the failed read, says nothing changed on this device, and keeps the switch out of it — a read that failed is neither "no backup" nor "your key is fine" | Try again, on the card |
| Backup restore locked out | The wait, in the restore box itself, counting down, with Restore closed while it stands. A refusal that IS the wait says nothing else | Wait; the box says how long |
| Encrypted history this device cannot read | The thread's banner names which key state it is in and carries that state's one action. In `sync` this device already holds the key, so the only action is to ask the server again, and where there is nothing to ask with no control is drawn | Restore, ask again, or Not now for a week |
| Blocked mid-thread | The composer is replaced by one line naming the block. `(not yet built)`: the line must distinguish who blocked whom, and arrive on a signal rather than on the next stale read | Unblock, where the viewer is the blocker |
| Private account | Lock, "this account is private", held behind the follow-state guard so it never flashes at a follower | Follow |
| A suspended account's profile | The handle names the state on the profile's own surface. It never falls into the not-found page a handle nobody has produces. The account's own view of itself is unchanged | Back home |
| Private space | "This space is gone or private": the RPC returns null for both, so the line allows either. `(not yet built)`: a distinct not-permitted state | Back to Spaces; request to join once it is distinct |
| Private post | "This post was deleted or is private": the read cannot tell the two apart, so the line allows either and the one link is the author's profile, which says whether it is theirs to withhold | The author |
| Invite expired, revoked, spent | The reason, on the invite's own landing screen, with the screen's fixed words already drawn. Call links distinguish revoked, expired and full; space invites cannot, by design, and say the link is no longer valid | Ask for a new link |
| Rate limited | The wait, with its unit, in the surface that refused. A login lockout counts down in place | Wait, and the control says how long |
| Sign-up: the email already has an account | The alert names the account and points at sign-in, which the screen already carries | Sign in |
| Sign-up: the password the server will not take | A rule in the field's own checklist, graded as it is typed, and create stays closed until it holds. `PASSWORD_MAX_BYTES` is bcrypt's 72 bytes, not the form's old 128 characters | Shorten it |
| Sign-up: the security check never loaded | An extension or a blocked origin left Turnstile with no token, so nothing is sent and the alert says to reload. The wait it gives up after covers the widget answering alone: once a challenge is on screen the scrim carries `CAPTCHA_CHALLENGE_ATTR` and Turnstile's own timeout governs, because a deadline that counts the person's time turns a solvable check into this alert | Reload the page |
| Sign-up: the security check expired | The token was spent or timed out; the retry mints a new one rather than resending it | Try again |
| Sign-up: new accounts are closed | The provider or the signup itself is switched off, named as the product's state rather than the person's mistake | None |
| Sign-up: two attempts overlapped | Only reachable from a second tab: one attempt runs at a time within a screen `[arrival §21]` | Try again |
| A draft's footage no longer reads | Each stored video is read at open, before the editor mounts, so nothing fails in three places later. A draft whose own video no longer reads does not open and says so; one whose added video no longer reads opens, with `LostFootageBar` over the stage until it is back | Select the video again, under the same clips |
| A draft link that points at nothing | `NotFoundPage` of kind `draft` in place of Make, decided on the server; a read that failed still opens Make, whose own open says it could not | Back to Make |
| Handle held for someone | The field names the name as held and carries the request in the same line `[input-and-forms §25]` | Request it, in `ReservedNameClaimSheet` |
| Handle request already open | The request's own waiting state, in place of the form it would file a second time | Wait; the decision arrives as a notification |
| File too large | The cap in whole megabytes, for the category the file is in | Choose a smaller file |
| A picked file refused | `MediaRefusalNotice` under the media names each reason once and stays until the next pick or a dismiss, since the person is looking at the media, not a toast | Pick another file, or dismiss |
| Unsupported format `(not yet built)` | "Unsupported format" today, which names nothing. Needs the accepted formats, the way the video path already does | Choose an accepted format |
| Video this device cannot convert | The codec, named, and that the clip uploads unconverted — said at attach, before a byte is sent | Export it as H.264 MP4 |
| Video longer than the tier converts | The same, carrying the tier's own minutes | Trim it under the limit |
| Empty file | One line naming the file as empty | Choose another file |
| Suspended or banned account | `CaseNotice` for the account's case: what the restriction does, the ground it rests on, whether a check or a person decided, the date it lifts, and a moderator's own words where they wrote any. An account whose decision predates the record reads its state, date and reason | Appeal, with a minimum-length counter, or sign out |
| `/suspended` signed out, with no appeal link | Where a decision is read: signed in, or through the link its email carried. Never a failed read with a Retry, and never a sign-out offered to no session | Sign in |
| A suspended account's refused write | Every write the account attempts is refused, so the line names the account's state rather than the value — and a flow that cannot continue without the write lands on the card above instead | Appeal, on `/suspended` |
| Appeal already filed | The appeal's status, the date its answer is owed, and the resolution note once there is one, in place of the form | None while under review; a denial names the redress outside Surf |
| A decision on the account or its content | Account status in settings lists every decision, open first; one opens `CaseNotice` at `/settings/account-status/<case>`, reached from its notification, from a held post and from `/suspended` | Appeal, once per decision |
| A decision that cannot be found | `NotFoundPage` of kind `decision`, back to account status | Back to account status |
| A profile field held | The notification names the field; account status shows the held copy, the ground and the appeal. The field reads empty to everyone else and the value is kept, never deleted, so a reversal puts it back. A held picture opens from its private copy through a short-lived link, its box held in the one shimmer while the link is read | Appeal, or change the field |
| A report filed without an account | The reference the report was filed under, and whether the outcome will reach the address given | None; the outcome arrives by email when an address was left |
| A report link that points at nothing | The refusal on the link field, in the field's own error slot | Paste another link |
| A report whose item was deleted first | The filer's notice says the report closed because the item was deleted, never that a person found nothing | None |
| A tone reading of a thread that could not run | The author of a thread past twelve replies reads that its tone could not be read, in the quiet tone, never a silence that passes for calm | Retry, or dismiss |
| Moderation pending, in a space | A held-for-review notice in the warning tone on the post itself, visible to the author and that space's moderators only | Wait, or read the rule |
| Moderation pending, elsewhere | The author's own row wears `PublishProgressBar` until the review clears, and nobody else sees the media until then; words alone never wait | Wait; it settles on its own |
| Held by an automated scan | The item stays where it is, hidden from every reader but its author and staff, with its media kept; the author's notice names what is held, shows a thumbnail of it, and carries the appeal. In the feed the author reads that a person will review it and opens the decision from it; anyone else reads only that the media is hidden | Appeal, or wait for a moderator |
| A post still being processed | The same hairline, in the one shimmer, while the author's transcript, description or scan is outstanding — never a percentage invented for it | Wait; it settles on its own |
| Quota reached | The ceiling, the tier that lifts it, and when it resets | Upgrade |
| Feature not in the tier | The panel's place is reserved and reads "available with {tier}" | Upgrade |
| A chat summary nobody has asked for yet | Only this person's devices can read a chat, so nothing reads it on its own: a busy room says how much was missed, and a quiet DM's recap carries one control naming AI. The first press states that the chat's recent lines leave this device once and are neither stored nor trained on, and nothing is sent until that is agreed. Below the plan the same control wears its lock and opens the plans. A failure is named where the answer would have been, beside a DM's own recap; a refusal is named as final | Retry, or put the offer away |
| Device link asked for | A device with no key of its own says it is fetching the older chats, with the number it is showing and the countdown; the recovery key is the second control, never the first | Allow it on the other device |
| Device link being asked | The device that holds the key names who is asking and the number that device shows, and answers with one press; drawn only where this device can actually seal | Allow, or not now |
| Device link waiting | The offer's QR, its typeable code, the match code to compare and the countdown, with Cancel, which retracts the one-time offer rather than only closing the sheet | Approve it on the other device, or cancel |
| Device link sealed | The key is written but the other device has not taken it: "Sent to the other device" until the relay row is consumed, and only then the linked confirmation | Wait, or close |
| Device link needs the account | An offer that arrived as a link takes the account's own password, or the typed word where there is none, before anything is sealed | Prove it, or cancel |
| Device link approved | The linked confirmation on the asking device; on the approving device the confirmation of delivery, and the recovery key held on screen when this approval is what minted it | Done, or save the key |
| Device link expired | The offer is spent and cannot be reused; the state says so and the retry publishes a fresh keypair and address, never the dead one | Try again |
| Device link failed | A blob that would not open, or a relay write that did not land, named as a failure rather than left polling | Try again |
| Device link code dead | A code that is malformed, or one whose offer is gone or already claimed, named on the approval surface instead of toasted out from under a handoff | Try again, back to the code entry |
| Device link needs restore | The approving device holds no chat key, so it refuses to seal one. The state names that and carries a one-box restore that re-submits the pending code on success | Restore on this device, or delete the saved history and link |
| Device link stale key | The saved history was replaced, so this device's key no longer opens it. Same surface, its own sentence, so it never reads as a wrong passphrase | Restore on this device, or delete the saved history and link |
| Session expired | Nothing: the token refreshes and the action completes `[resilience §33]` | None |
| Offline | The banner, plus a refusal naming the network on whatever was attempted `[resilience §2]` | Retry when back |

## Enforced by

- `__tests__/fitness/every-route-boundary-is-its-ghost.test.ts` covers rule 22. Walks every `app/(main)`
  `loading.tsx`, fails a route `routeGhostFor` does not register, and, outside the two-column
  shells, one it registers as a different component from the one its boundary draws.
- `__tests__/utils/sort-replies.test.ts`: rule 43, for the hidden count being what the card promised
  beyond what the last page delivered, and nothing before the last page.
- `__tests__/components/post/a-thread-says-what-it-hides.test.tsx`: rule 43, for every comment surface
  ending with the note rather than one of them alone, and for the note winning over the empty state when
  every reply is hidden.
- `__tests__/hooks/use-shorts-comments.test.ts` and `__tests__/hooks/use-post-page.test.ts`: rule 43 on the
  shorts panel and the detail page, for a dive counting the comment it opened rather than the root.
- `__tests__/fitness/a-surface-shows-one-state-at-a-time.test.ts`: rules 1 and 3. Flags any `isPending` fed from a
  query's `isLoading`, and any query's `isLoading` read as a render gate — a ternary, a guard or a conjunct —
  anywhere in `app/` or `components/`, with a named exemption per verified non-query hook. Re-checks its own
  scanner and asserts the primitive still documents the contract.
- `__tests__/fitness/an-error-names-what-failed.test.ts`: rules 26 and 27, since fencing the shrug keys to the
  mapper is what forces a surface to tell absence from failure. Owned by `[copy-and-voice]`.
- `__tests__/fitness/nothing-fails-silently.test.ts`: rules 4, 30 and 32, through the empty-catch and
  route-group-boundary checks. Owned by `[feedback]`.
- `__tests__/components/post/tagged-people-pill.test.tsx`: rule 7, for the tagged names' line held from the
  server render in the line box the names arrive in.
- `__tests__/feed-algorithm/for-you-ssr-seed.test.tsx`: rule 21, for the seed painting with no fetch, and a
  tuned first page landing around the rows the first screen showed while a refresh re-ranks all of it.
- `__tests__/fitness/a-route-skeleton-must-server-render.test.ts`: rules 8 and 20. A `loading.tsx` that hands a
  component to a client component drops its whole segment out of the server HTML, silently.
- `__tests__/fitness/an-app-wide-fallback-asks-for-no-provider-below-it.test.ts`: rule 32a. Resolves both
  graphs that replace the app and fails on any module in them that throws for a provider it no longer has.
- `__tests__/components/user/a-profile-holds-the-viewer-s-rows-from-first-paint.test.tsx`: rule 32b, for the
  profile card's mutual row, and rule 7 for the shared-topics line, which holds nothing and lands under
  the counts rather than above them.
- `__tests__/components/shared/a-list-ghost-owns-its-column.test.tsx`: rule 7a, for the spaces, inbox and
  notifications ghosts, and for the notifications bucket label sitting inside the bucket whose rows it
  labels rather than beside the strip.
- `__tests__/fitness/a-ghost-is-written-once.test.ts`: rule 11. Fails on a run of three identical bones in a second
  file.
- `__tests__/fitness/the-settings-ghost-names-the-page-it-stands-in-for.test.ts`: rule 8, for the settings tree.
- `__tests__/fitness/an-empty-state-is-the-primitive.test.ts`: rule 23.
- `__tests__/fitness/a-lazy-sheet-waits-to-be-opened.test.ts`,
  `__tests__/fitness/a-lazy-overlay-waits-for-its-first-open.test.ts`: rule 6, for fallbacks that render before an
  open.
- `__tests__/components/shared/query-state-wrapper.test.tsx`: rules 15, 16, 17 and 30a.
- `__tests__/components/shared/error-page.test.tsx`: rule 30a, for the header a crash keeps.
- `__tests__/components/story/use-story-compose-flow.test.ts` and `__tests__/story/story-compose-flow.test.tsx`:
  rule 42, for the tap that leaves, the arc the ring carries and the refusal that rolls it back.
- `__tests__/hooks/a-publish-bar-moves-after-the-composer-closes.test.ts`: rule 41, for a percent measured
  after the composer is gone landing on the row it seated.
- `__tests__/media/a-video-post-carries-its-poster.test.ts`: rule 7, for a video row reserved at its own
  frame rather than the image column's.
- `__tests__/components/settings/chat-backup-page.test.tsx`: the three backup-key rows of the catalogue,
  driven as a person lands in each of them.
- `__tests__/components/settings/device-link-section.test.tsx`: rules 36 and 37, for every link phase in the
  catalogue above including the two refusals that used to be toasts.
- `__tests__/fitness/a-refused-file-reads-its-own-label.test.ts`: rule 27 and the "File too large" row, for
  the refusal a person reads being the catalogue's line in their own language rather than the validator's
  own detail, which is untranslated and names bytes and MIME types.
- `__tests__/components/message/cross-domain-key-transfer.test.tsx`: rules 36 and 39, for the cross-origin
  half: the match code while it waits, and a spent offer naming a device that can approve instead of only
  retrying against one that cannot.
- `__tests__/fitness/a-sheet-fallback-declares-what-it-stands-in-for.test.ts`: rule 6. Every `SheetFallback`
  passes `rows` or `fullscreen`, so no fallback inherits bones for a chunk that holds a form or a picker.
- `__tests__/fitness/a-bone-sits-in-the-real-line-box.test.ts`: rule 7, for the line box. Every `TextScale`
  has a `--text-*--line-height` box, and no caller freezes a line bone's height with an `h-*` of its own.
- `__tests__/fitness/an-offscreen-estimate-tracks-its-frame.test.ts`: rule 7, for the off-screen estimate.
  Every kind `feedRowHeightClass` names has a `--cv-intrinsic-h`, inside the ratio its own frame allows,
  overridden at `lg` exactly where the frame moves there.
- `__tests__/fitness/one-shimmer-in-the-product.test.ts`: rule 12. `skeleton-shimmer` is declared once and
  named only where it is defined; no `animate-pulse` and no second loading keyframe.
- `__tests__/fitness/a-spinner-promises-no-shape.test.ts`: rule 13. `MediaSurfaceSpinner` still covers its
  surface, no lazy or Suspense boundary takes a spinner as its fallback, and no ghost spins.
- `__tests__/fitness/a-null-row-is-not-a-failure.test.ts`: rule 26, for the surface. Nothing reaches
  `NotFoundPage` through a guard that a thrown read falls into as readily as a resolved null.
- `__tests__/fitness/every-gone-surface-is-a-kind.test.ts`: rule 34. Missing-thing copy, read out of
  `en.json` by what it says, is spent only by `components/shared/not-found-page.tsx`.
- `__tests__/fitness/private-is-not-missing.test.ts`: rule 35. A permission refusal and a missing row share
  no key in `lib/domain/errors/map.ts` and no sentence in any locale we ship.
- `__tests__/messaging/a-chat-summary-leaves-the-device-only-when-asked.test.tsx`: rule 36, for the chat
  summary row of the catalogue: offered rather than run, disclosed the first time, sent only once agreed,
  and never with a line this device could not read.
- `__tests__/fitness/a-standing-change-reaches-the-open-tab.test.ts`: rule 38. Each standing state the
  catalogue lists carries a channel or an interval from a named constant, never `staleTime` alone.

**Not yet enforced.** Nothing: every rule above names a test that exists. The `(not yet built)` marks in
the rules and the catalogue are product gaps, not missing checks.

## Traps

- **3** `isLoading` is false for a disabled, gated or cached query, so the surface paints empty, then bones,
  then the list.
- **6** A fallback that inherits the default `rows` shows bars where a form is about to be.
- **7** A skeleton's outer margin collapses out to `<body>` and moves the whole document; pad the container.
- **7** A fixed pixel rung for a line bone misses the real line box by 2.5px once the viewer's text scale
  multiplies the ramp.
- **7a** A row strip whose parent is not a flex column carries an inert `flex-1`, so its height falls
  straight back to the row count with nothing in the markup saying so.
- **7** A frame capped at a breakpoint leaves its off-screen estimate behind. Stopping the short at square
  on desktop left one shared media estimate a fifth of the column too tall for every reel row.
- **8** A route's `loading.tsx` is a server component: handing a client component a component-valued prop bails the whole
  segment out of server rendering with no visible error, and the parent's fallback shows instead.
- **10** A light fallback panel in front of a camera that is about to be black reads as a flash, not a wait.
- **17** A crossfade applied to a surface that never showed bones is itself the flash it was meant to prevent.
- **21** Hydration once threw away the server-painted feed and refetched it from page one.
- **21** A calendar day read with local getters is UTC in the server's HTML and the reader's own day in the
  browser, so a list grouped by it hydrates into different rows and React throws #418.
- **21** A reader with a learned model got the seed only as a placeholder, and the tuned first page replaced it
  wholesale a second or three after paint, reordering the cards under them.
- **22** A route whose own boundary has a shape but which is missing from the route-ghost map paints a generic
  row list for one frame, then something else entirely. The tab bar's own destinations were missing for months.
- **18** A pending signal that lives inside the element the pending state replaces unmounts itself: the tab
  bar swapped its link for a button the moment the link reported pending, and no tab ever lit before commit.
- **26** `PGRST116` is a row that is not there, not a failure. Only the profile and list routes tell the two
  apart today.
- **35** Rendering the private state before the follow state resolves shows "this account is private" to a
  follower for a frame.
- **41** The composer's own effect wrote the seated row's percent, and the tap that seats the row unmounts
  the composer in the same commit — so the effect never ran once for a post and the hairline sat at 0%
  for the whole upload, minutes of it for a clip.
