# Nova — architecture showcase

[View the live showcase →](https://abhijeet-rane.github.io/nova-showcase/)

A voice-controlled personal AI assistant for Windows 11. Hybrid local + cloud inference, custom-trained wake word, layered security with a zero-lockout guarantee.

**The implementation repository is private.** This repo hosts the architecture documentation so the design can be reviewed without exposing source.

## What you'll find here

| File | Purpose |
|---|---|
| `index.html` | Landing page &mdash; project overview, four architecture views, tech stack. Rendered via GitHub Pages. |
| `diagrams/` | The four architecture views, each in SVG (web-quality), PNG (200 DPI for print/slides), and PDF (vector, print-ready). |
| `nova-design-doc.pdf` | The full 40-page design document &mdash; goals, non-goals, feature catalogue, resource budget, threat model, roadmap. |

## Four architecture views

1. **System Context** &mdash; how Nova sits in the user&rsquo;s environment
2. **Container View** &mdash; processes and storage that make up the runtime
3. **ML Pipeline** &mdash; custom wake-word training (offline) + streaming inference (runtime)
4. **Security &amp; Authorization** &mdash; tiers, factors, recovery paths

## Highlights (for the AI/ML-minded reviewer)

- **Custom wake-word classifier** trained from scratch in PyTorch on 13k Piper-TTS-synthesised samples. 4-layer CNN with SpecAugment + class-weighted BCE, exported to ONNX (~30 KB) for production.
- **Hybrid inference architecture** &mdash; local Whisper (int8) + rule-based intent classifier for ~80% of commands; Claude Sonnet escalation behind a tier gate + user-controlled spend cap for the rest.
- **Streaming real-time audio pipeline** &mdash; VAD &rarr; wake classifier &rarr; lazy-loaded STT &rarr; intent &rarr; policy &rarr; execute. Runs under a 3% idle CPU budget.
- **Feature-parity training/serving boundary** &mdash; training uses the same `AudioFeatures.embed_clips` feature extractor that the runtime wake classifier consumes, eliminating train/serve skew.
- **Four-tier authorization** with three independent auth factors (voice, PIN via Argon2id, Windows Hello) and six independent recovery paths &mdash; guarantees the assistant cannot lock the user out.

## Stack

Python 3.11+ &middot; PyTorch &middot; ONNX Runtime &middot; faster-whisper &middot; openwakeword &middot; Piper TTS &middot; Anthropic Claude API &middot; PySide6 &middot; pywin32 &middot; winsdk &middot; pycaw &middot; argon2-cffi &middot; cryptography &middot; SQLite &middot; Pydantic v2 &middot; uv &middot; mypy strict &middot; pytest (500+ tests, 83% coverage).

## Regenerating the diagrams

Diagrams are authored in [D2](https://d2lang.com/) text source. To regenerate from source:

```bash
# Install D2 and Inkscape (for PDF/PNG export)
scoop install d2 extras/inkscape

# Render SVG
for f in 01-context 02-containers 03-ml-pipeline 04-security; do
  d2 --theme 0 --layout elk --pad 40 "$f.d2" "$f.svg"
  inkscape "$f.svg" --export-type=png --export-dpi=200 --export-filename="$f.png"
  inkscape "$f.svg" --export-type=pdf --export-filename="$f.pdf"
done
```

The `.d2` sources live in the private implementation repo under `docs/architecture/`.
