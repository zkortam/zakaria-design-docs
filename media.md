# Media

Media is the product. Everything else on a screen is chrome around a person's photo, clip, voice or face, so
media is the one place the app is allowed to be saturated, edge to edge and loud. This document owns how a
picture arrives, how a camera behaves under the finger, how a video and a sound behave once they start, and what
a person is asked to write about what they posted. The feeling it protects is that nothing ever pops, nothing is
upscaled, nothing plays over anything else, and a clip that was playing is still the same clip a moment later
wherever it has moved to.

## Owns and defers

Owns: placeholders and the arrival of an image, requested size and DPR, image normalisation and orientation, alt
text, capture, the file a render publishes, video playback and its controls' behaviour, sound intent, the
decoder budget, audio playback and the waveform, picture-in-picture, and encrypted media in chat.

Defers: the fade, the burst and every curve to [motion]; column widths, the chat bubble and safe areas to
[layout]; the `VIDEO_CONTROL_*` and `FEED_MEDIA_*` recipes as primitives to [components]; on-media scrims,
shadows and hairlines to [colour-and-effects]; upload refusals, file-too-large and unsupported-format states to
[loading-and-states]; byte budgets and how they are measured to [speed]; every string to [copy-and-voice]; the
accessible-name rule in general to [accessibility].

## Rules

### A picture arrives

1. **Blurhash first, image second, and what the server painted is never held.** `BlurhashCanvas` is the one
   placeholder: a 32px decode of a 4×3 hash, painted under the image. Any image that can arrive after its
   layout carries one. A picture the client mounts fades over it through `useImageFade`. A picture in the
   server's HTML carries no `.img-fade` and the browser paints it as it decodes, because opacity 0 held the
   page's largest paint until React hydrated, seconds after its bytes. One that has painted nothing by
   hydration drops into the fade with no transition (`.img-fade-cut`) and rises like any other.
2. **The hash is made once, on the device that made the file.** `encodeBlurhashFromFile` runs beside the upload,
   and the hash is stored index-parallel to the URL so the reader never has to guess which picture it belongs to.
3. **Two pictures skip the placeholder, on purpose.** A single image drawn `object-contain` omits it, because
   the blur would fill the bars with a smear of the picture and the surface should show. And a tile already
   painted with this device's own preview of the same slot keeps that picture — `useImageFade`'s `holdsPreview`
   — because the element holds the painted frame until the published URL decodes, and re-fading would take the
   author's sharp photo back to a blur. That tile is keyed on the post and the media index, never on the URL,
   or the ghost row's swap unmounts it and the reveal happens anyway.
4. **An image is requested at the size it is drawn.** `sizes` is derived from the column the image sits in
   (`READING_COLUMN_SIZES`, `READING_COLUMN_HALF_SIZES`, both from `columnImageSizes`), never written by hand.
   A literal `sizes` string is a magic number that the layout will outgrow. A box whose width comes from its
   HEIGHT says so in the same units the box does: `sizes` takes any CSS length, `svh` and `calc()` included, so
   `SHORTS_FRAME_SIZES` is `calc(92svh * 9 / 16)` from `md`, read off the frame's mirrored height. The single
   value CSS cannot hand over — the preload scanner parses `sizes` before any element exists, so no `var()`
   resolves — is mirrored once, as `SHEET_HEIGHT_FULL`. A grid whose column count changes with the width
   passes `columnImageSizes` a function of the rung, which is how `EXPLORE_TILE_SIZES` follows the tile ladder
   rather than the flat 220px that was serving 256px into a 430px tile.
5. **Every responsive image declares its sizes.** An `<Image>` without `sizes` and without fixed `width`/`height`
   is requested at the largest device rung on every device.
6. **The DPR asked for is the DPR the device has.** Avatars request `AVATAR_PIXEL_RATIO` (3), because phones
   ship 2.6 to 3.0 device pixels per CSS pixel and a 2x request is an upscale on nearly all of them, and they
   snap to `AVATAR_REQUEST_TIERS` so the edge caches seven URLs rather than hundreds.
7. **The device-width ladder covers real devices.** `IMAGE_DEVICE_WIDTHS` carries the rungs a 430px phone at DPR
   3 actually asks for; a rung is added when a device falls between two, never rounded up to the next.
8. **Only what the optimiser cannot help is unoptimized.** `isUnoptimizable`: `blob:`, `data:`, GIF and SVG.
   Everything else goes through the optimiser, in AVIF or WebP.
9. **Quality is set on the transform, once.** The render endpoint and the compressor carry the numbers; no call
   site passes a quality of its own — `ImageCropper` writes its export at `IMAGE_COMPRESS_DEFAULTS` too, because
   a crop that chose its own encoder shipped a JPEG the compressor then re-encoded to WebP. And every canvas on
   the main thread encodes through `encodeCanvas`, which answers a browser that substituted PNG for the WebP it
   was asked for: a `toBlob` call of its own labels those bytes with what it requested rather than what it got.
10. **Orientation is applied at decode, never as a rotation afterwards.** Every `createImageBitmap` reads
    `imageOrientation: 'from-image'`, because the re-encode discards the EXIF tag and a portrait photo would
    otherwise decode sideways. Every path that reads a source's pixels reads them the same way, or the
    placeholder and the picture disagree.
11. **A still is re-encoded before it is uploaded, exactly once, and the size cap judges what that produced.**
    The pass converts what no viewer can decode and strips EXIF at the same time: `normalizeImage` is that pass,
    `useImageCompress` is the one way in, `usePickedImage` is how a surface converts and then judges in one step,
    and a component never reaches `validateImageForUpload` itself. A cap read on the picked bytes refuses a
    12MB PNG that this pass turns into 260KB. An animated image passes through untouched, and nothing PUTs a
    still it had the chance to re-encode and did not. Every canvas that pass draws through, and the crop editor's,
    resamples through `resampleSmoothly`: a canvas's default four-tap filter aliases thin lines the moment a
    4K source is cut to a fifth of its size, and the encoder then spends its bytes on the aliasing. Bytes another canvas already produced at those settings are
    marked rather than encoded again — `croppedFileFrom` builds the crop's file, marks it normalized and
    records its size, so the composer attaches the editor's own bytes instead of fetching a URL back and
    encoding a second time.
12. **Nothing is upscaled.** The fit is `min(maxW/w, maxH/h, 1)`, and no tier's cap sits below
    `STORED_PHOTO_MIN_EDGE` — the phone rung of the reading column at the ratio phones actually have, which
    no device pairs, so it bounds what any card draws from above — or the entry tier stores a photo smaller
    than the card it was stored for draws it. The line the composer warns at is a different number:
    `SOFT_PHOTO_MIN_WIDTH`, the narrowest phone at that ratio, under which every phone stretches the photo
    and not only the widest. `photoLooksSoftInFeed` reads the width, because the card fills the column
    whichever edge is longer, and the height at the 4:5 floor, where the card letterboxes instead; the
    toast names the width, so a person learns how many pixels the feed wants rather than being told the
    photo is small. A crop is the other way a photo gets under it, so the editor says that same line live
    under the frame while the zoom is still being chosen, and says nothing until the line is crossed
    `[input-and-forms §8]`. `framedRegion` is the pixels Done will write, read before it as well as at
    it, or the line would be drawn about a photo the canvas then sizes differently.
12b. **A photo's edit is data, drawn from the original once.** The composer keeps the attached photo, its
    `PhotoGrade` and its `CropFrame` on the entry (`photoEdits`) and draws the result with `renderPhotoEdit`,
    so the editor reopens on the edit it carries, a second visit redraws the original instead of grading the
    grade, and Reset is the original itself with nothing encoded. A draft keeps the original's URL and the
    edit beside the photo it made; a draft without them opens each photo as its own original.
12a. **A link's picture is one we keep.** The unfurler downloads a page's picture once, re-encodes it with
    `prepareCardImage`, and serves it from our media host with the size and BlurHash it measured; a card never
    asks the linked site for a byte, and a picture it could not keep or measure is not drawn. `LinkPreviewCard`
    lays a picture across the card only when it is wide enough (`LINK_CARD_LARGE_MIN_WIDTH`,
    `LINK_CARD_LARGE_ASPECT`), and beside the words as a thumbnail otherwise, so a narrow one is never upscaled.

### Alt text

13. **Alt text is offered, never demanded.** The composer shows for each picture whether it has a description and
    opens `MediaAltTextSheet` in one tap, capped at `ALT_TEXT_MAX_LENGTH`. Publishing is never blocked on it.
    Once the picture has uploaded, Describe asks a model for one, offered as an `AiProposal` beside the field;
    it is the author's only when they take it, and the post's "Made with AI" stays theirs to set.
13a. **A model's description is said to be AI's.** Where the author wrote none, the describe worker's alt in the
    reader's language (`media_described`, carried by `get_post_extras`) is read out through `useAltLabel` as
    "AI description: …". Nothing is drawn on the picture for it: the feed carries no ALT chip, and the viewer
    reads the same words as the image's name.
14. **Alt text travels with the picture.** Wherever the picture renders, the description renders with it: feed,
    carousel, viewer, embed, notification row and shared card all read the same array.
15. **`alt=""` is a claim, not a default.** It says this image carries nothing a reader would miss, which is true
    of a poster sitting under its own video and a decorative plate, and false of a person's photo. A primitive
    that defaults `alt` to an empty string ships that claim on every caller that forgot.
16. **An avatar's description is the person,** carried on the element a reader reaches, not on the inner image.

### Capture

17. **The shutter fires on the press.** The haptic and the capture wash land in the same frame as the pointer
    goes down; the blob arrives when it arrives. A shutter felt after the frame reads as lag.
18. **The preview is the capture.** One expression decides the mirror and one value the zoom, both read at the
    moment pixels are needed, so what was on screen is what is in the file.
19. **The camera delivers upright frames.** Portrait is enforced on the track when the device offers a
    landscape-biased one; nothing rotates a canvas at capture time, because a delivered frame is already upright.
20. **A lens switch cross-dissolves from a frozen frame,** never from a black gap while the new track opens.
21. **Capture is silent.** Feedback is the haptic and the light; the product never plays a shutter sound over
    what the person is recording.
21a. **Capture keeps the product's shape.** `full` is the window's own shape only while the window is taller than
    9:16, a phone's screen; a tablet's or a desk's wider window records 9:16 (`aspectRatioValue`). The stage fits
    its box on both axes and stands centred at full height, and the camera's controls ride the frame
    (`CAMERA_FRAME_CHROME`), not the screen's edges.
21b. **A 9:16 frame is never cropped to the window.** A short, a story and the camera stage keep 9:16: a regular
    window below `lg` draws the short at full height, centred, with its action rail on the picture and its comments
    sheet in the frame's own column (`SHORTS_SHEET_PANEL`), and the desk shape takes over at `lg` `[layout §1]`.
22. **What the browser cannot draw is converted before it is shown, and the conversion never holds Post.**
    HEIC is decoded through the wasm decoder only in browsers that need it, and the crop editor never
    receives a file it cannot paint. A clip's conversion is decided from its metadata — `planVideoFile` —
    and executed behind the accepted attachment, so publishing waits for the bytes and the person does not:
    `holdsPublish` covers validation, the plan and the frame scan, never the re-encode. The plan is also
    what refuses: a duration this browser cannot probe is not a refusal of its own, or the two messages
    that name the format — `videoCannotConvertHere` and `cannotPlayFormat` — are unreachable behind one
    that names nothing.

### Making a short or a story

22a. **One creation shell, three steps, one palette.** Capture, edit, publish, for a short and a story alike.
    Every camera, editor and publish step renders inside `ThemeScope` at `CREATE_SURFACE_THEME`, so the
    palette never flips between capture and post, and every sheet opened from them wears it without a prop.
    The modes under the shutter are quiet words (`CreateModeStrip`): the chosen one in full ink, never a pill.
    A camera that never opened stands `CameraUnavailable` in the frame, and only what needs no lens stays:
    Close, the gallery, the modes and the tools marked `worksWithoutCamera`. Drafts are one labelled
    `CameraDraftsButton` on both cameras. A short opens on its camera on a phone, with no screen before it;
    a desk opens on `ShortsUploadScreen` in the content column beside the rail, where one button picks a
    file, a file dropped anywhere on it imports, and the webcam is one header glyph. Both editors' top bars are `CREATE_BAR_GLYPH` glyphs around one
    `CREATE_NEXT_BUTTON`, and every sheet whose tool changes the frame leaves it undimmed and live.
22b. **The editor is the preview.** A take goes from the shutter straight onto the canvas; there is no
    confirm step between them, and closing the editor is the retake (`composeReturnTab`). The story editor
    stands in as the deck's panel while the deck mounts, so the camera returns at rest, not on a second slide.
22c. **Every tool in view, none behind More.** Both editors reach every tool through `CreateToolRail`, each
    glyph over its word: one row under the stage, scrolled for the rest and spaced by `usePeekGap` so the
    first tool that does not fit shows its glyph at the edge. A short's tools are that row on every width; a
    story's are that row on a phone and a column of tiles beside the stage on a desk. There is no More, no ··· menu and no bar of tool categories. A story leads with Text,
    Stickers, Draw, Music and Look; a short with AI, the editor's one AI entry, then Sound, Text and
    Captions, and the quick editor's row ends on the full editor and stays whole whatever is selected. In the
    full editor a selection's own tools take the row or column's place with Done held ahead of them at the
    tools' own spacing, each new set rising in from its first tool, and Done returns the whole video's. The
    header holds only Close, Undo, Redo and Next; a story's chosen track shows there, so Music carries no
    second cue. A short's stage is the largest 9:16 its column holds (`EDITOR_STAGE_BOX`); from `lg` its
    tool panel and agent open into one dock whose width is always held; narrower the dock floats at the
    stage's end (`EDITOR_DOCK_SLOT`) and the stage glides clear of it, smaller and still 9:16, while it is
    open (`EDITOR_STAGE_CLEARS_DOCK`). A story's stage sits centred at rest and glides aside with its tools as
    a panel opens (`deskGlide`); where the desk is too narrow for both, the stage shrinks on the same glide,
    still 9:16, so the frame never runs past the screen (`stageGlide`). One tool has one name and glyph in
    both editors (Look, Trim), and one window (`TrimWindowPanel`) trims a story's clip and a single-take
    short alike; a short joined from several takes trims on the timeline.
22d. **One Next, then the publish step.** A short's Next exports in the background (`useShortExport`): the
    editor stays live, the capsule carries the wait, a second Next exports the edit as it is then, and the
    publish step opens on the finished file. The export belongs to its edit: leaving the editor or opening
    another edit stops it, and a change made after Next keeps the file in the capsule, named as older than the
    edit, rather than opening publish on it. Exports run one at a time, and a finished copy waiting for its
    tap is never replaced. A short's publish states audience, cover, what others can do and
    AI on its first level, with everything else in one fold; its top bar's one other glyph keeps a
    high-quality copy on the device. A story publishes in one tap on the audience the button names, Your story
    or Close friends, and a group is one sheet away.
22e. **A cut is a swap, never a seek.** A short's stage holds two elements (`useVideoPair`): the one shown,
    which every layer that reads the frame follows, and a standby resting decoded on the frame the next cut
    lands on (`standbyShotFor`), so the picture changes in the frame the cut is taken and the outgoing clip
    never holds. The cut is taken on the frame clock one measured tick ahead of the out-point
    (`CUT_LEAD_SECONDS`, `PRESENTED_AHEAD_SECONDS`), so no frame the edit cut out is shown. A clip whose file
    the standby has not loaded holds the last frame shown until it has: never black, never the file's first
    frame. A gap is dark from the out-point, and a paused scrub keeps only its newest seek (`seekLatest`).
22f. **A short's stage is its export.** The stage is one canvas (`CanvasStage`) painted each frame by the
    export's own steps in the export's own order (`composeStageFrame`): the picture, or a transition through
    `paintTransition` with the outgoing clip on an element of its own (`useTransitionSource`), then
    `composeCutout` or `composeChroma`, `composeMasks` and `composeOverlays`, picture-in-picture included.
    The video elements and every text, image, caption and picture-in-picture box are hit targets that paint
    nothing, so the stage shows nothing the file will not carry and misses nothing it does. Both compose a
    frame at its moment on the timeline; a sticker plays its own frames at that moment
    (`useStageAnimations`); the stage asks for each face the text and captions draw with, as the export does
    (`overlayFonts`), and repaints as one lands (`useStageFonts`); and every filter either draws (a text's,
    a word's or a scene node's blur, a mask's effect, the look's adjustments) is computed on WebGL2, else on
    pixels, where the browser's canvas ignores `filter` (`drawFiltered`, `paintFiltered`, WebKit before Safari
    18). The one difference is the cutout's matte, read ahead for every file the edit's elements play (a
    reversed clip's own reverse included, `playedUrlOf`) by the small segmenter here and by BEN2 in the
    export. The stage paints at up to 2 device pixels per point, playing or still, so a pause never sharpens
    the picture a step (`PREVIEW_DPR_CAP`).

### Video

23. **The poster is an image, not an attribute, and every clip has one.** A `poster` attribute carries no
    srcset, no sizes and no preload, so it made the feed's largest element a full-size original discovered
    late. The poster is a real image element beneath the video, with its own blurhash and sizes, and the video
    holds at zero opacity until its first frame — a hold that is disabled where no poster exists, so a clip
    without one is a black rectangle and not a placeholder. A post captures it the way a short does:
    `captureBestPosterBlob` runs beside the PUT at attach, the still uploads silently, `encodeBlurhashFromFile`
    hashes it, and `postMediaColumns` writes `video_thumbnail_url`, the hash in `media_blurhashes[0]` and the
    clip's own pixels in `media_dimensions[0]` — which is what reserves the frame before a byte of video lands.
24. **Autoplay is muted, on screen, and only well on screen.** The intersection threshold is 0.6 so a clip
    scrolling past the edge never starts and two clips can never both qualify. Reduced data turns autoplay off
    and the person can still play.
25. **Sound intent is one value for the session.** `useSoundIntentStore` holds `unset`, `on` or `off`; turning
    sound on once answers for every clip after it until it is turned off, and the volume keys set it too.
    Every sound control writes it through `setPlayerSound` and draws `SoundToggle`: the feed and fullscreen
    players, the shorts frame, the story header, the floating window and a live stream. A short and a story
    start sounding unless the value is `off`, and nothing turns sound on behind a tap the person did not make.
25a. **Speed is chosen per clip, from four named stops.** `PLAYBACK_SPEEDS` is the one ladder and
    `PlaybackSpeedControl` the one control, on every player and on voice posts and voice notes; a tap opens
    the track and the stop under the finger plays before release. On a video player the track lives in
    `VideoMoreMenu`, and the strip shows the rate only while it is not `NORMAL_PLAYBACK_SPEED`. Speed is not a
    session value: a 2× short never carries into a voice note. A held press on a short or a full-screen clip
    plays at `HOLD_PLAYBACK_SPEED` over the chosen stop, only while held.
26. **One thing sounds at a time.** A video claims playback through `claimVideoPlayback` and an audio post claims
    the speaker through `claimAudioOutput`; an arriving player takes over from the one still playing.
27. **A clip well off screen releases its buffer.** Beyond `BUFFER_MARGIN` the source is detached and the element
    reloaded; the shorts track mounts `MOUNT_WINDOW` neighbours and loads `LOAD_AHEAD` ahead. Every feed video
    row asks for that release (`releaseWhenFar`), and none attaches a source at mount: a `<video>` mounted
    with one opens a range request whether or not the row is anywhere near the screen.
28. **A retry can tell its own abort from a refusal.** The muted-autoplay retry distinguishes the `AbortError`
    thrown by its own pause from a policy block, or it restarts every reel off screen.
29. **Scrubbing is continuous and precise.** `RangeInput` sits over one painted bar and takes the pointer itself,
    anywhere on the track and on the axis rule of `[motion §37]`, keeping a native range only for the keyboard and a screen reader;
    the playhead is written imperatively per frame as `VIDEO_PLAYED_VAR` on the bar, the fill reads it and
    nothing rides it `[craft §27]`, the step is
    `VIDEO_SEEK_STEP_SECONDS`, and the readout is announced through `aria-valuetext`. Where there is no
    strip, the shorts rail and simple mode's bar, it is `OnMediaScrubBar`: a hairline at rest that swells to
    the held bar as a clip, its fill a `RangeFill`, the time floated above it while held.
29a. **A chapter never takes the timeline's room.** Chapters break the bar itself into `VideoChapterTrack`
    capsules with a hairline gap, each filled from the same `VIDEO_PLAYED_VAR`, on a player's strip and on the
    shorts rail alike. A chapter's name is never a title in the control row: on a player it is
    `VideoChapterScrubLabel`, floated over the playhead while the bar is dragged and fading on release, and it
    reaches a screen reader through `aria-valuetext`. The shorts rail names it in its scrub readout.
30. **Captions are on by default wherever a track exists.** `useVideoCaptions` reads the track in the
    viewer's language and `useCaptionsPreference` holds the choice, on the account or on the device; both
    players carry `VideoCaptionTrack` and draw `VideoCaptionOverlay`, which rises over the strip while it shows.
    The switch is in `VideoMoreMenu` and on the C key, and in a short's More, never a glyph on the strip or the frame. Burned-in editor captions
    are a creative overlay and are not captions.
31. **HLS is one adapter.** `useHlsPlayer` prefers the platform's native playback and loads `hls.js` only where
    it does not exist. A surface never writes its own manifest check.
32. **Two players, one core, one strip.** The inline player and the fullscreen player share `useVideoPlayerCore`
    and draw one `VideoControlStrip` over `useVideoTimeline`; a third
    surface that needs a video extends one of them rather than building a sibling. A tap that moves a clip
    from one to the other carries its playhead, its sound and the frame it was showing — `startAt`,
    `startMuted`, `startSpeed` and `captureVideoFrame` — so fullscreen opens on the same clip rather than a black restart.
    The picture grows out of the frame's own box (`origin`) and shrinks back into it, and leaving hands the
    frame a `FullscreenLeave`, so the clip carries on from where full screen left it.
33. **A feed frame is portrait on a phone and stops at square on a desktop.** `videoFrameStyle` writes both
    shapes onto the frame and the `lg:` half of `VIDEO_FRAME_ASPECT_CLASS` chooses the second, so every clip
    floors at `VIDEO_FRAME_SHORT_ASPECT` where the column is the screen and at square where it is not — and no
    card is taller than a landscape window. The same class caps the frame at `--app-vh`, so a phone held
    sideways never draws a frame taller than its screen. The composer previews that same frame, so what the author sees
    attached is what the card draws. Nothing is measured in JS, the crop is `object-cover` about the centre,
    and the poster fills the same box as the clip.
34. **A start that never comes is a failure, not a spinner.** A file that cannot be decoded downloads
    whole, leaves `play` pending forever and raises no `error`, so bytes arriving prove nothing:
    `hasDecodedFrame` is the question, and `SHORTS_START_DEADLINE_MS` is how long the surface asks it
    before giving the person `MediaErrorFallback` and its retry. Whether a clip is playing is the
    element's own `playing` and `pause`, never a settled promise. Every playback surface asks it the same
    way — the shorts track through `useShortsPlayback`, the feed and fullscreen players through
    `useVideoFailure` — and the retry re-attaches the source rather than reloading the page.
34a. **A player shows what is needed at a glance and keeps the rest behind More.** The strip is play, sound,
    time and the bar, the rate only off the normal one, `VideoMoreMenu`, and fullscreen. More holds the
    speed track, captions, loop, the floating window and a link to this moment; nothing rare earns a glyph
    on the picture. At rest a phone's frame keeps one control, `SoundToggle` with the time left inside it,
    and a tap brings the strip up and takes it down again, because a touch never leaves the way a pointer
    does. A phone's feed frame that opens a page of its own (a feed short) keeps play and sound, and its bar
    becomes the picture's bottom edge (`VideoControlStrip` `edge`); time, speed, More and fullscreen wait for
    that page. A clip `useHasAudio` finds silent offers no sound control anywhere, only its time. No scrim
    darkens the picture behind the strip; its pills carry their own ground. Every state change on the picture
    fades or scales; the strip, the pill and the play badge are mounted once and faded, never mounted on show.
34b. **A key does what its control does.** `videoCommandForKey` is the one map, `useVideoCommands` the one
    executor and `useVideoKeyboard` the one listener: space and K play, J and L skip `VIDEO_SKIP_SECONDS`, the
    arrows seek `VIDEO_ARROW_SEEK_SECONDS` with the timeline and mirror under right-to-left, up and down move
    the volume, M, C and F toggle, a digit jumps to that tenth, comma and full stop step a frame, and the
    angle brackets step the speed. A key the focused element answers is left to it. The key is written
    beside the control's name (`media.withShortcut`) so it is learnt where the control is.
34c. **One tap owns the screen.** `FullscreenVideoPlayer` puts its own element in the browser's fullscreen
    where `canFullscreenElements` says it can, so its controls, captions and gestures come with it; an iPhone,
    which cannot, gets the same player filling the viewport. Leaving the browser's fullscreen leaves the
    player. The browser draws nothing outside that element, so the player names itself through
    `PortalRootProvider` and every menu, sheet and tooltip opened in it portals there. A phone turned sideways
    while a landscape clip plays opens it here, and turning it back leaves. Its strip clears the cutout and the home bar with `ps-safe-min-*`, `pe-safe-min-*` and
    `pb-safe-min-*`, and tightens under `landscape:`.
34d. **The hand drives the full-screen clip.** `useVideoSurfaceGestures` is the one reader: a tap shows or
    hides the controls, a double-tap on a side third skips and each later tap on that side keeps adding
    while `VideoSkipFeedback` shows the total, a hold plays fast, a drag down follows the finger and leaves
    past `releaseDismisses`, and two fingers fill the screen or fit the picture on an animated scale. A vertical
    swipe on the left third moves the picture's brightness (`VIDEO_BRIGHTNESS`, since a page cannot set the
    screen's) and on the right third the sound, through `useVideoLevels`, where `canSetVolume` says the browser
    allows it; elsewhere it falls through to the drag. A mouse click plays or pauses and a double-click leaves.
34e. **A clip picks up where this device left it.** `useVideoResume` stores `VIDEO_RESUME`'s bounded map
    and resumes past its opening and short of its end, then offers `VideoResumeOffer` for a moment: an undo,
    not a question.
34f. **The system's media controls follow the clip that played last.** `useMediaSession` names it with
    `videoSessionMeta` and answers play, pause and seek from the lock screen, a headset or media keys.
34g. **A moment is addressable.** `momentHref` writes `VIDEO_MOMENT_PARAM`, `useMomentFromLink` opens a post's
    video there, and a reply written while the clip plays is offered `ReplyMomentChip`; a reply that carries a
    moment draws `ReplyAnnotation`, whose tap plays the video it answers from there through
    `useVideoMomentStore`.
34h. **A video is followed by the videos it is about.** `get_related_videos` ranks ready videos by their text
    embedding's distance from this one's, dropping blocked and muted authors, and `useRelatedVideos` asks only
    once the row opens. `VideoRelatedPanel` rises under the full-screen picture, which lifts and shrinks still
    playing, on V, on `RELATED_WHEEL`'s travel, on a swipe up through the middle and from More; Escape, the wheel
    back at the row's top, a tap on the picture or a drag down lower it. A picked video plays in place, and
    the frame the player came from gets back its own clip where it left it.

### Audio

35. **Every waveform is one continuous shape.** `waveformEnvelopePath` draws the envelope through the levels,
    mirrored about the midline, so a stored clip, a voice message and a take being recorded all read as one
    thing. Bars represent discrete choices and are not a waveform.
36. **The waveform is the scrubber.** The progress is a clip of the same shape, never a second element drawn on
    top, and a transparent range sits over it for input.
37. **Levels are normalised once.** `normalizeWaveform` for real levels, `placeholderWaveform` before they land,
    at one bar count; a surface never rescales amplitudes of its own.

38. **An editor sounds what its export will mix.** `useMusicBed` is the one music bed: the story
    viewer plays what was published and the canvas editor plays what is about to be, from the same
    pooled element, at the same authored offset, looping with the clip and holding the speaker
    `[feedback §13]`. A track that only plays inside the picker's own row leaves the author judging a
    soundtrack they have never heard, and a mixer whose slider moves nothing is not a mixer. So a track
    being chosen is heard under the creator's clip, from the part it would play, with the editor's own
    track held (`useClipAudition`); only the camera, which has no clip yet, previews a track alone. The
    mix is one Original ↔ Music balance, `mixBalance` over the same two levels the precise volumes one
    fold down set (`MusicMix`) `[input-and-forms §38]`.
39. **A provider's bytes are read through our own origin.** `musicStreamUrl` is the only URL the app
    holds for a catalogue track, and `resolveMusicUrl` heals one stored before it existed. A
    third-party CDN answering without `Access-Control-Allow-Origin` can be PLAYED by an element and
    never READ, so both export paths mix silence. The catalogue also drops a track whose audio is
    gone rather than offering a row that publishes nothing.
39a. **A catalogue song is credited where it plays.** Its licence is part of the song: `musicTrackOf` carries
    the Creative Commons deed from the provider, `creditMix` looks up every song the export mixed and stores
    each as an `audio_tracks` row with its `license_url`, and `set_post_music_credits` records each use on the
    post with the title, artist and deed of that day. A song picked where no deed travelled (a camera pick, an
    older draft, a save) has it read back from the provider by `recordCatalogueTrack`, the one writer of a
    catalogue row. The credit arrives with the short through `get_post_extras`, so the line under its author and
    the feed player's sound pill read title, artist and deed on their first paint (`soundLineOf`), a story's
    ticker reads its deed from the story row (`songCreditLabel`), and the sound page links the deed through
    `CreditLink`. Only Attribution and CC0 songs reach the catalogue or a post (`isPublishableLicence`, and the
    database's own check on every credit): a mix that plays any other, or a song whose licence cannot be read
    back, is refused on the Post tap with its own message, before the reel is seated, so the person is still
    there to change it. An empty provider answer for a song already held is the provider's flake, never a
    delisting. The credit is written with the post, and a post whose credit fails is taken back rather than
    left uncredited.
40. **A story claims music only when its file carries it.** The row is written from what the render
    shipped — `onMusicBaked` — never from what was chosen, because a bake can still be refused by a
    dead track or a browser that cannot encode audio.
40a. **A cut is heard as one continuous sound.** The shorts preview plays the original track from the
    export mixer's own render of it, `originalTrackOptions`, so every seam is the export's 10 ms
    equal-power crossover; the heard position, `previewOriginalStemTime`, is the clock the picture
    follows across cuts (22e), and no element is seeked at a cut. The element carries the sound only where
    the graph cannot run or while an edit re-renders. Every other sound (an effect, a voice, the next pass of a
    looping bed) waits paused where it starts and rolls in unheard `AUDIO_PREROLL_S` ahead, so a splice finds
    it playing and on time and crossfades as the export's does. Only a routed element (`previewElementIsRouted`)
    rolls in: one playing direct is heard at any level on iOS, so it starts from its first sample.
40b. **A mute silences its own track and nothing else.** Original, music, voiceover and effects each
    have a gain (`setPreviewTrackMuted`) and an entry in the edit that both export paths read through
    `withMutedTracks`; levels are kept, so an unmute restores them.
40c. **A render keeps the sound it was given.** The mix runs at 48 kHz stereo and writes AAC-LC to
    `AAC_EXPORT_CONFIG`; a phone clip is never resampled, and a value curve is drawn fine enough that a
    10 ms fade stays 10 ms on a clip of any length. A speed change keeps the pitch through
    `stretchToLength`, whose grains line up with each other rather than beating into a warble.

### Picture-in-picture

41. **One element, moved.** The window is the card's own video reparented into the dock, never a second player
    and never the browser's own picture-in-picture. The card's stage renders nothing but that element, so a move
    lands nowhere React is tracking.
42. **The dock stage is mounted on every route,** because a media element pauses the moment it leaves the
    document; the hand-off appends it somewhere in the same task.
43. **The window is earned.** Only a clip the person actually watched with sound docks on navigation; a muted
    clip that scrolled past does not follow them.
44. **Dismissing returns the element home,** pauses it, unpins playback and releases the floating slot, so the
    next arriving player can claim it.
44a. **There is one floating window.** `FloatingWindow` holds the video dock and a minimised live stream alike: it
    follows the finger 1:1 once a press becomes a drag, releases on `SPRING_SHEET` into the corner
    `releaseFloatingWindow` projects the velocity toward (`useFloatingCorners`: four on a phone, the two end
    corners on a desk, where the start side is the rail's) inside the `--fw-top`, `--fw-bottom`, `--fw-start` and
    `--fw-end` insets, and a throw past an edge dismisses it. A stream
    thrown away is offered back with an undo; live keeps its resize grip, on the corner opposite the anchor.

### Media in chat

45. **A sealed picture is decrypted only when it is near the screen,** and the decision latches, so scrolling
    back does not decrypt twice. The observer attaches to the documents column when there is no picture grid, or
    a file-only bubble waits forever for a grid that never renders.
46. **A sealed tile in a gallery is never fetched.** It shows the lock and the way into the thread that can open
    it, because a mounted `preload="metadata"` video requests its bytes at once.
47. **Chat media is one width.** `CHAT_MEDIA_WIDTH`, viewport-bounded, resolved against the bubble's content box.
48. **A sealed picture still gets a placeholder.** `(not yet built)`: `messages` carries no thumbnail and no
    blurhash column, so every encrypted tile is a full-size original decrypted on the device and a chat picture
    has no placeholder of any kind. It needs a blurhash computed at send and carried in the envelope's plaintext
    header, and a thumbnail beside the original.

### What a render publishes

49. **A render is muxed under a header a decoder accepts, or it is not published.** `videoChunkSink` is the one
    path from a `VideoEncoder` to a muxer: it rebuilds an `avcC` record the browser wrote malformed, from the
    key frame's own parameter sets where the record itself cannot be read, and fails the render rather than
    handing the person a file that plays nowhere.

50. **An animated overlay is never flattened.** A canvas keeps only the frame it drew, so `useCanvasExport`
    proves each media overlay with `decodeAnimatedImage`: a video export bakes the animation into its own
    frames, and a still export publishes the overlay unbaked for the viewer's `<img>` to animate `[media §8]`.
    Unbaked, it is decoration and not a control, so a tap on it still pages the story.

51. **An encoder hands frames back in the order it took them, or the render fails over.** An MP4 sample table
    holds one decode order, so `MUXABLE_ENCODER_CONFIG` asks for `realtime` latency — the one knob that stops an
    encoder reordering behind B-frames — and both sinks refuse a chunk whose timestamp went backwards. A throw
    inside an encoder's output callback reaches `window.onerror` and never the render's promise, so
    `videoChunkSink` and `audioChunkSink` leave the failure for the render to read and write nothing after one.

52. **An export's own playback starts muted rather than not at all.** The real-time fallback records the source
    through an `AudioContext`, and an element that feeds one can never be un-fed — WebKit then gates its
    `play()` on live user activation whatever `muted` says. So the unmuted start is proven before the element is
    wired in, a refused one records silently, and a refused muted start is `FALLBACK_PLAYBACK_REFUSED` rather
    than a filed autoplay error over a capture of nothing.

53. **One story ratio, from the stage to the pixels to the viewer.** `CANVAS_EDITOR_STAGE` is
    `aspect-[9/16]` on every breakpoint, letterboxed on the editor's own ground, and both export paths
    resolve to `STORY_EXPORT_DIMENSIONS` — so what the author placed is what the file contains. On a
    phone the viewer's stage is the screen (`STORY_VIEWER_STAGE`): `StoryAmbient` paints that picture
    to the edges, and `STORY_AUTHORED_FRAME` holds the frame in the space under the progress, name and
    music and above the reply. A stage that took the phone's ratio published a frame the author's own
    preview then cropped, and drew baked overlays a fifth larger than the text beside them.

54. **A shared post in a story is drawn by the viewer, never baked.** `embed-content` is in
    `UNBAKED_STICKER_KINDS`: the exported pixels hold only the backdrop, and the card renders live — its
    relative time is read at view time, its text is crisp at the device's own pixels, a clip in it plays,
    and the tap affordance is a real element. The fullscreen short is the one exception: its poster is
    drawn into the frame as the base layer under the live video. Stories from before the split carry
    `bakedIntoMedia` and keep their pixels, with the live card serving as their invisible tap target.

55. **A render that could not reproduce something says which thing.** `RenderGap` is the named list,
    `RenderResult.gaps` carries the ones that happened, and `renderGapNotice` says them in one line in
    the reader's own language. A `catch` that drops an effect and carries on with a lesser frame is a
    preview that lied, and both export paths read this before they hand the file on. A capability is
    asked of the CONFIG, never of the class: `isAudioEncodingSupported` probes
    `AudioEncoder.isConfigSupported`, and `pickRecorderMimeType` refuses a format whose codec string
    names no audio codec — a browser that ships the class with no encoder behind it otherwise declares
    a track it cannot fill and publishes it with no samples in it.

55a. **A published short streams; a saved copy keeps.** The standard export is the file every viewer
    streams: H.264, no larger than 1080p, at a bitrate `videoBitrateFor` raises with how much the finished
    picture moves (`editMotion`: the source's own motion from `measureTrackMotion`, sped up as the edit plays
    it, and the particles, FX, graphics and transitions it adds) under a ceiling a mid 4G link carries even
    at the full `VBR_AVERAGE_OVERSHOOT` the export probe allows. A source is published as it is only under
    `isBitrateOverBudget`, judged at its own frame rate against `maxDeliveredBitrateFor`: the most a fresh
    export of that frame averages, so no source outstreams what Surf would make of it. The high export is the creator's own copy:
    `chooseVideoEncoding` takes HEVC, then AV1, where the device encodes them in hardware, else H.264, up to
    the source's own 2160p, at 10-16 Mbps for busy 1080p60, declared at the lowest level that carries the
    frame at its rate. Before the first frame, `exportFrames` steps either down to the largest frame this
    device can hold and finish, and a low-tier phone to 30 fps; a frame only a software encoder can make must
    also finish in time (`finishesInTime`).

55b. **An export never holds its file in memory.** `openMp4Writer` writes the media into the origin's private
    storage as the encoder produces it (from a worker's sync access handle where the page has no writer, as in
    Safari), and the index into space reserved at the front, so the file plays before it has all arrived and
    says its full length. Only a browser without private storage, or a disk without room for the file
    (`exportDiskRoom`), holds it whole, and the device guard counts it when it does. A render nothing will read
    again is deleted (`discardExportFile`); the rest go a day later. The audio is encoded first and closed, so
    the muxer never waits on it to write a second of video.

55c. **HDR arrives as SDR, not clipped and not dimmed.** An HLG or PQ source is tone-mapped on the GPU
    (`createToneMapper`): reference white lands near SDR white and highlights roll off into it. A frame with
    planes is read from them; an opaque one, as a hardware decoder hands over iPhone HEVC, is uploaded as
    extended-range float where the engine keeps its highlights. A frame drawn from a `<video>`, on the seek
    path and the recorder path, goes through the same map (`createElementToneMapper`). A frame none can map
    is named in the render's gaps (`hdr`), never exported dim in silence.

55d. **A model download is named before it starts.** However cutout turns on, its export model is fetched
    at once on a link that costs nothing, and otherwise its size is said before the export downloads it: in
    the cutout panel's line, or once where the panel is closed (`useExportMatte`).

### Listening

56. **A microphone is heard in its own blocks, on its own clock.** Whatever a surface judges from a voice —
    its level, where a word starts and ends, the audio a model reads — comes from `openMicrophoneTap`'s sample
    blocks, timed by the samples themselves. A level read when a video or animation frame happens to run
    misses the voice between them, and a stalled main thread must delay a word, never lose it. The tap is
    opened from the press: its `AudioContext` is made before the first `await`, where iOS still lets it start.
    A gate or a message hears the voice `processed`; a take kept as recorded, the editor's voiceover, opens it
    `as-sounded`, because echo cancellation and gain control pump and colour a voice nobody is calling.
57. **A surface that cannot hear says why, and offers the fix.** A microphone sending nothing (muted, unplugged,
    the wrong input, a graph that never started) is `silent`; sound too soft to be a voice is `quiet`. Each has
    its own line, and while either stands the other inputs are offered by name, moving the reading to the one
    picked without losing what was already said. The level shown is `levelShown`: decibels above the room, so a
    soft voice on a laptop moves the edge as visibly as a loud one on a phone. A clip a model heard no words in
    is not an attempt. Whisper mishears a word alone that it hears after two others, so every check carries the
    last `SPEECH.contextClips` heard clips in front of it. A worklet that will not load falls back to a script
    processor rather than ending the reading.

### The dark

58. **In the dark, the screen is the front camera's light.** `useScreenLight` reads the whole frame's brightness
    against `SCREEN_LIGHT`, so it comes on before any face could be found, and on a found face too dim to
    measure. The stage (`FaceStage`) fills with fixed white at `slower`, the lens steps back so more of the
    screen shines, and the type turns half-way through the fade. It turns off only after a spell of plain
    daylight, and stays on once it has had to come back. Under reduced motion the fade is `fast`, never a cut:
    a full screen snapping to white is a flash. The web cannot raise the display's brightness; white at full
    opacity is as far as a page can go.

### Quality

59. **A pixel, a colour or a sample is given up only at a named step.** `MEDIA_RENDITIONS` in
    `lib/constants/media.ts` is every downscale there is: the tier photo edge, a DM photo's HD and Smaller, the
    avatar and the cover, the story still and story video, the poster, the display renditions, the composer
    preview and the never-uploaded thumbnails and analysis frames. Every resize, re-encode and quality reads its
    step from there; a number written anywhere else is a downscale nobody chose.
60. **A step is never below what its screen draws at a phone's 3x.** A camera still is stored at 1440×2560, the
    widest phones' device pixels; a banner at 3840, a phone held sideways; an avatar at 1024. A display rendition
    swaps for the stored original past `originalPastZoom`, once the original has decoded, so a lightbox zoom is
    never a rendition drawn larger than it was encoded.
61. **A camera still asks for the most pixels the camera gives.** `takeLargestPhoto` asks `takePhoto` for the
    capability maximum by name and the camera's own flash where it lists one; anywhere else the lit frame is drawn
    while the torch is still on. Either becomes the stored still in the shape it was framed in (`stillOutputFor`).
62. **A picked picture is encoded once.** A picker that crops keeps the original it was handed
    (`keepOriginal`) and the crop is its one encode; a file this app already encoded is never encoded again.
63. **Wide colour is kept.** A still is drawn into a `display-p3` canvas (`STILL_CONTEXT`) wherever the engine
    has one, so a P3 photo is not clipped to sRGB on its way to storage. Gain-map HDR stills are stored SDR.
64. **A recording's quality is said, never assumed.** The camera asks for H.264 High at level 4.0 before
    Baseline, a voice note for its own bitrate, and a take's microphone opens as it sounded
    (`takeMicrophoneConstraints`), with echo cancellation only while a music bed plays. A recorder that refuses
    or lowers the bitrate is reported.
65. **A save is the stored bytes.** A download of a picture a person may keep is the file as it is stored, never
    a canvas re-encode; a sealed attachment opens a chunk at a time (`openSealedStream`) and never whole.
66. **A camera opens each side on its default lens.** `pickPreferredCameraDeviceId` never opens on an ultra-wide or
    a telephoto, and a live broadcast flips by facing through it. A broadcast holds the `MEDIA_RENDITIONS.live`
    bitrate ladder and keeps its frame rate when the link tightens. Dual camera is offered only where two lenses
    hold at once (`canHoldTwoCameras`).

## Where a size comes from

| Surface | `sizes` source |
|---|---|
| Feed image, single | `READING_COLUMN_SIZES` |
| Feed grid tile | `READING_COLUMN_HALF_SIZES` |
| Row thumbnail | `ROW_THUMB_SIZES` |
| Grid thumbnail, reading column | `THUMB_SIZES` |
| Grid thumbnail, discovery | `EXPLORE_TILE_SIZES` |
| Picture-search tile | `WIDE_COLUMN_HALF_SIZES` |
| Shorts, full-bleed | `SHORTS_FRAME_SIZES` |
| The lightbox stage | `LIGHTBOX_SIZES` |
| Avatar | `AVATAR_PX` tier × `AVATAR_PIXEL_RATIO` |
| Inside a dialog | the `DIALOG_SIZES` preset the dialog uses |
| Chat media | `CHAT_MEDIA_WIDTH` |

## The playback registries

| Registry | Holds | Claimed by |
|---|---|---|
| `claimVideoPlayback` / `releaseVideoPlayback` | the one playing video | every video that starts |
| `pinVideoPlayback` / `hasPinnedVideo` | the docked video, which outranks autoplay | the picture-in-picture dock |
| `claimAudioOutput` / `releaseAudioOutput` | the one sounding player | audio posts, the video core |
| `claimFloatingPlayer` / `releaseFloatingPlayer` | the one floating window | the video dock, the audio bar |

## Enforced by

- `__tests__/fitness/every-downscale-is-named.test.ts`: 59, at zero, for a size or a quality written outside
  `MEDIA_RENDITIONS`; `__tests__/media/media-renditions.test.ts`: 60, for every step against its screen at 3x;
  `__tests__/media/take-photo-asks-for-the-largest.test.ts`: 61; `__tests__/messaging/sealed-media-v2.test.ts`:
  65, for a sealed file cut, reordered or extended refused, and a v1 file still opened;
  `__tests__/utils/camera-constraints.test.ts` and `__tests__/stores/live-session-store.test.ts`: 66, for the lens
  each side opens on and a broadcast's flip; `__tests__/live-streaming/whip-publish.test.ts`: 66, for the ladder.
- `__tests__/fitness/a-media-frame-keeps-its-shape.test.ts`: 21a and 21b, the capture shape, the short's frame
  below `lg` and the camera stage fitted on both axes.
- `__tests__/components/shared/create-tool-rail.test.tsx`: 22c, for every tool in the row with no More, what
  the menus held now in the rows, and a joined take trimming on the timeline;
  `__tests__/components/shorts/editor-viewport-panels.test.tsx`: 22c, for no category folds, and Done and the
  quick row on a real editor's state; `__tests__/components/compose/the-mode-strip-reads-on-its-ground.test.tsx`:
  22a; `__tests__/hooks/use-story-capture.test.tsx`: 22b, for the take handed straight to the editor;
  `__tests__/story/story-compose-flow.test.tsx`: 22d, for the one-tap audiences.
- `__tests__/shorts/standby-shot.test.ts`, `__tests__/hooks/editor/use-video-pair.test.ts` and
  `__tests__/hooks/use-editor-playback.test.ts`: 22e, for the frame the standby holds playing and paused, a
  swap only once its seek has landed, a scrub across a cut that shows the standby, a new file held behind the
  last frame shown, and a file that ends not stopping playback;
  `__tests__/components/shorts/editor/video-preview.test.tsx`: 22e, for both elements bound and painted only
  through the stage.
- `__tests__/components/shorts/editor/canvas-stage.test.tsx`, `__tests__/shorts/compose-picture.test.ts` and
  `__tests__/utils/transition-renderer.test.ts`: 22f, for a gap left dark, a transition blended from the
  outgoing clip's own element, a picture-in-picture, a freeze frame's still, a sticker's own frame, a face
  that lands, a cutout from a second file, a mask and a scene reaching the stage, the playing and still
  pixel density, and each step's backdrop, matte and filter at the frame's scale;
  `__tests__/shorts/canvas-filter.test.ts` and `__tests__/shorts/pixel-filters.test.ts`: 22f, for a filter
  computed as CSS computes it where the canvas ignores one;
  `__tests__/shorts/frame-renderer-timeline-time.test.ts`: 22f, for each exported frame composed at its
  timeline moment; `e2e/video-export.spec.ts`: 22f, for the stage's frames held to the exported file's by
  SSIM, over a wipe, and over two files under a mask, an animated sticker, a picture-in-picture and a green
  screen.
- `__tests__/identity/speech-run.test.ts`: 56, 57, for the gate that opens, holds and closes a word on the
  block clock, drops a click, cuts a voice that never pauses, and never hears a steady room; for the level
  shown above the room, and a clip with no words in it spending no try.
- `__tests__/components/identity/speech-stage.test.tsx`: 57, for the silent and quiet lines and the inputs
  offered while either stands.
- `__tests__/identity/screen-light.test.ts`: 58, for the light that comes on in the dark, holds through a
  flicker at the line, goes out in daylight and stays on once it has had to come back.
- `__tests__/media/a-render-says-what-it-left-out.test.ts`: 55, for every gap the union can carry having
  a line, in every locale, joined into one notice rather than a toast each.
- `__tests__/hooks/use-clip-audition.test.ts` and `__tests__/audio/music-mix.test.ts`: 38, for the track heard
  under the clip and the balance being the two levels.
- `__tests__/audio/preview-bus.test.ts`, `__tests__/hooks/editor/use-playback-audio-sync.test.ts` and
  `__tests__/audio/export-mix.test.ts`: 40a, 40b, 40c, for the stem playing from the playhead, one
  track muting alone in the preview and the export, and the seam fade holding its length.
- `__tests__/audio/time-stretch.test.ts`: 40c, for a tone at 0.5x and 2x keeping its pitch and level.
- `__tests__/hooks/use-music-bed.test.ts`: 38, for the one bed seeking to the authored start, following
  the mixer, rewinding with the stage and holding the speaker.
- `__tests__/media/music-credits.test.ts`, `__tests__/shorts/use-shorts-create-flow.test.ts`,
  `__tests__/hooks/create-content-creators.test.ts`, `__tests__/feed-algorithm/enrich-post-signals.test.ts`,
  `__tests__/components/shorts/shorts-author-overlay.test.tsx`, `__tests__/components/shorts/feed-short-player.test.tsx`
  and `__tests__/components/sounds/sound-detail-page.test.tsx`: 39a, for the songs a mix plays, the deeds a
  post may carry, the credit stored in mix order and with the post, a story's deed read back, a ShareAlike
  song stopping the tap before the reel is seated, and the credit shown on the short and in the feed and
  linked on the sound page;
  `__tests__/rls/a-catalogue-song-is-credited-on-the-post-that-plays-it.test.ts`: 39a in the database;
  `__tests__/live/music-licences.test.ts`: 39a against the live catalogue.
- `e2e/story-music.spec.ts`: 38, 39, 40, by picking a real catalogue track and proving it is audible in the
  editor and present in the audio track of the file the export produces.
- `__tests__/fitness/a-picture-carries-its-hash.test.ts`: 1, 2, 3, for the one canvas that paints a hash, the
  one pass that makes one, and every picture arriving over one but the letterboxed case.
- `__tests__/components/media/feed-image.test.tsx`: 1, for a server picture already decoding staying on screen
  through hydration, one that had painted nothing entering the fade with no transition, and a retry or a
  later mount fading in.
- `__tests__/fitness/a-pinned-tag-lands-inside-its-image.test.ts`: 14.
- `__tests__/components/compose/media-alt-text-sheet.test.tsx` and `__tests__/components/media/media-grid.test.tsx`:
  13 and 13a, for a description taken only when the author takes it, and a model's read out and opened as AI's.
- `__tests__/fitness/an-empty-alt-is-written-on-purpose.test.ts`: 15, for the primitive that defaults the
  claim and the call site that falls back to it when the description is missing.
- `__tests__/fitness/a-sheet-tier-is-mirrored-once.test.ts`: 4, for the one CSS value a `sizes` has to copy.
- `__tests__/fitness/a-size-comes-from-a-column.test.ts`: 4, 5, for the `sizes` nobody writes by hand and the
  responsive image that declares one.
- `__tests__/fitness/a-request-honours-the-devices-pixels.test.ts`: 6, for the factor and for the tiers being
  the avatar boxes themselves.
- `e2e/the-desk-scales.spec.ts`: 4, 12, by reading back what the shorts poster asked the optimiser for
  against the pixels the frame draws at 1440, 1920 and 3840.
- `__tests__/fitness/ranking-and-upload-ssot.test.ts`: 11, for the upload path's single owner.
- `__tests__/fitness/every-decode-reads-the-orientation.test.ts`: 10, for every decode reading the tag the
  re-encode then throws away.
- `__tests__/media/active-video.test.ts`: 26.
- `__tests__/media/sound-intent.test.ts`: 25.
- `__tests__/media/floating-window.test.ts`: 44a, for the projected corner, the throw that dismisses and the
  grip that grows the window away from its anchor.
- `__tests__/hooks/use-shorts-playback-sound.test.tsx`: 25, 25a, for a short starting from the session value,
  writing it, never turning sound on at the next tap, and a speed that stays with its own short.
- `__tests__/components/media/video-control-strip.test.tsx`: 25a, 29, for the four stops and the fill written on
  transform per frame.
- `__tests__/components/media/on-media-scrub-bar.test.tsx`: 29, for the range, the swell as a clip, the fed
  playhead and a drag kept from the swipes above it.
- `__tests__/media/normalize-image.test.ts`: 10, 11, 12.
- `__tests__/media/a-social-photo-is-not-small.test.ts` and `e2e/a-social-photo-is-not-small.spec.ts`: 12, for
  the line sitting where every phone would stretch the photo, the toast naming the width, and the crop
  editor naming it before Apply.
- `__tests__/components/crop-geometry.test.ts`: 12, for the export size the editor reads and the canvas is
  sized to being one number.
- `__tests__/media/a-crop-is-the-file-it-encoded.test.ts`: 9, 11, for the crop's own encode answering a
  substituted PNG and handing back a file marked at the size it drew.
- `__tests__/fitness/a-photo-is-judged-by-the-bytes-it-stores.test.ts` and
  `e2e/a-big-photo-is-judged-after-compression.spec.ts`: 11, for the cap reading the bytes the pass produced
  rather than the ones the person picked.
- `__tests__/media/a-video-post-carries-its-poster.test.ts`: 23, from the capture beside the upload to the
  columns the card reads back.
- `__tests__/fitness/a-poster-is-an-image.test.ts`: 23, for the attribute an element replaced.
- `__tests__/fitness/capture-is-silent.test.ts`: 21, for the shipped sound no capture surface reaches.
- `__tests__/media/a-transcode-does-not-hold-post.test.ts` and `__tests__/utils/video-validation.test.ts`: 22,
  for the encode running behind the attachment and the refusal belonging to the plan.
- `__tests__/hooks/a-crop-cannot-be-silently-dropped.test.tsx`: 9, 11, 12, 12b, for the crop being one encode,
  for a crop that lands under the line saying so, and for the original an edit names staying attached.
- `__tests__/components/compose/composer-media-editors.test.tsx`, `__tests__/components/compose/photo-editor.test.tsx`
  and `__tests__/components/restore-draft.test.ts`: 12b.
- `__tests__/media/a-tier-is-a-quality-not-a-shape.test.ts`: 12, for the floor under every tier's cap.
- `__tests__/components/media/feed-image.test.tsx`: 3, for the tile that holds its painted preview.
- `__tests__/components/media/media-grid-video-row.test.tsx`: 3, 27, 33, for the slot key, the release and
  the reserved frame.
- `__tests__/components/media/video-frame.test.ts`: 33.
- `__tests__/domain/video/video-controls.test.ts`: 34b, 34d, 34e, 34f, 34g, for every key's command and its
  mirror, the skip thirds, a resume that ignores the opening and the end and forgets the oldest, a moment's
  link and what the lock screen names.
- `__tests__/components/media/fullscreen-video-player-gestures.test.tsx`: 25a, 32, 34b, 34d, for a skip that
  chains, a hold that plays fast only while held, the keys, and the playhead handed back on leaving.
- `__tests__/components/media/inline-video-player-keys.test.tsx`: 34a, 34b, for keys on a focused frame and the
  rest of the controls behind More; `__tests__/components/media/inline-video-player-handoff.test.tsx`: 32,
  34a, for the frame carrying on from full screen and a phone frame's one control.
- `__tests__/utils/a-video-can-be-watched-the-way-the-phone-is-held.test.ts`: 34c.
- `__tests__/components/compose/media-preview-grid.test.tsx`: 33, for the composer drawing the same frames.
- `__tests__/components/media/inline-video-player-failure.test.tsx`: 27, 34.
- `__tests__/components/media/fullscreen-video-handoff.test.tsx` and
  `__tests__/components/media/inline-video-player-handoff.test.tsx`: 32, 34.
- `__tests__/components/media/inline-video-player-viewport.test.tsx`: 24, 27.
- `__tests__/fitness/a-recipe-lives-in-styles.test.ts`: 32, for the control recipes.
- `__tests__/fitness/hls-has-one-adapter.test.ts`: 31, for the one loader and the one manifest decision.
- `__tests__/components/shorts/feed-short-player.test.tsx`: 33.
- `__tests__/domain/seek-buffer.test.ts` and `__tests__/components/shorts/shorts-player-failed-start.test.tsx`: 34.
- `__tests__/fitness/there-is-one-waveform.test.ts`: 35, for the one envelope every surface draws through
  and the bar row nothing builds from levels.
- `__tests__/fitness/there-is-one-player.test.ts`: 38, for the browser's own picture-in-picture and the
  dock's stage holding no element of its own.
- `__tests__/fitness/a-sealed-tile-waits-for-the-viewport.test.ts`: 42, 43, for the latched gate on every
  decrypt and the gallery asking whether a tile is sealed before it asks for bytes.
- `__tests__/shorts/avcc-record.test.ts`: 49, for the record itself.
- `__tests__/shorts/muxer-sink.test.ts`: 49, 51.
- `__tests__/shorts/encoder-config.test.ts`: 51, for the configuration every candidate is probed with; 55a,
  for the published file staying H.264 and streamable, the copy taking a hardware HEVC or AV1 encoder and
  2160p where there is one, and the bitrate following motion.
- `__tests__/shorts/source-motion.test.ts`: 55a, for motion read off real encoded files.
- `__tests__/shorts/edit-motion.test.ts`: 55a, for the motion the edit adds and the speed it plays at.
- `__tests__/shorts/frame-supply.test.ts`: 55c, for a seeked HDR frame the engine cannot map named as a gap.
- `__tests__/shorts/mp4-writer.test.ts`: 55b, for a file the disk has no room for held in memory from its
  first byte, and a finished render deleted only where the export wrote it.
- `__tests__/shorts/export-plan.test.ts`: 55a, 55b, for the frame stepped down to what a device can hold and
  finish, and the whole file counted where it cannot be written to disk.
- `__tests__/shorts/use-short-export.test.ts` and `__tests__/shorts/use-shorts-create-flow.test.ts`: 22d, for
  the export in the capsule behind a live editor, a second Next replacing the first, an export stopped when
  its edit is left and held when the edit changed, one export at a time, and a copy never lost before the
  tap that shares it.
- `__tests__/shorts/use-export-matte.test.ts` and `__tests__/utils/ai-cutout.test.ts`: 55d, for every path
  that turns cutout on, and a prefetch that lands or fails saying so.
- `__tests__/fitness/a-chunk-reaches-the-muxer-one-way.test.ts`: 49, 51, for the single write path.
- `tools/export-probe/run.mjs`: 55a, 55b, 55c, run by hand in real Chromium and WebKit, for each file's codec,
  frame, codec level, bitrate, clean decode, playback, index ahead of the media, AAC start, the worker
  writer, and HDR levels from planes, an opaque frame, a seeked element and the recorder path.
- `__tests__/story/sticker-media-geometry.test.ts`: 50, for the one ratio and for a sticker landing on the
  same fraction of the picture as the text beside it.
- `__tests__/story/story-phone-frame.test.ts`: 53, for the phone stage filling the screen, the authored
  frame staying 9:16 clear of the name, music and reply, and a video story not mounting a second clip
  to fill the bars.
- `__tests__/story/sticker-publish-and-gradient.test.ts`: 51, for the embed sitting in the unbaked and
  live-display lists and the bake surface drawing no published kind.

**Not yet enforced**

- `a-clip-carries-its-captions` (30): every playback surface renders a caption track when the clip has one, and
  the default is on. Waits on rule 30 being built: there is no caption column, no `<track>` and no preference
  to read.

## Traps

- 22f: The stage drew text, captions, images, picture-in-picture and masks as DOM and CSS, and transitions not
  at all, so a creator first saw a whip pan in the published file and a mask's feather at a different size.
- 22f: Drawn by the canvas, a GIF sticker is its first frame unless its frames are decoded, and a face the
  editor loads on demand stays in its stand-in until the paused stage is painted again.
- 22f: `ctx.filter` does nothing on WebKit before Safari 18, so a canvas that relies on it shows a cutout's
  blurred backdrop sharp, on the stage and in the file alike.
- 22e: A cut seeked the editor's one element, so the stage held the outgoing clip's last frame for the whole
  seek while the sound carried on; a cut to another take set `src`, which blanked the element and then showed
  that file's first frame before the seek landed.
- 22e: A playing element presents about a frame ahead of the `currentTime` it reports, so a cut judged on
  `currentTime` alone showed a frame of what the edit had cut out.
- 22e: A file that reaches its end pauses its element and fires `pause`, which the editor read as the person
  pausing: playback stopped a frame short of a clip that ended on its file's last frame.
- 59: Posters were cut to 720 px at JPEG 0.82 and covers to the tier cap before the cover cap, so the full-screen
  short's first frame and every banner were softer than the screen they were drawn on.
- 61: `takePhoto` with no size answers the smallest supported size on WebKit and the first-listed on Chromium,
  and a torch pulsed before the snapshot was drawn had already gone out.
- 64: A recorder handed a bitrate it refuses falls back to the browser's own 2.5 Mbps without a word.
- 58: The fill lamp lit only on a face already found too dark, and a face in the dark is never found: the
  person was told to centre a face the camera could not see, in front of a black screen.
- 57: Whisper's encoder in fp16 on WebGPU transcribed every clip as "I'm not going to be a little bit of
  the way", so on Chrome with a GPU no word of a reading was ever heard (`ON_DEVICE_SPEECH_DTYPE` keeps it fp32).
- 57: A reading's duration was kept on the microphone's block clock, 21.33 ms a block at 48 kHz, and the
  route takes whole milliseconds: every reading heard in full was refused with a 400 and dropped to the form.
- 57: A worklet that failed to load threw `AbortError`, which the fault map read as "another app is using
  the microphone"; a person with a muted input was told to speak louder, into a meter that never moved.
- 56: The reading sampled the microphone's level only on new camera frames, through an `AudioContext`
  made after `getUserMedia` resolved, and judged four words from one clip: people read every word and were
  told they could not be heard.
- 55c: A hardware 10-bit frame has no WebCodecs pixel format, so a tone map that only read planes left
  iPhone HEVC to the browser's dim conversion; Chromium's float upload holds its highlights above 1.0.
- 55c: Chromium draws an HLG frame into an sRGB canvas with reference white at 187 of 255, so iPhone HDR
  footage exported nearly a stop darker than it was shot; the tone map reads the planes and puts it at 244.
- 55b: A fragmented MP4 carries no total length; WebKit read a three-second export as two.
- 55: A browser can ship the `AudioEncoder` class with no AAC encoder behind it and answer
  `typeof AudioEncoder !== 'undefined'` exactly as one that can encode does. The muxer then declares an
  `mp4a.40.2` track nothing fills, and the finished file reports zero samples on a track every player
  shows. `MediaRecorder.isTypeSupported('video/mp4')` answers true on the same build.
- 39: Jamendo's CDN serves audio with no `Access-Control-Allow-Origin`, so the picker's own `<audio>`
  element played it perfectly while `fetch` was refused and `createMediaElementSource` mixed silence —
  every video story published a music label over an audio track that never had the song in it.
- 39: Its ranking index outlives its storage objects, so the first row of a live, popular, perfectly-formed
  search result can be a track whose audio 404s.
- 38: `crossOrigin` set after the constructor already took the `src` never reaches the load, so the element
  fetches without CORS and the graph it feeds is silent with no error anywhere.
- 1: An image with no placeholder is a hole the layout has already reserved; the fade has nothing to fade from
  and the row reads as broken until the bytes land.
- 1: `.img-fade` was in the server's HTML, so every picture but the one marked `priority` stayed at opacity 0
  until hydration: on `/home` a faded image was the phone's largest paint at 5.3s p75, the unfaded one 3.1s.
- 3: Keyed on the URL, a just-published card's tile unmounted the instant the ghost row's `blob:` media became
  the published one, and the author watched their own sharp photo fade back through its blurhash.
- 6: Avatars requested 2x on a DPR-2.6 phone and were upscaled: a 0.76x draw of a "retina" image.
- 12: The entry tier's cap was written from a 2x assumption, so a free-tier photo was cut to 1280px at attach
  and enlarged again by the card that cut it — and the "this will look soft" warning could never fire, because
  its own floor sat below the cap.
- 12: The check ran at attach and nowhere else, so a crop zoomed to the editor's own ceiling shipped a 400px
  photo in silence, long after the attach-time toast had been and gone.
- 10: A re-encode through a canvas discards the EXIF tag, so an orientation not applied at decode is lost
  forever. A hash or a dimension read without `imageOrientation` disagrees with the picture it belongs to.
- 23: A `poster` attribute has no srcset and no sizes, so the feed's largest contentful paint was a full-size
  original discovered after the video element mounted.
- 23: Nothing on the post path ever captured a frame — `video_thumbnail_url` was written by the shorts flow
  alone — so every feed video post painted solid black over its whole frame, and the first-frame hold, which
  keys off a poster existing, was off for all of them.
- 11: A crop exported `toDataURL('image/jpeg', 0.9)`, which no pass had registered, so publishing compressed
  it again to WebP 0.85 on top of the JPEG ringing — and held a base64 string a third larger than the file.
- 11: Four pickers — the poll option, the edit-post replacement, the story background and the shorts overlay —
  judged the picked file against the 10MB image cap and re-encoded afterwards, so a 12MB PNG was refused
  before the compressor that stores it as 260KB ever ran.
- 11: The studio's asset library ran no pass at all: a creator's camera original went to R2 exactly as it
  was picked, EXIF and GPS intact, and the cap judged bytes nothing was going to store.
- 9: Three crop call sites relabelled the editor's blob by hand as `image/webp`. A browser with no WebP
  encoder answers `toBlob` with a PNG, so those uploads carried PNG bytes under a WebP type — and none of
  them was marked, so publishing encoded the crop a second time.
- 28: The muted-autoplay retry could not tell the `AbortError` thrown by its own pause on scroll-out from a
  policy block, so it restarted reels off screen: fifteen decoding at once, 29fps.
- 33: The frame's floor was square at every width, so the commonest clip on the product — a 9:16 one shot on
  the phone reading the feed — lost 44% of itself on that phone.
- 34: A short published through the in-browser render shipped an AVC configuration record whose parameter
  sets each carried their NAL header byte twice. Chromium never produced a frame, never raised an error
  and never settled play(), so the surface spun on a start that could not arrive — forever, over the poster.
- 38: A media element pauses the instant it leaves the document, so the dock stage stays mounted on every route
  and the reparent happens in one task. Unmount cleanup runs in a layout effect, because a passive one runs after
  the browser has already paused the removed video.
- 42: Without a viewport gate, an off-screen bubble pulls and decrypts its own full-size original, because
  `messages` has no thumbnail to stand in for it.
- 44: `max-w-full` on a chat media box resolves against the bubble's content box, which the negative margins are
  wider than.
- 48: Firefox's High-profile encoder returned presentation timestamps out of decode order — `DTS went from
  100000 to 33333` — and the muxer's throw, raised inside the encoder's own output callback, landed on
  `window.onerror` 84 times in one night, from `/shorts/create`, `/home` and a profile.
- 49: An unmuted start refused by the autoplay policy was retried muted, but the element already fed the
  recording graph, so WebKit refused that too and the export drew the same first frame until its own stall
  timeout — three times the clip's length, for nothing.
- 51: The baked card froze the post's `1m` age into the pixels forever, upscaled 411px-stage text to a
  1080px frame soft, and showed a shared clip as a still — the card looked like a screenshot because it
  was one.
