# agenteacher
Why not upgrade in schooling systems , rather than traditional method make the classroom more interactive and a 3d one, this model assists teacher it recognizes voice commands and acts as a classroom jarvis 
An agent that watches your slide, listens to you teach, and generates a live diagram to
help explain the concept you're currently talking about.

## What it does
- Press **F9** anytime → it screenshots your current slide, sends it (plus a bit of context)
  to Claude, and shows a generated diagram in a small always-on-top window.
- If voice mode is on, saying things like **"visualize this"** or **"visualize the whole page"**
  while you teach triggers the same thing automatically - Claude also gets the last ~45 seconds
  of what you were saying, so the diagram reflects the concept you were mid-explanation on.

This is a **local script that runs on your own computer** - nothing is uploaded except the
slide screenshot sent to Claude at the moment you trigger it.

---

## 1. Install Python
If you don't have it: https://www.python.org/downloads/ (get 3.10 or newer). On Windows,
tick "Add Python to PATH" during install.

## 2. Get the files onto your computer
Download this whole `teaching_visualizer` folder and open a terminal inside it.

## 3. Install the dependencies
```bash
pip install -r requirements.txt
```
A couple of notes depending on your OS:
- **Windows**: everything above installs cleanly via pip, no extra system packages needed.
- **Mac**: you may need `brew install portaudio` before `pip install sounddevice` works.
- **Linux**: you may need `sudo apt install portaudio19-dev` first, and the `keyboard` library
  needs to be run with `sudo` for global hotkeys to work.

## 4. Get a Claude API key
Sign up / log in at https://console.anthropic.com, create an API key, then set it as an
environment variable (don't paste it directly into the code):

**Windows (PowerShell):**
```powershell
setx ANTHROPIC_API_KEY "your-key-here"
```
Close and reopen your terminal after this.

**Mac/Linux:**
```bash
export ANTHROPIC_API_KEY="your-key-here"
```
Add that line to your `~/.zshrc` or `~/.bashrc` to make it permanent.

## 5. Run it
```bash
python main.py
```
You should see a small window appear (top-right by default) and a console message saying
it's ready. Open your PowerPoint, start talking, and either press **F9** or say a trigger
phrase.

## 6. Adjust settings
Open `config.py`:
- `ENABLE_VOICE = False` if you just want the F9 hotkey and don't want continuous mic listening.
- `WHISPER_MODEL_SIZE` - use `"tiny"` if your laptop is slow, `"small"` or `"medium"` if you
  want more accurate transcription and have a decent CPU/GPU.
- `TRIGGER_PHRASES` - add/remove phrases to whatever feels natural for you to say.
- `HOTKEY` - change from F9 to any other key if it conflicts with something.

---

## How it works (architecture)

```
Mic ──► faster-whisper (background thread) ──► rolling transcript buffer
                                                        │
F9 key ──────────────────┐          trigger phrase heard│
                          ▼                              ▼
                    run_visualization() ◄─────────────────
                          │
              screenshot (mss) + cursor position (pyautogui)
                          │
                          ▼
              Claude API (vision) - sees slide + transcript + cursor
                          │
                          ▼
                 Claude returns raw SVG code
                          │
                          ▼
        pywebview overlay window renders the SVG live
```

## Known limitations (things to improve later)
- The voice listener transcribes in fixed 4-second chunks, so there's a small delay (a few
  seconds) before a trigger phrase registers - it's not instant streaming.
- Cursor position is a rough proxy for "what you're pointing at" - it works well if you're
  moving your mouse while explaining, less so if you're not touching the mouse at all.
- Each F9 press / voice trigger = one Claude API call, so there's real (small) per-use cost.
- If you want a punchier, truly real-time voice pipeline later, look at these repos for ideas:
  - https://github.com/collabora/WhisperLive
  - https://github.com/Jemtaly/Whispering
