# SIGNAL SCOPE

A local WAV inspector with a waveform, spectrogram, silence detection and
sample-aligned fragment export. Built with Python, PySide6 and NumPy.

Open a recording, inspect its frequency content, select a section and save the
original PCM frames. Everything runs on your machine. There are no accounts,
network requests, model downloads or external decoding tools.

![SIGNAL SCOPE waveform and spectrogram](screenshot.png)

## Features

- Waveform overview that retains the minimum and maximum samples in each bin.
- Hann-windowed spectrogram with adjustable FFT size and display range.
- Separate channel inspection and an optional average mix for multichannel files.
- Linked time scales, drag selection, numeric selection boundaries, zoom and pan.
- Peak and RMS amplitude for the visible range.
- Full-scale sample counts and silence intervals.
- Selection playback through Qt Multimedia, when a working audio backend is available.
- WAV export that retains the original channels, sample rate and integer bit depth.
- Background loading, analysis and export.

## Requirements

- Python 3.10 or newer; a 64-bit installation is recommended.
- Windows or Linux with a working desktop environment.
- PySide6 and NumPy, listed in `requirements.txt`.

Supported input: uncompressed, little-endian integer PCM WAV, 8/16/24/32-bit,
1–32 channels, up to 768 kHz. Eight-bit PCM is unsigned; the other depths are signed.

IEEE float WAV, compressed WAV, RF64, MP3 and FLAC are not supported. Extensible
PCM WAV support depends on the Python standard library version; Python 3.12 or
newer is recommended for those files. Convert unsupported audio to integer PCM
before opening it. Export creates a plain PCM WAV and does not copy extra metadata
chunks such as cue points, BWF annotations or embedded tags.

## Install

Download and extract the project, then open a terminal inside `SIGNAL_SCOPE`.

### Windows

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe main.py
```

These commands use the environment's interpreter directly, so PowerShell script
activation is not required.

### Linux / WSL

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python main.py
```

WSL needs GUI support, such as WSLg. Windows files are accessible through `/mnt/c`;
for example, `/mnt/c/Users/<username>/Downloads`.

On Ubuntu, if the virtual environment or Qt platform libraries are missing:

```sh
sudo apt update
sudo apt install python3-venv libegl1 libopengl0 libgl1 libxkbcommon0 libxcb-cursor0 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 libxcb-render-util0 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0
```

Installation needs access to the packages or a local wheel cache. The installed
application works offline. It never installs packages itself.

## Workflow

1. Click **OPEN WAV**. The sidebar shows the duration, sample rate, channel count,
   PCM depth and frame count.
2. Choose the channel to inspect. **Mix (average)** averages the channels; opposite
   phase signals can cancel in this view. Channel choice affects analysis plots
   and peak/RMS metrics, not export or playback.
3. Drag across either plot to select a time interval. Both plots show the same
   selection. You can also enter start and end times at the bottom.
4. Click **ZOOM SELECTION** to analyze that interval more closely. **+** and **−**
   zoom around the center of the current view. Use the horizontal scrollbar to pan;
   **FULL FILE** restores the full recording.
5. Adjust **FFT** to trade frequency resolution for time resolution. The display
   range controls contrast without recalculating the FFT. Hover over the spectrum
   to inspect time, frequency and the nearest displayed FFT value.
6. Click a silence interval in the sidebar to select it.
7. Use **PLAY SELECTION** / **STOP** to listen, or **EXPORT WAV** to save the selection.

**SELECT ALL** selects the complete file, even while zoomed in. A zero-length
selection cannot be exported. Selection boundaries snap to source frames; end
boundaries are exclusive. Export keeps every source channel, even when the plot
shows a single channel or the average mix.

## Reading the analysis

Time is measured from the beginning of the file. Frequency uses a linear Hz axis
from zero to half the sample rate. Spectrogram colors show Hann-windowed peak
amplitude in dBFS, not a power spectral density. A bin-centered full-scale sine
approaches 0 dBFS away from selection boundaries. Zero-padding at the boundaries
can reduce the displayed amplitude there. Values below the selected display floor
share the darkest color.

The metrics and silence list describe the **visible range**:

- **Peak:** maximum absolute amplitude in the chosen analysis channel.
- **RMS:** root mean square amplitude in that channel.
- **At rail:** count of exact minimum/maximum integer samples across all channels.
  This is a full-scale sample indicator, not proof that the recording was clipped.
- **Silence:** all-channel RMS at or below −50 dBFS in 20 ms blocks, with runs of
  at least 0.25 seconds. Boundaries have roughly 20 ms resolution. Detected runs
  may be cut off by the current view; they do not describe the rest of the file.

Digital silence is displayed as −240 dBFS in numeric metrics. Silence detection
uses all source channels, so phase cancellation in the average mix cannot create
false silent regions.

### Long recordings

The application streams PCM data in chunks instead of decoding the whole file
into memory. The waveform and numeric metrics cover every sample in the view.
The spectrogram is limited to 1,200 analysis windows. Long views therefore sample
windows across time and may miss short events between them; the status line marks
these views as **TIME-SAMPLED STFT**. Zoom in to inspect a short event more closely.

Loading and full-file metrics still read the audio from disk and can take time.
The interface remains usable, but closing during a job waits for it to finish.
There is no background cancellation or persistent analysis cache.

## Export and playback

Export copies the selected raw PCM frames to a new WAV without normalization,
resampling or lossy encoding. The original file and aliases of it are protected.
Existing output files require confirmation. Output is written to a temporary file
and replaces the destination only after the write succeeds.

Playback uses the original recording, including all channels. It depends on the
system's Qt audio backend and codec support. Selection playback uses millisecond
positions and a timer; its stopping point is approximate. Export boundaries are
sample-accurate. A playback failure does not prevent analysis or export.

Keep the source file unchanged while it is open. If you replace or edit it in
another program, reopen it before continuing.

## Tests

```sh
python -m unittest -v
```

Tests cover PCM decoding at all four bit depths, exact fragment export, known-tone
FFT frequency and amplitude, stereo phase cancellation, silence detection, invalid
and truncated files, source protection, short inputs and the GUI workflow. GUI
tests run with Qt's offscreen platform and do not require an audio output device.
Audible playback and native Windows behavior require checking on the target system.

## Files

| File | Purpose |
| --- | --- |
| `main.py` | Window, plots, background tasks and optional playback |
| `audio.py` | PCM decoding, waveform, FFT, silence detection and export |
| `test_scope.py` | Audio and GUI checks using `unittest` |
| `requirements.txt` | Runtime dependencies |
