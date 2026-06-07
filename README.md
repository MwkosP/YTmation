# YT Pipeline

Python framework for helping in video production :)

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

## Planned

- **Stitching refactor**: `buildTimeline()` (hash-based) → `renderTimeline()` (ffmpeg) → `exportOTIO()` (DaVinci Resolve)
- **GITRAIL**: pip library for large artifact versioning (git + DVC), agent-friendly API: `snapshot`, `rollback`, `branch`, `getBest`
