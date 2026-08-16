# Hourglass Frame Visualiser

Single-file HTML app for previewing artwork inside different frame moulding
options, with an accurate to-scale render and a carousel to flip between
frame choices for the same piece.

## Backend

Shares the **hourglass-gallery** Supabase project. New tables (prefixed
`fv_` so they don't collide with the existing HG Cat schema):

- `fv_frames` — moulding library (name, face width mm, rebate depth mm, corner sample photo)
- `fv_artworks` — artwork library (title, size mm, medium, frame/mount requirement)
- `fv_frame_selections` — (not currently used by the UI, reserved for saving named carousels later)

Storage buckets: `fv-frames`, `fv-artworks` (public read, authenticated write).

Auth: Supabase email/password, same pattern as the Auction Tracker. Sign up
from the app itself to create a team login, or add users directly from the
Supabase dashboard.

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
