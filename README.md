# JAVIS Dashboard

A "Jarvis"-styled personal briefing dashboard: a dark HUD with a glowing orb that
listens for **two claps followed by the phrase "Wake up, Daddy is home"**, then
speaks and displays a briefing pulled from four sources:

- USD/ZMW exchange rate (Bank of Zambia reference)
- This week's email, with the items that need attention flagged
- Upcoming Google Calendar events
- A summary of Tayant Pay transactions for the selected month, from Google Drive

## Running it

Open `index.html` through a local web server (not `file://`, since the wake-word
flow needs microphone access, which browsers only grant on `https://` or
`http://localhost`):

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

Click **ENABLE MICROPHONE** once to grant access, then clap twice and say the
wake phrase. If the browser doesn't support `SpeechRecognition` (Web Speech
API — Chrome/Edge only) or you'd rather skip the voice trigger, click the same
button again to trigger the briefing manually.

## How the data gets there

`data/briefing.json` is the single source of truth the page renders (with an
embedded fallback copy inline in `index.html` for the `file://` case). It is
**not** fetched live from Gmail/Calendar/Drive/Bank of Zambia in the browser —
those all require authenticated, server-side API calls that can't safely run
from client-side JavaScript. Instead, `briefing.json` is meant to be
regenerated periodically by a backend job (e.g. a scheduled Claude session,
or a small script using the Gmail/Calendar/Drive APIs and a forex source)
and dropped in place. The current file was populated from a real pull on
2026-08-31.

## What's simulated vs. real right now

- **Real, as of generation time:** the forex rate, email summary, calendar
  events, and Tayant Pay ledger totals in `data/briefing.json` all came from
  live Gmail, Google Calendar, and Google Drive lookups, plus a web search
  for the USD/ZMW rate (Bank of Zambia's own site is JS-rendered and
  couldn't be scraped directly — see the `forex.source` field).
- **Client-side, browser-only:** the clap detector (Web Audio API amplitude
  spikes) and the wake-phrase listener (`webkitSpeechRecognition`) run
  entirely in the browser. Voice **output** uses the browser's built-in
  `speechSynthesis` API so the page works offline with no API key. A
  higher-quality ElevenLabs narration of this same briefing was generated
  separately and sent alongside this dashboard — wiring a real TTS API key
  into the page itself was left out on purpose so nothing here silently
  spends the user's API credits.
- **Not implemented:** an actual scheduler that refreshes `briefing.json`
  automatically. Wire that up with a cron job or a scheduled Claude Code
  session that re-runs the same four lookups and overwrites the file.
