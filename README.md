# Hourglass Frame Visualiser

Single-file HTML app for previewing artwork inside different frame moulding
options, with an accurate to-scale render and a carousel to flip between
frame choices for the same piece.

## Backend

Shares the **hourglass-gallery** Supabase project. New tables (prefixed
`fv_` so they don't collide with the existing HG Cat schema):

- `fv_frames` — moulding library (name, face width mm, rebate depth mm, texture photo). Fully editable and deletable from the Frames tab.
- `fv_artworks` — artwork library (title, size mm, medium, frame/mount requirement)
- `fv_frame_selections` — (not currently used by the UI, reserved for saving named carousels later)

Storage buckets: `fv-frames`, `fv-artworks` (public read, authenticated write).

Auth: Supabase email/password, same pattern as the Auction Tracker. Sign up
from the app itself to create a team login, or add users directly from the
Supabase dashboard.

## Camera capture

When adding an artwork, "Take photo with camera" opens a live camera view
(rear camera preferred, on devices that have one) instead of requiring an
existing file — capture, retake if needed, then use the shot directly.
Requires the browser's camera permission for the site, and only works over
HTTPS (which the deployed Vercel URL already is).

## Units

Every measurement is stored in **mm** in Supabase — that never changes. The
**in / cm toggle** in the top bar only affects what's typed into and shown
in the forms; it defaults to inches and remembers your last choice (stored
locally in the browser, per device). Switching units live-converts whatever
you have currently typed in an open form, so you don't lose in-progress
values.

## How the frame preview is built

There's no true 3D render — instead, for each frame you upload one photo of
the moulding and select a clean flat patch from it as a repeating texture.

**Best source: about 4 inches of a straight, flat length of the moulding**,
shot square-on, laid flat, evenly lit — no corner needed. A corner sample
also works (just crop a flat section away from the corner itself), but a
straight length is simpler to shoot consistently since there's no depth or
perspective to worry about.

The app then:

1. Lets you drag a crop box over a clean patch of that photo (avoiding
   background, edges, glare).
2. Tiles that patch as a repeating texture along all 4 sides of the frame,
   scaled to the frame's real face width in mm.
3. Renders the 4 corners **geometrically** as true 45° mitre cuts — the same
   way a real frame is built — rather than trying to reuse a photographed
   corner, which can't be warped into 4 different orientations without
   distortion (a mitre has a face, a step, and a rebate lip at different
   depths, so a flat photo of one corner can't stand in for all four).
4. Scales frame width, mount width, and artwork size using the real mm
   values you enter, so proportions stay accurate regardless of on-screen
   size.
5. Uses the frame's **rebate depth (mm)** to scale the inner shadow cast at
   the mount/artwork opening — a deep rebate throws a more pronounced step
   than a shallow one. This is the one field besides width that actually
   changes the render; **notes is plain reference text** (supplier, price,
   whatever's useful to you) and isn't read by the app.

## Layered frames

The Visualise tab can build a "look" from up to 3 frames nested one inside
another (outer / middle / inner) — a common gallery technique. Each frame in
the stack is rendered as its own true mitred layer at its own face width, so
proportions stay accurate; the artwork (and mount, if enabled) sits inside
the innermost one. Each look is one entry in the carousel, so you can flip
between single frames and layered combinations side by side.

## Tray / shadow box frames

A tray frame is geometrically different from a flat moulding: the artwork
sits recessed into the frame, and there's a second surface — the riser (the
frame's inner wall) — between the flat outer face and the artwork. Because
that riser is a separate (usually vertical) surface, it can't be represented
by tiling the same face texture at an angle; it needs its own treatment.

When adding or editing a frame, choose **Tray / shadow box** as the style
and enter a **riser height**. The riser renders as its own nested band,
shaded as a shadowed vertical wall (a flat, uniform shadow tone) rather than
the raking-light gradient used for the flat face — that gradient simulates
a surface catching light at an angle, which doesn't apply to a straight
riser wall.

You can optionally upload a **separate photo of the riser** for a closer
match — useful since it's genuinely a different surface with its own
lighting. If you skip it, the app auto-approximates the riser by reusing
the face texture with a heavier darkening applied.

## Deploy

Static site, no build step.

```
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/digweze-bit/hourglass-frame-visualiser.git
git push -u origin main
```

Then import the repo in Vercel (Framework Preset: **Other**, no build command,
output directory `.`).
