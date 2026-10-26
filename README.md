# Live Tennis Scores for Kodi

A Kodi add-on that pops up a notification when a tennis match changes, and opens a scoreboard of the matches in play. Scores come from the [Live Tennis API](https://livetennisapi.com).

## What it shows

The add-on has two parts. A background service checks the live match list on a timer and raises Kodi notifications. A scoreboard window opens when you launch the add-on from Kodi's program add-ons.

Notifications, each one a Kodi popup with the add-on icon:

- "Now on court" when a match appears in the live list, with the tournament name.
- "Set complete" when a player's set count goes up, with the set scores so far.
- "Match finished" with the last score seen. The winner's name is shown only when the feed named one. A retirement or walkover is shown in brackets.
- "Break point" when the receiver is a point from breaking serve. This one is off by default.

A break point is read from the score. It means the receiver holds advantage, or the receiver is at 40 while the server is at 0, 15 or 30. It is never raised during a tiebreak, and at most once per game.

The service caps itself at five notifications per check. Anything past that becomes one line, "and N more changes".

The scoreboard is a plain text window in a monospace font. Each match gets a block with the tournament and round, the two names, the set scores, the current game's points and two markers. `*` stands in front of the player serving and `BP` flags a break point. Tiebreak points are prefixed `TB`. If the scores came from the service's last check instead of a fresh request, a footer says how many minutes old they are.

The scoreboard has its own text for each state it can end up in:

- no matches in play
- no API key in the settings
- the API key was not accepted
- daily request limit reached
- plan without the live list
- the API answered with an error
- the API could not be reached

There is no tour filter. The add-on shows every match the API's live list returns. The API's `tour` field can be atp, wta, challenger, itf or juniors.

## Requirements

- Kodi 20 (Nexus) or later. `addon.xml` requires `xbmc.python` 3.0.1, which Kodi 20 was the first release to ship.
- A Live Tennis API key. The free tier is enough: the add-on calls only `GET /matches?status=live`, which is a FREE endpoint. The free tier allows 100 requests a day and 30 a minute, and needs no card. Get one at [livetennisapi.com/subscribe/free](https://livetennisapi.com/subscribe/free).
- Nothing else. The add-on uses only Python's standard library and declares no dependency beyond `xbmc.python`.

## Install

There is no Kodi repository and no release zip yet. Make a zip yourself, or copy the folder in by hand.

### From a zip file

Kodi needs the zip to contain one top level folder named `script.livetennisscores`.

```sh
git clone https://github.com/livetennisapi/script.livetennisscores.git
zip -r script.livetennisscores.zip script.livetennisscores -x '*.git*'
```

Then in Kodi:

1. Settings, System, Add-ons. Turn on "Unknown sources".
2. Settings, Add-ons, "Install from zip file". Pick `script.livetennisscores.zip`.
3. Open the add-on's settings and paste your API key.

### By copying the folder

Copy the `script.livetennisscores` folder into Kodi's add-ons directory. On Linux that is `~/.kodi/addons/`. Restart Kodi. If Kodi lists the add-on as disabled, enable it under My add-ons, Program add-ons.

The service starts with Kodi. With no key it stays idle and logs one line saying so.

## Settings

Settings, Add-ons, My add-ons, Program add-ons, Live Tennis Scores, Configure.

| Setting | Default | Range | What it does |
| --- | --- | --- | --- |
| API key | empty | text | Your key from your livetennisapi.com account. Leave it empty and the add-on does nothing. |
| Minutes between checks | 20 | 15 to 240, in steps of 5 | How often the service reads the live list. |
| Tell me when a match goes on court | on | on or off | The "Now on court" notification. |
| Tell me when a set is won | on | on or off | The "Set complete" notification. |
| Tell me when a match finishes | on | on or off | The "Match finished" notification. |
| Tell me about break points | off | on or off | The "Break point" notification. |
| Seconds a notification stays on screen | 6 | 2 to 30 | How long each popup is shown. |

The service re-reads the settings every 30 seconds. A new check interval takes effect within that time. After you paste in a key, the service uses it at the next scheduled check. The scoreboard uses it straight away.

The 15 minute floor is enforced in code, not only by the slider. A value edited by hand below 15 is read as 15, and a value above 240 as 240.

## How it refreshes

One check is one request to `GET /matches?status=live` with the largest page size the API allows, 200. If the response says more pages exist the service reads up to three pages in that check. The socket timeout is 4 seconds, so a slow network costs one skipped check.

The free tier allows 100 requests a day. At the default 20 minutes a full day costs 72 requests. At the 15 minute floor it costs 96. That is why 15 is the fastest the add-on will go.

Opening the scoreboard normally costs nothing. The service writes each check's match list to a cache file, and the scoreboard reads that file when it is younger than the check interval. Only an older cache triggers a request of its own.

Some consequences of a check every 15 to 20 minutes:

- A break point lasts a few points, so the service catches only some of them. Each check sees one moment of one game.
- A finished match is inferred, not observed. The live list only ever holds matches in progress. So a match must be missing from three checks in a row before "Match finished" is raised. At the default interval that is an hour after it dropped out. A feed outage longer than that can produce a finished notification for a match still being played.
- The first check after Kodi starts announces nothing. Otherwise every match already on court would arrive at once.

When the API answers 429 the service skips that check and logs it. When the key is rejected with 401 it raises one notification and stays quiet until you change it. When the plan does not include the live list, 403, it logs an error. Network failures and malformed responses are logged at debug level and the next check goes ahead as scheduled.

## Privacy

The key is stored in this add-on's settings on the device, in Kodi's `userdata/addon_data/script.livetennisscores/` folder. It is sent as the `X-API-Key` header to `api.livetennisapi.com` and nowhere else. It is never put in a URL and never written to the Kodi log.

The same folder holds `live_matches.json`, the last match list the service fetched. That is the only file the add-on writes.

The add-on contacts no other host and sends nothing about you or your Kodi install. Log lines go to Kodi's debug level and name the request path and the HTTP status only.

## Who makes this

The add-on is written by the operator of the Live Tennis API, the service it reads from.

## Licence

GPL-3.0-or-later. See [LICENSE](LICENSE).

## Links

- API docs: <https://docs.livetennisapi.com>
- Pricing: <https://livetennisapi.com/pricing>
- Free API key: <https://livetennisapi.com/subscribe/free>
- GitHub org: <https://github.com/livetennisapi>
