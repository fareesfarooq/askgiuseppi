# Ask Giuseppi

A tiny browser game where you talk, in your own words, to **Giuseppi Mezzoalto**, the shady van-dwelling "family man," and watch his mood, expression, and opinion of you shift in real time based on what you say. The dialogue is generated live by a large language model rather than pulled from a fixed script.

![Retro GBA-styled dialogue screen](sprites/pleased.png)

---

## ⚠️ Disclaimer

This is an **unofficial, non-commercial fan project** inspired by **_The Sims: Bustin' Out_** (Electronic Arts / Maxis). It is **not affiliated with, endorsed by, or sponsored by Electronic Arts or Maxis**. All trademarks, characters, and original concepts referenced here belong to their respective owners. It was built purely for **personal and educational purposes** (to experiment with LLM-driven game dialogue) and is **not intended for distribution or commercial use**. No original game assets, ROMs, or copyrighted code from the retail title are included in this repository.

---

## What it is

Instead of picking from a menu of pre-written lines, you type free-form messages to an NPC. Each reply is produced by an LLM that stays in character, and the game reacts to the *emotional tone* the model reports:

- Giuseppi's **sprite** swaps to match the emotion (happy, angry, shocked, unsure, and so on).
- A **mood meter** rises and falls, driving screen shake, a happy-flash, and floating `+`/`-` popups.
- A **relationship rank** (Stranger, Associate, Ally, Confidant) climbs as you earn his goodwill.
- Push him to zero mood and he storms off for a **cooldown** before he'll talk again.

It's styled to feel like a Game Boy Advance conversation screen: pixel font, tiled scrolling background, chunky speech bubble, and chiptune-ish sound cues.

## AI architecture

The whole "brain" is a single client-side call flow, no backend:

- **LLM-driven NPC dialogue.** The player's text plus a character system-prompt are sent to the [Groq](https://groq.com) chat-completions API (`qwen/qwen3.6-27b`, an OpenAI-compatible endpoint). The system prompt pins Giuseppi's voice, backstory, and rules (short replies, never break character), and is anchored on real lines of his dialogue from the source games.
- **Structured emotion output.** The model is instructed to end every reply with a JSON tag, e.g. `{"emotion":"angry"}`. The client parses that tag off the end of the response with a regex, uses it to pick the sprite and mood delta, and strips it from the text shown in the speech bubble. If the tag is missing or malformed, it safely falls back to `neutral`.
- **Sliding-window conversation memory.** Replies are appended to an in-memory history array that is resent on each turn so Giuseppi remembers the conversation. The window is capped at 12 messages — 6 exchanges — with the oldest user+assistant pair dropped, so requests stay bounded in size and cost.

The relevant code lives in [`index.html`](index.html). See `askGiuseppi()` for the API call and emotion parsing, and `showAIResponse()` for how the emotion maps to mood, sprite, and effects.

## Running it locally

You need a **free Groq API key** (get one at [console.groq.com/keys](https://console.groq.com/keys)).

1. Clone or download this repository.
2. Serve the folder over a local web server (recommended, since some browsers mute audio on `file://` pages):
   ```bash
   # from inside the "Ask Giuseppi" folder
   python -m http.server 8000
   ```
   Then open <http://localhost:8000> in your browser. (Opening `index.html` directly by double-clicking also works, but audio may be muted.)
3. On the start screen, **paste your Groq API key** into the field and click **START**.

### About the API key (please read)

- The key is **never hardcoded, saved to disk, or written to `localStorage`**. It is held in a single in-memory variable for the lifetime of the browser tab and is discarded when you close or reload the page.
- It is sent **only** to Groq's API, exactly as any client-side API call would be.
- Because there is no backend, the key does live in the page's memory while you play, so use a **personal, revocable key**, and never commit a key into this repo. The included [`.gitignore`](.gitignore) already excludes common secret/config files as a safety net.

## Project structure

```
Ask Giuseppi/
├── index.html      # the entire game: markup, styles, and logic
├── sprites/        # background tile, player portrait, emotion frames
├── audio/          # background music + emotion sound cues
├── .gitignore      # excludes secrets/config from commits
└── README.md
```

## Usage

This is a personal, non-commercial fan project with no formal license. It's shared for learning and demonstration, not for redistribution or commercial use. _The Sims: Bustin' Out_, its characters, names, and trademarks remain the property of Electronic Arts / Maxis. See the disclaimer above.
