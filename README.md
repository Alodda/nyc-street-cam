# Midtown Camera Wall

A single-file viewer for New York City's public DOT traffic cameras, focused on the Midtown / Hell's Kitchen cluster around W 53rd St.

**Live: https://alodda.github.io/nyc-street-cam/**

![Midtown Camera Wall showing the Broadway @ 51 St feed](docs/preview.png)

## What it does

- Opens on **Broadway @ 51 St**, the fastest camera on that block — a new frame roughly every second
- 24 cameras grouped by area: the five nearest W 53rd St (with distance), Midtown, the four-way Times Square quad, and a handful elsewhere in the city
- **Single** or **Quad** layout — quad shows the selected camera plus the next three, all refreshing together
- Refresh rate: 1s / 2s / 5s / Hold
- Live / stale / offline indicator — amber after 6s with no new frame, red after three failed pulls
- Arrow keys change camera, space toggles Hold

## Data source

Frames come straight from the NYC Traffic Management Center's public endpoint. No API key, no rate limit encountered at one request per second.

```
https://webcams.nyctmc.org/api/cameras/              # all 983 cameras, JSON
https://webcams.nyctmc.org/api/cameras/{id}/image    # current frame, JPEG
```

Each frame is 352×240 and about 15 KB, with the capture time burned into the top of the image. That burn-in is the real liveness check — the status line in the UI only reports what the page has managed to fetch.

Measured refresh cadence on the W 53rd cluster:

| Camera | Distance from W 53rd & Broadway | New frame every |
| --- | --- | --- |
| Broadway @ 51 St | 56 m | ~1s |
| 7 Av @ 54 St | 249 m | ~2s |
| 50 St btwn 8 Av & Bway | 118 m | ~2.5s |
| 8 Av @ 49 St | 235 m | ~2.5s |
| 7 Av @ 49 St | 225 m | ~3s |

## How it's built

One `index.html`, no build step, no dependencies beyond two Google Fonts. Each tile is two stacked `<img>` elements; the next frame loads into the hidden one and only swaps once it has decoded, so the image never flickers or flashes white between refreshes.

## Known limitation

The camera endpoint returns no `Access-Control-Allow-Origin` header. Displaying frames in an `<img>` works fine, but anything that reads pixels does not — drawing a frame to a canvas taints it, so `toBlob()` and `captureStream()` both throw. That rules out in-page recording, timelapse export, or any client-side computer vision. Doing any of that needs a small proxy re-serving the frames same-origin.

The full camera list is fetched from the same origin-less endpoint, so it can't be loaded at runtime either; the 24 cameras here are embedded in the page. Adding more means pasting another entry into the `CAMS` array — name, UUID, coordinates.

## What these cameras are not

These are traffic cameras: low-resolution, fixed, mounted high on a mast and pointed down a roadway. They show traffic flow and weather. They cannot resolve a face, a license plate, or a person. There is no public camera network in New York that can.

## Running locally

Clone and open the file — no server required.

```bash
git clone https://github.com/Alodda/nyc-street-cam.git
open nyc-street-cam/index.html
```

Safari can block remote images on a `file://` page; Chrome does not.

## License

MIT. Camera imagery is published by the New York City Department of Transportation and is subject to their terms, not this license.
