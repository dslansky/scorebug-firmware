# ScoreBug firmware releases

This repository is where ScoreBug boards look for software updates. It holds
only finished firmware images and the manifest that points at the newest one.
The source code lives in a separate, private repository.

**Don't commit to this repository by hand.** Releases are made by
`esp32/sports-display/scripts/publish-standalone.sh` in the source repository
(branch `scoreboard-pro`), run on the Mac that holds the signing key.

## What's here

| Path | What it is |
|---|---|
| `latest.json` | The current release: version, image URL, size, SHA-256, signature and rollout share. Boards read this about once an hour. |
| `releases/standalone-<version>.bin` | One firmware image per published version. |

Until the first release is published there is no `latest.json`, and boards
treat the update check as "nothing to install".

## How boards decide to install

A board installs the release in `latest.json` only when all of these hold:

1. The version is newer than the one it runs. Versions compare number by
   number (`2026.09.25.10` is newer than `2026.09.25.9`), and an older release
   is never installed, so a board can't be rolled back by replaying one.
2. The signature is valid for the ScoreBug public key built into the board.
   It covers the version and the image's SHA-256. A release that isn't signed
   with the ScoreBug key is refused, whoever serves it.
3. The board falls inside the release's `rollout` share (1-100%). A board's
   place comes from its hardware address, so the same boards go first every
   time.
4. The downloaded image matches the signed SHA-256. It's written to the
   board's spare slot and only started after that check. A bad or incomplete
   download is discarded, and the board keeps running the version it had.

Because the signature carries the trust, this repository can be public, and
must be: boards download from it without logging in.

## Releasing

From `esp32/sports-display` in the source repository, on the publishing Mac:

```
./scripts/publish-standalone.sh "what changed" 10    # 10% of boards first
./scripts/set-rollout.sh 100                         # then everyone
```

Bump `CHAIKIN_STANDALONE_VERSION` in `platformio.ini` before each release. The
script builds the image, signs it, checks the signature, and pushes to this
repository. It refuses to run if the signing key is missing or isn't the key
the boards trust. It also refuses if the image contains anything from the
household configuration.

The script expects this repository cloned at `~/projects/scorebug-firmware`,
or set `SCOREBOARD_RELEASE_DIR` to point elsewhere.

## Setup release: 2026.09.27.6

This release improves first-time setup:

- Protected Wi-Fi requires a password; a saved password is reused only for
  the same network. Known open networks allow a blank password.
- Incomplete saves and failed connection tests leave previous choices intact.
- Setup remains available once a phone has joined it.
- Progress messages explain the connection test, which allows up to 40 seconds.
- Save-triggered restarts do not accidentally activate the three-power-cycle
  setup gesture.

The setup network is **ScoreBug-XXXX**. If the page does not open by itself,
open **http://192.168.67.1** in Safari or Chrome while connected to that network.
**No Internet** is normal during setup. Enter the home or office Wi-Fi password,
choose leagues and teams, then tap **Save and connect**.

The release passed 27 regression checks. Fresh setup on the product test board
was verified through password acceptance, saved teams, restart, reconnection,
and repeated score downloads. The automatic phone popup is not guaranteed;
manual browser access was verified. A separate physical unplug/replug check
was not recorded.

## Home boards are a separate channel

This repository is only for ScoreBug product builds. The household boards use
`dslansky/scoreboard-firmware`, a separate repository and manifest URL. Publishing
here must not change that channel or flash any household board.
