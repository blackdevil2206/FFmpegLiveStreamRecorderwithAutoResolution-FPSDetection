A Python script that automatically records live streams using FFmpeg and FFprobe. It detects resolution, framerate, and ensures smooth recording with error handling, timestamped filenames, and text watermarking.

Key Features:
✅ Auto-detects resolution & FPS (via FFprobe)
✅ Checks stream availability before recording
✅ Applies deinterlacing & FPS normalization
✅ Adds watermark text (customizable)
✅ Generates timestamped output filenames
✅ Handles audio sync & corrupt frames

Explanation of Key Components:
FFprobe Integration
Extracts width, height, and fps from the stream.
Falls back to defaults (1080p50) if detection fails.
Stream Health Check
Uses ffmpeg -t 5 to verify if the stream is online before recording.
Smart Recording
Forces 50 FPS if the source FPS is lower (for smooth playback).
Adds a watermark ("BlackDevil") at the bottom-right corner.
Uses libx264 (CRF 21) for balanced quality/compression.
Error Resilience
Handles corrupt frames, audio sync (aresample=async=1), and FFmpeg crashes.

Suggested Dependencies:
FFmpeg 7.1+ (static build recommended)

Python 3.6+
