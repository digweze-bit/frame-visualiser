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
a **mitred corner sample**, shot square-on with the corner at the top-left
of the frame and the two arms running right and down. The app:

1. Uses that corner photo (rotated 0/90/180/270°) as the four corner tiles.
2. Crops a mid-band strip from the photo as a repeating texture, tiled along
   the straight runs between corners.
3. Scales the frame width, mount width and artwork size using the real mm
   measurements you enter, so relative proportions are accurate regardless
   of on-screen size.

For best results: even, diffuse lighting, plain background, corner filling
most of the frame, minimal glare.

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
