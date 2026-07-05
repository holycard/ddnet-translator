# DDNet Chat Translator

A [DDNet](https://ddnet.org/) client modification — a built-in chat translator powered by the DeepL API. Translates incoming messages directly in-game and lets you reply in any language you choose, without switching to a third-party translator.

> Works **client-side only** — this is not a server mod. The feature is only active for whoever builds/downloads this client, and works on any DDNet server regardless of whether other players have this modification installed.

---

## Screenshots

<!-- Screenshot 1: an example of a translated message right in the in-game chat -->
![Chat translation](screenshots/chat-translation.png)

<!-- Screenshot 2: the full Translator settings tab (key, languages, checkboxes) -->
![Translator settings tab](screenshots/settings-tab.png)

<!-- Screenshot 3: the Players tab with the Translate column -->
![Translate column in the player list](screenshots/players-tab.png)

---

## Features

- Automatic translation of incoming **whispers (private messages)**.
- Automatic translation of messages that **mention your name**.
- A **"translate all chat"** mode — opt-in, one checkbox.
- Marking a **specific player** whose messages are always translated, regardless of other settings.
- Translating and sending **your own reply** — to a specific player or to the whole chat, in any language you pick.
- Smart duplicate suppression: if a message is already in the language you're reading, the translation isn't shown again.
- The language list is fetched directly from DeepL, with a search box — no need to remember language codes.

---

## Requirements

Translation requires a **personal DeepL API key**. Without a key, the translator simply does nothing — the rest of the client works as a regular DDNet build.

### Getting a key

1. Sign up at [deepl.com/pro-api](https://www.deepl.com/pro-api) — there's a free tier (Free), whose keys end in `:fx`.
2. Copy the key from your DeepL account dashboard.
3. Paste it into the client: **Settings → Translator → DeepL API key**.

Current free-tier limits (monthly character quota and terms) may change over time — check your DeepL account dashboard for up-to-date figures.

**Never share or publish your key** — it's tied to your DeepL account and your quota.

---

## Installation / Building

```bash
sudo apt install build-essential cargo cmake git glslang-tools google-mock \
  libavcodec-extra libavdevice-dev libavfilter-dev libavformat-dev libavutil-dev \
  libcurl4-openssl-dev libfreetype6-dev libglew-dev libnotify-dev libogg-dev \
  libopus-dev libopusfile-dev libpng-dev libsdl2-dev libsqlite3-dev libssl-dev \
  libvulkan-dev libwavpack-dev libx264-dev ninja-build python3 rustc spirv-tools

git clone --recursive https://github.com/<your-username>/ddnet
cd ddnet
mkdir build && cd build
cmake ..
make -j$(nproc)
```

Resulting binary: `build/DDNet`.

> If the build fails with an internal compiler error — this is a known bug in newer GCC versions when compiling `<format>`. Build with Clang instead:
> ```bash
> cmake -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ ..
> make -j$(nproc)
> ```

---

## Setup after installation

Open **Settings → Translator** in the client menu:

<!-- You can duplicate the settings tab screenshot here with labeled fields -->
![Translator tab setup](screenshots/settings-tab-annotated.png)

1. **Enable chat translation** — turns translation on.
2. **DeepL API key** — paste your key (see "Requirements" above).
3. **Reading language** — the language incoming messages are translated into. The list is fetched from DeepL and includes search.
4. **Reply language** — the language your own text is translated into when using the `/tw` and `/twa` commands.
5. **Translate all chat messages** — enable this if you want the entire chat translated, not just whispers and mentions of your name.

---

## Commands

Work **both in the in-game chat and in the console** (`~`) — you can type them as a regular console command, or directly into the chat box with a leading `/`.

| Command | Where it works | What it does |
|---|---|---|
| `/tr text` | chat and console | Translates `text` into your **Reading language** and shows the result only to you — nothing is sent to anyone. Handy for quickly reading a foreign phrase or checking how something sounds in another language. |
| `/tw Player text` | chat and console | Translates `text` into your **Reply language** and sends it to the given player as a whisper (like `/w`). |
| `/twa text` | chat and console | Translates `text` into your **Reply language** and sends it to the whole chat, for everyone. |

All three commands are saved in your chat history — you can recall them again with the **up arrow**, just like regular messages.

**Example:**
```
/tw Player123 Hey, how's it going?
```
`Player123` will receive a whisper with your phrase translated into whatever language is set as **Reply language**.

---

## Marking individual players

The **Players** tab (the server player list — where the mute and friend buttons also live) has a **Translate** column.

<!-- Close-up screenshot: a player row with the Translate toggle enabled -->
![Translate toggle on a player row](screenshots/player-translate-toggle.png)

Turning the toggle on for a specific player means you'll get a translation of **all** of their messages in regular chat — even if the general "translate all chat" mode is off, and even if they don't directly mention your name.

> This flag only lasts for the current session and resets on reconnect — unlike the friends list, it is not stored persistently.

---

## How it works under the hood (brief)

1. The server sends the client a chat message (whisper or regular).
2. The client checks the translation conditions: is it a whisper? does it mention your name? is "translate all chat" enabled? is the sender marked with the Translate toggle?
3. If a condition is met, the text is sent as an asynchronous HTTP request to the DeepL API, without blocking the game.
4. When the response arrives, the source language detected by DeepL is compared to your **Reading language** — if they match, the translation isn't shown (to avoid duplicating identical text).
5. Otherwise, the translation is printed as a separate line in chat, alongside the original.
6. The `/tr`, `/tw`, and `/twa` commands work the same way, but with text you type yourself, and using **Reply language** instead of Reading language.

---

## Known limitations

- Requires your own DeepL API key — the feature doesn't work at all without one.
- DeepL Free's limits (character quota) apply per account — heavy use (e.g. translating the entire chat on a busy server) will burn through your quota faster.
- Translating your own messages only works through the explicit `/tw` / `/twa` commands, not automatically for regular messages sent with Enter.
- The "always translate this player" flag doesn't persist across server rejoins.

---

## Disclaimer

This is an unofficial DDNet client modification, not affiliated with the DDNet team or DeepL SE. It uses the DeepL API under their terms of service — review [DeepL's API terms](https://www.deepl.com/pro-api) before use.
