# Sticker Pack Maker Bot

A Telegram bot that takes a folder of base animated stickers (`.tgs` / Lottie JSON),
injects a logo, text, or SVG into each one, lets you preview and adjust the size
before committing, then bulk-creates a full custom emoji/sticker pack.

## Features

- **Three logo modes**: inject a Lottie `.json` logo, an `.svg`, or auto-generated text
- **Smart color swapping**: detects existing brand colors in each base animation and
  recolors both the base animation and the injected logo to match, with sensible
  defaults you can skip
- **Live GIF preview**: renders a real animated GIF preview before you commit, with
  no sticker-format constraints (size, fps, file size) getting in the way while you
  check how it looks
- **Paged preview across all selected stickers**: browse ◀️ ▶️ through every base
  animation you selected before committing, so you can catch a logo that fits
  well on one sticker but not another
- **Adjustable scale**: nudge the logo size up/down with buttons or type a value,
  re-preview instantly
- **Two output types**: build either a Custom Emoji pack or a regular Sticker pack
  from the same selection
- **Bulk pack creation**: processes every selected base animation and builds one
  Telegram sticker set in one go
- **Single-user locked**: only responds to the Telegram user ID set in `ALLOWED_USER`

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

If `cairosvg` fails to import at runtime with a `cairo` library error, install the
system Cairo library too:

```bash
# Debian/Ubuntu
sudo apt-get install libcairo2
```

### 2. Configure your bot token

Copy the example env file and fill in your real token (get one from
[@BotFather](https://t.me/BotFather)):

```bash
cp .env.example .env
```

```
BOT_TOKEN=123456789:your_real_token_here
```

**Never commit `.env` to git.** If a token has ever been pushed to a public repo,
treat it as compromised — revoke it in @BotFather and generate a new one immediately.

### 3. Set your Telegram user ID

Open `main.py` and set `ALLOWED_USER` to your own numeric Telegram user ID (you can
get this from a bot like [@userinfobot](https://t.me/userinfobot)). Only this user
can operate the bot.

### 4. Add your base animations

Drop your base `.json` Lottie files into `lotties/`, named numerically:

```
lotties/001.json
lotties/002.json
...
lotties/103.json
```

The bot automatically detects how many files are in this folder — you don't need to
edit any code when you add or remove stickers. The "FULL" button and the manual
index validator both read the live count.

### 5. Run it

```bash
python3 main.py
```

Runs via long polling — no webhook/HTTPS setup needed. Recommended to run under
`systemd` on a VPS so it restarts on crash/reboot.

## Usage flow

1. `/start` in your bot chat
2. Choose logo mode: **JSON**, **Text**, or **SVG**
3. Choose which base animations to modify — tap **FULL** for all of them, or type
   specific numbers (e.g. `1,5,12` or `1.5.12`)
4. Enter/skip the two brand color overrides
5. Send your logo file / text / SVG
6. Enter/skip the two logo colors (used depending on whether the base animation has
   the special brand colors or not)
7. **Preview**: you'll get an animated GIF of the first selected sticker.
   - ◀️ / ▶️ page through every sticker you selected, so you can check the logo
     fits correctly across all of them (not just the first one — different base
     animations can have the logo positioned differently)
   - +10/+5/-5/-10 buttons or typing a value (e.g. `+8`, `-3`, `75`) resizes the
     logo across the whole pack and re-renders the current page
8. Tap **✅ Done** (or type `DONE`) when it looks right
9. Choose the output type: **🙂 Custom Emoji Pack** or **🎨 Sticker Pack** — the bot
   then builds the full pack across every selected base animation and gives you the
   `t.me/addemoji/...` or `t.me/addstickers/...` link when finished

## Project structure

```
main.py            — bot logic (single file)
requirements.txt    — Python dependencies
.env.example         — template for your bot token
lotties/             — your base animation files (001.json, 002.json, ...)
Anton-Regular.ttf     — font used for auto-generated text logos
```

## Notes on how it works

- **Preview GIFs** are rendered with the `lottie` library's GIF exporter
  (`cairosvg` + `Pillow` under the hood) directly from the in-memory animation dict —
  they never touch Telegram's sticker upload pipeline, so there's no size/format
  limit to worry about while iterating.
- **Final pack stickers** are exported as gzip-compressed `.tgs` (the actual Telegram
  animated sticker format) and uploaded via `create_new_sticker_set` /
  `add_sticker_to_set`.
- Color detection (`color_exists` / `replace_color_smart`) compares RGB values with a
  small tolerance, so near-matching brand colors are still caught.
