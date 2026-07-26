# Playable WebGL builds go here

Drop a Unity WebGL build at `public/games/<slug>/` so it is served as
`https://<site>/games/<slug>/index.html`, then set that path as the project's
`liveUrl` in the API.

A path starting with `/` is what makes `components/playable-embed.tsx` take the
embed path instead of rendering an external link, so the build must be
same-origin — deployed with the frontend, on the CDN.

**Never serve a build from the laptop.** A Unity WebGL build is 20-40 MB; off a
5400 RPM disk over residential upload it would be the slowest thing on the site
by an order of magnitude, and it would make the laptop's uptime a prerequisite
for playing. The whole point of `liveUrl` being a plain URL column is that the
API does not care where the build lives.

The embed is click-to-load and states the size, so nothing downloads for the
majority of visitors who came to read rather than play.

Planned: **AtomBall Climber** (`/games/atomball-climber`) — the only one of the
three Unity projects small enough to ship this way.
