# YT Automation Pipeline

Modular Python system for automated YouTube video production.

---

## Structure

```
├── main.py                  # Orchestrator
├── Footage/footage.py
├── Voice/voice.py           # ElevenLabs TTS
├── Script/script.py         # Claude API
├── Stitching/stitching.py   # All ffmpeg + overlay functions
├── SFX/sfx.py
├── Music/music.py           # yt-dlp
├── Thumbnail/thumbnail.py
├── Thumbnail/templates/     # bold_text.py, gradient.py, radial_glow.py
├── Templates/               # reddit_card.py, lower_third, title_card, end_card
├── Games/games.py           # pygame animations
├── Games/animations/        # each has run() with all params exposed
├── Blender/
├── Storage/storage.py       # JSON db
├── Upload/                  # not yet built
├── Math/                    # Manim pipeline
└── Cache/                   # temp files
```

---

## Conventions

- All public functions: `camelCase`
- `utils.py` is internal only — never called from `main.py`
- No hardcoded prompts in `.py` files
- No `if __name__ == "__main__"` blocks
- Every workflow: `try/except` + `logRun()` at end
- All functions accept a `channel=` param

---

## Module APIs

| Module | Key Functions |
|---|---|
| `Stitching` | `overlayImage`, `overlayAudio`, `overlaySFX`, `overlayMusic`, `overlayMultipleSFX` |
| `SFX` | `getSFX`, `listSFX`, `downloadSFX` |
| `Music` | `downloadMusic`, `getMusic`, `listMusic`, `prepareMusic` |
| `Games` | `renderAnimation`, `previewAnimation`, `listAnimations`, `getAnimation`, `randomAnimation` |
| `Thumbnail` | `generateThumbnail`, `previewThumbnail`, `listTemplates`, `getThumbnail` |
| `Storage` | `logUpload`, `cacheScript`, `getCachedScript`, `logRun`, `getLogs`, `getFailedRuns` |
| `Math` | `math.py` → `mathAbstractions.py` → `Math/library/` (composable, never rendered standalone) |

---

## Storage

JSON files in `Storage/db/`: `uploads.json`, `cache.json`, `history.json`, `logs.json`

---

## ffmpeg Rules

- Never use `format=yuva420p` (breaks alpha)
- Overlay pattern: `[0:v][1:v]overlay=x:y:enable='between(t,start,end)'[outv]`
- SFX delay: `adelay={ms}|{ms},volume=x`
- Music: `stream_loop -1` + `afade` out
- GPU (when available): `-c:v h264_nvenc -preset p4`

---

## Examples

```python
from Script.script import generateScript
from Voice.voice import generateVoice
from Stitching.stitching import overlayAudio, overlayMusic
from Music.music import prepareMusic
from Thumbnail.thumbnail import generateThumbnail
from Storage.storage import logRun

def run(channel="math"):
    try:
        script = generateScript("Top 5 math facts", channel=channel)
        voice  = generateVoice(script, channel=channel)
        music  = prepareMusic(vibe="ambient", channel=channel)

        video  = overlayAudio("math.mp4", voice, start=0, channel=channel)
        video  = overlayMusic(video, music, channel=channel)

        generateThumbnail("Top 5 Math Facts", channel=channel)
        logRun(success=True, channel=channel)
    except Exception as e:
        logRun(success=False, error=str(e), channel=channel)
```

> Input video (`math.mp4`) lives at the project root, same level as `main.py`.

---

## Planned

- **Stitching refactor**: `buildTimeline()` (hash-based) → `renderTimeline()` (ffmpeg) → `exportOTIO()` (DaVinci Resolve)
- **GITRAIL**: pip library for large artifact versioning (git + DVC), agent-friendly API: `snapshot`, `rollback`, `branch`, `getBest`
