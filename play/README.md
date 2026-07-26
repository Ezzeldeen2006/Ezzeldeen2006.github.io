# Playable WebGL builds go here

Drop a Unity WebGL build at `public/play/<slug>/` so it is served as
`https://ezzsalah.me/play/<slug>/index.html`, then set that path as the project's
`liveUrl` in the API.

## Why `/play/` and not `/games/`

`/games/<slug>/` is already a **route** — the project detail page. A build placed at
`public/games/atomball-climber/` would export to the same
`out/games/atomball-climber/index.html` as that page, and one would silently overwrite
the other. `/play/` sits outside the route namespace, so there is nothing to collide with.

## How it gets embedded

A `liveUrl` starting with `/` is what makes `components/playable-embed.tsx` embed the
build rather than render an external link — see `lib/playable.ts`. An absolute URL to
another host falls back to a link, because we cannot promise anything about a third
party's framing rules, latency or uptime.

The embed is **click-to-load and states the size**. Do not change it to autoload: the
build is tens of megabytes and most visitors came to read, not to play. On mobile data,
autoloading would be genuinely rude.

## Never serve a build from the laptop

Locked decision #7. Off a 5400 RPM disk over residential upload, a 30 MB download would
be the slowest thing on the site by an order of magnitude, and it would make the laptop's
uptime a prerequisite for playing. `liveUrl` is a plain URL column precisely so the API
does not care where the build lives.

## Size limits that will actually stop you

- **GitHub blocks any single file over 100 MB.** Unity's `.data` file is usually the
  largest artefact and the one that hits this first.
- Pages sites are capped around **1 GB published**, with a soft 100 GB/month bandwidth
  limit. One or two builds is fine; committing every iteration is not — each rebuild adds
  another full copy to git history, permanently.

## Build settings (Unity 6, Web platform)

| Setting | Use | Why |
|---|---|---|
| **Development Build** | **off** | Ships the profiler and debug symbols. Dramatically larger and slower. |
| **Code Optimization** | **Disk Size with LTO** | Download size dominates the visitor experience here; runtime speed does not. Costs a much slower build. |
| **Texture Compression** | **DXT** | The desktop GPU format, supported across desktop browsers. ETC2 and ASTC are mobile formats that fall back badly on desktop. |
| **Client Browser Type** | irrelevant | Only chooses which browser *Build and Run* launches locally. No effect on the output. |

In **Player Settings → Publishing Settings**:

- **Compression Format: Brotli**, with **Decompression Fallback enabled**. GitHub Pages
  will not send `Content-Encoding: br`, so without the fallback the loader receives
  compressed bytes it cannot read. The fallback ships a small JS decompressor that works
  on any host.
- **Data Caching: on** — caches into IndexedDB so a repeat visit does not re-download.

In **Player Settings → Optimization**:

- **Managed Stripping Level: High**, **Strip Engine Code: on**. Usually the single largest
  size win.
- **Exception Support: Explicitly Thrown Exceptions Only**. Full support is much larger;
  "None" is smallest but makes any runtime error impossible to diagnose.

Judge the result by the **transferred size in the browser's Network tab**, not by the
folder size on disk.
