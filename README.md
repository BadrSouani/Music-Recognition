# Music Recognition

A real-time music recognition system implemented in C++, inspired by the Shazam algorithm. It records audio from a microphone, extracts spectral fingerprints using FFT, and identifies the matching song from a pre-built database.

## How It Works

The recognition pipeline follows the same core idea as Shazam's constellation-map algorithm:

1. **Preprocessing** — incoming audio is converted to mono, low-pass filtered at 5000 Hz, and downsampled from 44.1 kHz to 11.025 kHz to reduce computation.
2. **Feature Extraction** — the audio is split into 1024-sample windows (≈92 ms each). A Hamming window is applied to each chunk before a 512-bin FFT is computed. The dominant frequency in each of 6 logarithmic bands is identified as a *peak*.
3. **Hashing** — peaks above the mean amplitude are encoded into a single integer hash using a base-1000 scheme across the band indices, producing a compact fingerprint per frame.
4. **Matching** — hashes from the recorded audio are looked up in a pre-built hash map. The song accumulating the most hash matches is returned as the result.

### Frequency Bands

| Band | Range |
|------|-------|
| 0 | 0 – 10 Hz |
| 1 | 10 – 20 Hz |
| 2 | 20 – 40 Hz |
| 3 | 40 – 80 Hz |
| 4 | 80 – 160 Hz |
| 5 | 160 – 511 Hz |

## Features

- Real-time audio capture and spectrogram visualization (900×900 window)
- FFT-based spectral fingerprinting with Hamming windowing
- Persistent binary database (`songs.dat` / `hashes.dat`) — auto-generated on first run
- Built-in DSP filter library (Butterworth, Chebyshev I/II, Elliptic, Bessel, Legendre, RBJ, custom)
- Millisecond-level performance timing for both database generation and recognition
- 10 sample songs included for testing

## Project Structure

```
Music-Recognition/
├── src/
│   ├── Application.cpp         # Entry point, window, key bindings
│   ├── AudioRecorder.*         # Audio capture, LPF, downsampling
│   ├── DebugAudioRecorder.*    # Extended recorder with debug logging
│   ├── FFT.*                   # Cooley-Tukey FFT + spectrogram rendering
│   ├── ModelFFT.*              # Fingerprinting and matching logic
│   ├── Database.*              # Hash map storage and serialization
│   ├── DataPoint.*             # (songId, timeOffset) pair
│   ├── Dsp.h                   # DSP filter library umbrella header
│   └── [Filter sources]        # Bessel, Butterworth, Chebyshev, Elliptic, ...
├── Database/
│   ├── Songs/                  # .wav files used to build the database
│   ├── songs.dat               # Serialized song names (auto-generated)
│   └── hashes.dat              # Serialized fingerprints (auto-generated)
└── CMakeLists.txt
```

## Dependencies

- **[SFML](https://www.sfml-dev.org/)** — audio recording/playback and 2D graphics
- **C++14** standard library (`<complex>`, `<valarray>`, `<filesystem>`, `<fstream>`)
- **CMake 3.13+**

## Building

```bash
mkdir build
cd build
cmake ..
cmake --build .
```

On Windows you can also open the project in Visual Studio via CMake integration.

Make sure SFML is installed and discoverable by CMake (e.g., via `SFML_DIR` or system packages).

## Running

```bash
./Music_Recognition        # Linux / macOS
Music_Recognition.exe      # Windows
```

### Controls

| Key | Action |
|-----|--------|
| `SPACE` | Start / stop microphone recording |
| `ENTER` | Match the recorded audio against the database |
| Close window | Exit |

On the first run the application scans `Database/Songs/` for `.wav` files, builds the fingerprint database, and caches it to `songs.dat` and `hashes.dat`. Subsequent runs load the cache directly.

## Adding Songs

Drop any `.wav` file into `Database/Songs/`, then delete `songs.dat` and `hashes.dat` so the database is regenerated on the next launch.

## Sample Songs Included

| File |
|------|
| Camellia - Nacreous Snowmelt.wav |
| Eminem - Ass Like That.wav |
| Halogen - U Got That.wav |
| Herbie Hancock - Chameleon.wav |
| My First Story - Winner.wav |
| Nanahira - Petals.wav |
| One Ok Rock - Mighty Long Fall.wav |
| Snarky Puppy - Flight.wav |
| Xi - ascension_to_heaven.wav |
| sifflet.wav |

## Technical Notes

- Sample rate used internally: **11 025 Hz** (4× telephone quality — sufficient for melody recognition and faster to process)
- FFT window size: **1024 samples** → fingerprints generated every **~100 ms** (10 frames/second)
- The DSP filter library bundled in `src/` is based on Vincent Falco's [DSPFilters](https://github.com/vinniefalco/DSPFilters)
- Code comments and some variable names are in French

## License

No license file is included in this repository.
