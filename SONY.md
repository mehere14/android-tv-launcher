# What this project changes

This project is based on [Free My TV](https://github.com/simbasjungles/free-my-tv) (MIT). It sets Free My TV up for a Sony
BRAVIA running Android TV 10, with a new home screen design, Google features (Ask AI, weather, radar, routes) and
NASA's picture of the day. This page records everything this project changes from the original, in the order it
was built. For installing, settings and undoing, see the [README](README.md).

Everything here runs on the Android app (`app/`). The LG and Samsung web app (`web/`) only lost the crypto prices; it
doesn't have the new features yet (see [Not done yet](#not-done-yet)).

To set up the keys (Gemini, Google Maps) and send them to the TV, see
[the README](README.md#set-up-ask-ai-the-weather-and-the-maps).

## 1. Set up for the Sony BRAVIA

- **Turned off on the switch** (`HomeMode.DEBLOAT`): Sony's promotions, Sony Select, the store demo, usage reporting
  and the stock home screen's customizer. Never uninstalled; *Original* turns them back on.
- **Never touched** (`HomeMode.PROTECTED`): Sony's picture, sound, inputs, live TV, BRAVIA Sync, remote, setup and
  update apps, plus YouTube, Ambient mode and voice search.
- **The TV keeps its own screensaver.** New setting `own_screensaver` (default `false`): the switch doesn't change the
  screensaver or turn off its app, and *Original* only resets a screensaver it set itself.
- **Apps that were already off stay off.** Apps the TV had turned off before the first switch are no longer recorded as
  Free My TV's, so *Original* doesn't turn them on.
- **The shelf:** Netflix, Prime Video, StbEmu, Spotify and Apple TV, five at most.
- **Removed:** crypto prices (Android app, LG and Samsung app, build, settings, website and docs), and the Russian and
  German translations. Languages are now English, Spanish and Portuguese.
- **`sony-before.txt`** records the TV's home screen, screensaver and already-off apps before the first switch, for
  undoing by hand from the computer (see *Undo without the app* in the README).

## 2. A new home screen design

The approved mockup (`mockup/`), redrawn on the TV by `HomeView`:

- A greeting and the clock, with **Home settings**, **TV settings** and **Profile** (a letter: `profile_initial`) in
  the top right.
- A large **search row**: the search box with a mic inside it, and **Ask AI**.
- Four boxes: the **weather** (now, hour by hour, sunrise, sunset, chance of rain), a **radar** thumbnail, the
  **time and date**, and **Plan a route**.
- **No trending picks.** The original's recommendation cards are gone from the home screen; the TMDB key now only
  feeds the optional screensaver (`own_screensaver=true`).
- Every box has the shelf's rounded corners (62 px) and softer shadows. The sizes come from `mockup/tokens.json`.
- Nothing is allocated while drawing: text is built once a minute or when the data changes. On the Sony, 99% of frames
  take 8 to 9 ms while the focus moves.

## 3. Keys sent from the computer

The Gemini and Google Maps keys don't have to be typed on the remote. Push `config.properties` to the TV
(`/data/local/tmp`, readable only by the debugging shell) and press Home: Free My TV imports `gemini_key`,
`gemini_model`, `gemini_grounding` and `maps_key`, encrypts them with AES-GCM under an Android Keystore key
(`Keys.java`), and deletes the file. Keys never go to the log. The commands are in the README.

## 4. Google features (each one optional)

Without keys the home screen still works: the weather comes from Open-Meteo and the maps from Esri.

- **Ask AI** (`AskActivity`, `Gemini.java`): Gemini answers in large type, with its sources when *Google Search
  grounding* is on. Typed questions go here too, because Google TV's search ignores text handed to it on this TV.
- **Settings → Ask AI**: the key (only its last 4 characters shown), the model (`gemini-flash-latest`), grounding on
  or off, and *Test connection*.
- **Weather**: Google Weather with a Google Maps Platform key (hourly), or Open-Meteo without one or if it fails.
- **Maps** (`MapTiles.java`): Google's map, or Esri's grey map without a key.
- **Radar** (`Radar.java`, `RadarActivity`): OK on the weather or radar card shows the last hour of radar, animated
  (Environment Canada in Canada, NOAA in the US). Ready in about 1.4 s on the Sony.
- **Plan a route** (`PlanActivity`, `Places.java`): say or type a place, pick from Google's matches near home, and
  see the drive with live traffic on the map.
- Background work runs at low priority so the home screen stays smooth.

## 5. Voice

This BRAVIA has no built-in microphone: voice comes from the remote over Bluetooth, while its mic button is held.
Google's voice overlay listened to the missing built-in mic, heard nothing, and covered Ask AI's search box.

- **Hold the remote's mic button** to talk (`Voice.java`); letting go stops listening. The screens show their own
  "Listening…".
- **The on-screen mic, Ask AI and Speak** start listening with one press and stop when you stop talking. The search
  row's mic goes to Ask AI, since Google's voice search doesn't start listening on this TV.
- **No remote paired?** The screens say so, and where to pair it, before asking for the microphone.

## 6. Ask AI as a chat

- **Follow-ups keep the conversation:** a box at the bottom with **Send** and **Speak**, and the whole chat in the
  answer area. Gemini gets the earlier turns with each question.
- **New chat** (by the heading) starts over. **Try again** keeps the chat.
- A new answer appears at the start of its turn, so you read it from the top.

## 7. NASA's picture of the day

- **The background** (`Apod.java`): the newest still image from NASA's Astronomy Picture of the Day feed (no key; days
  with a video are skipped). It's checked hourly and kept on the TV, so it shows straight away after a restart, even
  offline. Over the photo, the home screen uses its night look all day: white text and smoked glass.
- **Under the shelf** (press ↓ from the apps): **‹ 1 / 5 ›** steps through the last five photos, the credit sits in
  the middle, and **ⓘ** opens the photo in full. ↓ again opens the inputs drawer, as before.
- **About this photo** (`ApodActivity`): the whole photo on the left; on the right the title, date, NASA's description
  (↑ ↓ to scroll), the credit and ×. Back closes it.
- **Frosted glass on Android 9 to 12** (`Scene.java`). These versions can't blur live (no AGSL shaders), so the glass
  used to be a flat grey fill. Now a blurred copy of the photo is made once, and each card shows its piece of it, then a
  dark tint and a thin light edge.
- **Ask AI** shows the same photo, blurred, behind the chat, so the two screens look like one.
- To save data, the photo is downloaded at about the screen's size (NASA's originals can be 8 MB or more).

## The mockup

`mockup/` is the design reference: the approved home screen in a browser, with live data. Run it with
`python3 mockup/server.py` and open http://localhost:8765/. It reads the keys from `config.properties`; they never
reach the browser or the log (the page only sees their last 4 characters), and only this computer can connect.

## Testing

- **On the Sony:** `./build.sh`, then `adb install -r FreeMyTV.apk` and `adb logcat -s FreeMyTV`.
- **Without the TV:** an Android 10 emulator. Google has no Android TV 10 image for Apple Silicon Macs, so use the
  plain Android 10 (API 29, arm64) image at 1920×1080. It has the same OS version and the same graphics limits as the
  Sony (no AGSL), which is what matters for the glass. Its shelf is empty, since it has no TV apps.

## Keeping secrets out of the repo

- `config.properties` (your keys), `freemytv.keystore` (the signing key), `out/` (the generated `Config.java` has the
  TMDB key built in) and the built `.apk`, `.ipk` and `.wgt` files are all in `.gitignore`.
- A scan of the files and of every commit found no API keys, tokens or private keys.
- Commits use the GitHub noreply address.
- Don't upload your own `FreeMyTV.apk` to a release: it has your TMDB key inside. Release builds use `--generic`.

## Not done yet

- **LG and Samsung** (`web/`): the new design, Ask AI, voice, radar, routes and NASA's photo aren't there yet.
  `web/home.js` is meant to follow `HomeView`.
- **On the TV:** the space between the shelf and the photo row, and how readable the small text is over bright photos,
  are still to be checked on the Sony.
- **The mockup's** own APOD background, picker and ⓘ screen aren't committed yet.
