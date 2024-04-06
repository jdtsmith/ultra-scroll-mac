---
author: J.D. Smith
---

- [ultra-scroll-mac: scroll emacs-mac like
  lightning](#ultra-scroll-mac-scroll-emacs-mac-like-lightning)
  - [Compatibility, Installation, and
    Usage](#compatibility-installation-and-usage)
  - [Configuration](#configuration)
  - [Related packages and
    functionality](#related-packages-and-functionality)
  - [Questions](#questions)
    - [What was the motivation behind
      this?](#what-was-the-motivation-behind-this)
    - [Why is this emacs-mac only?](#why-is-this-emacs-mac-only)
    - [So what should I use for other Emacs
      builds?](#so-what-should-i-use-for-other-emacs-builds)
    - [How does this compare to the built-in smooth
      scrolling?](#how-does-this-compare-to-the-built-in-smooth-scrolling)
    - [Why are there so many smooth scrolling modes? Why is this so
      hard, it's just
      *scrolling*?](#why-are-there-so-many-smooth-scrolling-modes-why-is-this-so-hard-its-just-scrolling)
    - [What should I know about developing scrolling modes for
      Emacs?](#what-should-i-know-about-developing-scrolling-modes-for-emacs)

# ultra-scroll-mac: scroll emacs-mac like lightning

`ultra-scroll-mac` is a smooth-scrolling package for
[emacs-mac](https://bitbucket.org/mituharu/emacs-mac). It provides
highly optimized, pixel-precise smooth scrolling which can readily keep
up with the *very* high event rates of modern trackpads and
high-precision wheel mice. You move your fingers, the page responds,
instantly:

<https://github-production-user-asset-6210df.s3.amazonaws.com/93749/290018933-ed5cf414-eab5-4ba8-b077-30cac0c5ace0.mov>

Importantly, it can cleanly *scroll right across* tall images and other
jumbo lines – a perennial problem with scrolling packages to date. As a
bonus, it enables relatively smooth scrolling even with dumb third party
mice.

> [!NOTE]
> **Do you need this?**
>
> If you don't scroll with a high-speed device (modern mouse or
> trackpad), no. If you do, but aren't sure, here's a good test to try:
>
> Open a big, heavy-duty emacs buffer, full screen. While scrolling
> smoothly such that lines move across your window's height in a few
> seconds, *can you easily read the content you see*, without stopping?
> Is the text as clear scrolling down as it is up? Now, try this
> exercise again with your browser – I bet it's *very* readable there.
> Shouldn't emacs be like this?
>
> If you scroll buffers with large images, this is also a good reason to
> give a try.

## Compatibility, Installation, and Usage

> [!WARNING]
> This is only for the
> [emacs-mac](https://bitbucket.org/mituharu/emacs-mac) port (see
> [why](https://github.com/jdtsmith/ultra-scroll-mac#why-is-this-emacs-mac-only)).
> `M-x emacs-version` should mention `Carbon`, not `NS`, and
> `window-system` should be `mac`.

**Install**: simply `git clone` (or `package-vc-install`) and setup
like:

``` commonlisp
(use-package ultra-scroll-mac
  :if (eq window-system 'mac)
  :load-path "~/code/emacs/ultra-scroll-mac" ; if you git clone'd
  :init
  (setq scroll-conservatively 101 ; important!
        scroll-margin 0) 
  :config
  (ultra-scroll-mac-mode 1))
```

**Usage**: just start scrolling :).

## Configuration

There is little to no configuration. If desired for use with dumb mice,
the variable `ultra-scroll-mac-multiplier` can be set to a number
smaller or larger than `1.0` to decrease/increase mouse-wheel scrolling
speed. Note that many fancier mice have drivers that *simulate*
trackpads, so this variable will have no effect on them. For these, and
for the trackpad itself, scrolling speed should be configured in MacOS
settings.

To reduce the likelihood of garbage collection during scroll, which can
introduce slight pauses, the value of `gc-cons-percentage` is
temporarily increased. The defaults should work well for most
situations, but if necessary, can be configured using
`ultra-scroll-mac-gc-percentage` and `ultra-scroll-mac-gc-idle-time`.

## Related packages and functionality

emacs-mac's own builtin `mac-mwheel-scroll`  
This venerable code has been providing smooth scrolling on
[emacs-mac](https://bitbucket.org/mituharu/emacs-mac/) by default for
nearly a decade.

`pixel-scroll-precision-mode`  
New, fast pixel scrolling by Po Lu, built in to Emacs as of v29.1 (see
`pixel-scroll.el`). `ultra-scroll-mac` was initially based on its
design, but many elements have changed. The core scrolling functions
have been contributed back upstream.

`pixel-scroll-mode`  
A simpler line-by-line pixel scrolling mode, also found in
`pixel-scroll.el`.

[good-scroll](https://github.com/io12/good-scroll.el)  
An update to the simple `pixel-scroll-mode` with variable speed.

[sublimity](https://github.com/zk-phi/sublimity)  
Includes smooth scrolling based on sublime editor.

## Questions

### What was the motivation behind this?

Picture it: a fast new laptop and 5K monitor with a large heavy-duty,
full-screen buffer in `python-ts-mode`. Scrolling with a decent mouse is
mostly OK, but pixel scrolling with the trackpad is just… *painful*.
Repeated attempts to rationalize this fail, especially because it's
notably worse in one direction than the other. Scrolling Emacs feels
like moving through (light) molasses. No bueno.

Checking into it, the smooth scroll event callback takes 15-20ms
scrolling in one direction, and 3–5x longer in the other. Perfectly fine
for normal mice which deliver a few scrolling events a second. *But
trackpad scroll events are arriving every 15ms or less*! The code just
couldn't keep up. Hence the molasses.

I also wanted to be able to peruse image-rich documents without worrying
about jumpy/loopy scrolling behavior. And my extra dumb mouse didn't
work well either: small scrolls did nothing: you'd have scroll pretty
aggressively to get any movement at all.

How hard could it be to fix this? And the adventure began…

### Why is this emacs-mac only?

Only the emacs-mac port exposes the full pixel-level scrolling event
stream of trackpads (and fancy mice). This makes `ultra-scroll-mac` much
simpler than packages which have to simulate this.

### So what should I use for other Emacs builds?

I recommend the built-in `pixel-scroll-precision-mode`. The core
scrolling functions used in `ultra-scroll-mac` may be directly useful,
and have been contributed upstream for potential inclusion.

### How does this compare to the built-in smooth scrolling?

In addition to fast scrolling, the built-in
`pixel-scroll-precision-mode` (new in Emacs v29.1) effectively simulates
a *feature-complete trackpad driver* in elisp, complete with scroll
interpolation, a timer-based *momentum* phase, etc. Since all of this is
handled by the OS for emacs-mac, it's not necessary to include.

Compared to the built-in precision scrolling, `ultra-scroll-mac`
obviously works correctly with emacs-mac, but is also even faster, and
can smoothly scroll past tall images.

### Why are there so many smooth scrolling modes? Why is this so hard, it's just *scrolling*?

Emacs was designed long before mice were common, not to mention modern
high-resolution trackpads which send rapid micro-updates ("move up one
pixel!") more than 60 times per second. Unlike other programs, Emacs
insists on keeping the cursor (point) visible at all times. Deep in its
redisplay code, Emacs tracks where point is, and works diligently to
ensure it never falls outside the visible window. It does this not by
moving point (that's the user's job), but by moving the *window*
(visible range of lines) surrounding point.

Once you are used to this behavior, it's actually pretty nice for
navigating with `C-n` / `C-p` and friends. But for smooth scrolling with
a trackpad or mouse, it is *very problematic* – nothing screams "janky
scrolling" like the window lurching back or forth half a page during a
scroll. Or worse: getting caught in an endless loop of
scroll-in-one-direction/jump-back-in-the-other.

So what should be done? The elisp info manual (`Textual Scrolling` /
`set-window-start`) helpfully mentions:

> …for reliable results Lisp programs that call this function should
> always move point to be inside the window whose display starts at
> POSITION.

Which is all well and good, but *where* do you find such a point, in
advance, safely *inside the window*? Often this isn't terribly hard, but
there is one common case where this admonition falls comically flat:
scrolling past images which are *taller than the window* – what I call
**jumbo lines**. Where can I place point *inside the window* when a
jumbo line occupies the entire window height?

As a result of these types of difficulties, pixel scrolling codes and
packages are often quite involved, with much of the logic boiling down
to a stalwart and increasingly heroic pile of interwoven attempts to
*keep the damn point on screen* and prevent juddering and looping as you
scroll.

### What should I know about developing scrolling modes for Emacs?

For posterity, some things I discovered in my own mostly-victorious
battle against unwanted recentering during smooth scroll, including
across jumbo lines:

- `scroll-conservatively=101` is very helpful, since with this Emacs
  will "scroll just enough text to bring point into view, even if you
  move far away". It does not defeat recentering, but makes it… more
  manageable.
- You cannot let-bind `scroll-conservatively` for effect, as it comes
  into play only on redisplay (after your event handler returns).
- Virtual Scroll:
  - `vscroll` – a virtual rendered scrolling window hiding below the
    current window – is key to smooth scrolling, and setting `vscroll`
    is incredibly fast.
  - There is plenty of `vscroll` room available, including the entirety
    of any tall lines (as for displayed images) in view.
  - `vscroll` can sometimes place the point off the visible window (I
    know, sacrilege), but more often triggers recentering.
- Scrolling asymmetry:
  - `vscroll` is purely one-sided: you can only access a vscroll area
    *beneath* the current window view; *there is no negative vscroll*.
  - Unlike `window-start`, `window-end` does not get updated promptly
    between redisplays and cannot always be trusted.
  - For these two reasons, smooth scrolling up and scrolling down are
    *not symmetric* with each other (and will likely never be). You need
    different approaches for each.
  - If the two approaches for scrolling up and down perform quite
    differently, the user will feel this difference.
- For avoiding recentering, naive movement doesn't work well. You need
  to learn the basic layout of lines on the window *before redisplay*
  has occurred.
- The "usable window height" deducts any header and the old-fashioned
  tab-bar, but *not* the tab-bar-mode bar.
- Jumbo lines:
  - Scrolling towards buffer end:
    - When scrolling with jumbo lines towards the buffer's end (with
      `vscroll`), simply keep *point on the jumbo line* until it
      disappears from view. As a special case, Emacs will not re-center
      when this happens.
    - This is *not* true for lines that are smaller than the usable
      window height. In this case, you must avoid placing point on any
      line which falls partially out of view.
  - Scrolling towards buffer start:
    - When scrolling up past jumbo lines, using `set-window-start`
      (lines of content move down), you must keep point on the jumbo,
      but *only until it clears the top of the window area* (even by one
      pixel).
    - After this, you must move the point to the line above it (and had
      better insist that `scroll-conservatively>0` to prevent
      re-centering).
    - In some cases (depending on truncation/visual-line-mode/etc.),
      this movement must occur from a position beyond the first full
      height object (which may not be at the line's start). E.g. one
      before the visual line end.
- `pos-visible-in-window` doesn't always work near the window
  boundaries. Better to use the first line at the window's top or
  directly identify the final line (both via `pos-at-x-y`) and adjust
  from there.
- Display bugs
  - There are [display
    bugs](https://debbugs.gnu.org/cgi/bugreport.cgi?bug=67533) with
    inline images that cause them to misreport pixel measurements and
    positions sometimes.
  - These lead to slightly staccato scrolling in such buffers and
    `height=0` gets erroneously reported, so can't be used to find
    beginning of buffer. Best to guard against these.
  - **Update:** Two display bugs have been fixed in master as of Dec,
    2023, so scrolling with lots of inline images will soon be even
    smoother.

So all in all, quite complicated to get something that works as you'd
hope. The cutting room floor is littered with literally dozens of
almost-but-not-quite-working versions. I'm sure there are many more
corner cases, but the current design gets most things right in my usage.
