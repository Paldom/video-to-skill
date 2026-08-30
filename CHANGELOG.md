# Changelog

All notable changes to this repository's skills are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning: [SemVer](https://semver.org) on the plugin manifest
(breaking skill-interface change → major, new skill → minor, fix → patch).

## [Unreleased]

### Added
- `video-notes` — turns a YouTube URL or local video into one markdown knowledge
  note with inline screenshots and rendered timestamp deep links. Orchestrates
  the two skills below; ships a note template, genre routing, ordered evidence
  rules (credential redaction, pixels-over-narration, cite-or-flag) and
  prompt-injection handling for video-sourced text.
- `video-transcribe` — caption ladder (creator captions → original-language
  auto-captions → local Whisper) with a deterministic, model-free quality gate.
  Refuses YouTube auto-translations, which compound ASR and MT error.
- `video-keyframes` — scene-change frame capture with a threshold derived from
  each video's measured scene-score distribution, perceptual-hash dedup with
  pinned frames, OCR, and a `timeline.json` digest plus on-demand `frame-at` /
  `clip` re-sampling.
- `docs/setup-prompt.md` — paste-ready `/goal` running all three end to end.
- Repository scaffolded from the skills template.

### Fixed
- `video-keyframes`: coverage was checked on candidate frames but perceptual
  dedup runs afterwards and can empty a stretch entirely. On a 13.6-minute talk
  this opened a 280 s blind spot under a 120 s limit — and it swallowed the one
  slide carrying the setup commands. Coverage is now re-checked after dedup and
  after budget trimming, and blind stretches are interval-sampled.
- `video-transcribe`: rolling-caption dedup only matched a plain prefix, so it
  did nothing on YouTube's actual scrolling window ("a b" / "b c" / "c d") and
  left transcripts ~3x inflated. Now trims the overlapping suffix/prefix at word
  level and reconstructs the source stream exactly.
- `video-transcribe`: a caption-endpoint HTTP 429 no longer aborts the run.
  YouTube throttles `timedtext` independently of media download (observed:
  captions 429 while audio downloaded fine), so the ladder continues to ASR and
  records `caption_failure` in metadata.

### Notes
- The transcript repetition metric is measured **per 1000-character window**,
  not per transcript. A whole-transcript ratio saturates with duration (0.48 at
  1 min, 0.96 at 60 min on healthy prose), so a global threshold rejects every
  long video. Fail threshold is 0.93, above the 0.86 that a legitimate
  enumerated procedure measures.
- Scene thresholds are measured, not assumed: a full slide transition on a light
  deck scores ~0.03, so the conventional `gt(scene,0.4)` finds nothing there.
- ffmpeg 8 removed `-vsync` (it hard-errors); the scripts probe for `-fps_mode`
  and fall back only on older builds.
