# Crystal Beach Luxury kiosk

Guest welcome screen for Crystal Beach Luxury (Crystal Beach, Destin FL). Host: Hunter.
Same build pattern as the Blue Pearl kiosk (Pi 5 + Chromium kiosk + GitHub Pages + Raspberry Pi Connect).

Public URL: https://bluepearlretreat.github.io/crystal-beach-kiosk/
(One GitHub account can host many Pages sites, one per repo; this sits beside `blue-pearl-kiosk`.)

## Files
- `index.html`: the page. All property settings are in the `CONFIG` block at the top.
- `beach.jpg`: background photo, 1920x1080 crop of Pexels 32942402 ("Seagull strolling on Destin beach shoreline", Esteban Carriazo, free Pexels license).
- `guest-config.json`: `{ "guestName": "...", "checkoutDate": "YYYY-MM-DD" }`, same keys as Blue Pearl's control panel. The page also accepts `name`/`checkout`.

## Background priority
Live webcam (HLS `.m3u8` via hls.js, or a snapshot image URL) > `beach.jpg` > the built-in SVG scene.
Webcam = the Pompano Joe's (Miramar Beach) cam the Blue Pearl kiosk uses. If the stream fails or freezes for 90 s, the photo shows and the cam is retried every 60 s.

### Webcam server auto-find
HDRelay can move a cam between servers (this one moved `watch.hdrelay.io` -> `east2.hdrelay.io` in Nov 2026, and the old address stopped working). For any `*.hdrelay.io/live/<cam>/index.m3u8` URL, `findCamUrl()` runs on every (re)connect:
1. Asks HDRelay's public player API (`watch.hdrelay.io/api/player-profile/<cam>/<CONFIG.hdrelayProfile>`) for `streamUrl`. Browsers currently reject this response because HDRelay sends the `access-control-allow-origin` header twice; if they fix that, this step starts working by itself.
2. Otherwise tries the cam on the last good server, then each name in `CONFIG.hdrelayServers`, and uses the first that returns `#EXTM3U`.

If the cam moves to a server not in `hdrelayServers`, add the name there (and to `CAM_SERVERS` in the Blue Pearl repo). The Mac check `~/camwatch` (see its README) emails when that happens.

## Outage behaviour
Every 15 min the page checks that it can reach the site before reloading. If the internet is down, it keeps the current screen and checks again every minute. The local fallback copy (`file://`) switches back to the live site as soon as the site is reachable.

## TODO before go-live
- [ ] When the repo/Pages site is live, change the "Crystal Beach kiosk" `source` in `~/camwatch/config.json` from this local file to the Pages URL.
- [ ] `wifiSsid` / `wifiPassword`: blank for now. The tile says "Ask your host" and the QR tile is hidden until these are set.
- [ ] `guideUrl`: blank for now. The guide QR is hidden until it is set.
- [ ] `latitude` / `longitude`: approximate Crystal Beach (30.3832, -86.4282). Replace with the exact address if wanted.
- [ ] TV automation (checkin-scheduler, CEC): owner wants it. Needs the **TV model** and the **booking iCal URL**. Copy `~/kiosk/checkin-scheduler.py` + tests from the Blue Pearl Pi (the Pi copy is the source of truth; it is not in the GitHub repo).

## Pi build (done 2026-11-25)
Pi 5 4GB, Raspberry Pi OS Lite 64-bit (trixie), hostname `crystalbeach-kiosk`, timezone America/Chicago, SSH off, managed through Raspberry Pi Connect (Lite) with linger on.
- Xorg + Openbox + Chromium kiosk started by `kiosk.service` (`startx ~/.xinitrc`). `launch-kiosk.sh` shows this site, or the local copy `~/kiosk/index.html` if the site can't be reached.
- Chromium zoom = TV width / 1920 (2 on the 4K Samsung), measured at launch.
- Hourly restart by the root timer `kiosk-restart.timer`.
- `/etc/X11/xorg.conf.d/99-kiosk-kms.conf` points X at `/dev/dri/card1`; without it X fails on the Pi 5 with "No devices detected".
- Webcam playing at 4K: about 56°C, no throttling.
