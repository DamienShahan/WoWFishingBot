# Real-Time Audio Detection Bot

An automated audio detection system that listens for specific sounds and triggers keyboard actions. Designed for applications like game automation where audio cues need to be detected and responded to automatically.

## Features

- **Real-time audio monitoring** using WASAPI loopback
- **Dual sound detection** - monitors for two different audio files simultaneously
- **Smart key press logic** - differentiates between manual and automatic action endings
- **Randomized wait times** - makes behavior more natural and unpredictable
- **Cross-correlation detection** - robust audio matching that handles volume variations
- **Detailed logging** - timestamped events for debugging and monitoring

## How It Works

### Flow

1. **Start Cycle**: Program begins listening for audio
2. **Press 'k'**: Automatically presses 'k' to start an action
3. **Listen**: Monitors audio output for 23 seconds
4. **Detect**:
   - **Target sound found** → Press 'k' to end action, wait 3-5 seconds, restart cycle
   - **Out-of-range sound found** → Action ends automatically (no key press), wait 1.5-2.5 seconds, restart cycle
   - **No sound found** → Wait 0.5-1.5 seconds, retry

### Key Press Logic

| Event                 | Key Pressed? | Wait Time         |
| --------------------- | ------------ | ----------------- |
| Cycle starts          | ✓ 'k' once   | -                 |
| Target detected       | ✓ 'k' once   | 3-5s (random)     |
| Out-of-range detected | ✗ No press   | 1.5-2.5s (random) |
| Timeout (no sound)    | ✗ No press   | 0.5-1.5s (random) |

## Requirements

### Dependencies

```
pyaudiowpatch
numpy
scipy
librosa
pyautogui
```

### System Requirements

- **OS**: Windows (WASAPI loopback support)
- **Audio**: Active audio output device
- **Python**: 3.7+
- **Weakaura**: [FishingRangeAlert](https://wago.io/4jGL2hFGC)
- **AddOn**: [Better Fishing](https://www.curseforge.com/wow/addons/better-fishing)

## Installation

### 1. Install Python Packages

```bash
pip install pyaudiowpatch numpy scipy librosa pyautogui
```

### 2. Prepare Audio Files

Create two WAV files in the same directory as the script:

- `target.wav` - The sound you want to detect (e.g., fishing bobber splash)
- `out-of-range.wav` - The sound indicating failure (e.g., "out of range" voice)

**Tip**: Record these directly from your application for best accuracy.

### 3. Find Your Audio Device Index

Run the device listing script:

```python
python list_devices.py
```

Look for devices marked with **⭐ WASAPI LOOPBACK** and note the index number.

## Configuration

Edit the configuration section at the top of `main.py`:

```python
# Configuration
TARGET_FILE = "sounds/target.wav"
OUT_OF_RANGE_FILE = "sounds/out-of-range.wav"
OUTPUT_DEVICE_INDEX = 38
LISTEN_DURATION = 23  # seconds

# Wait times as ranges (min, max) in seconds
WAIT_AFTER_NOT_FOUND = (1, 2)
WAIT_AFTER_TARGET_FOUND = (3, 5)
WAIT_AFTER_OUT_OF_RANGE = (2, 4)

THRESHOLD = 1.2  # Correlation threshold (adjust if needed)
CHUNK_DURATION = 0.3  # Process audio every 0.3 seconds for faster response

# Hotkey configuration
ACTION_KEY = 'k'  # Key to press which starts/ends the action
```

## Usage

### Basic Usage

```bash
.\.venv\Scripts\Activate.ps1
python fishing.py
```

### Stop the Program

Press `Ctrl+C` to safely stop the program.

### Typical Output

```
============================================================
REAL-TIME DUAL AUDIO DETECTION PROGRAM
============================================================
[01:44:14] Device: Headphones (3- Astro A50 Game) [Loopback]
[01:44:14] Device native sample rate: 48000 Hz
[01:44:14] Loading audio file: target.wav
[01:44:14]   Loaded: 22050 samples, sample rate: 48000 Hz, duration: 0.459s

============================================================
🎣 CYCLE #1 - STARTING
============================================================
[01:44:15] >>> PRESSING KEY: 'k' <<<
[01:44:15] Now listening for up to 23 seconds...
[01:44:17]   Still listening... 2.1s elapsed (buffer: 2.1s)

============================================================
🐟 TARGET SOUND DETECTED!
============================================================
[01:44:19] Detection Score: 0.847
[01:44:19] >>> PRESSING KEY: 'k' <<<
[01:44:19] Waiting 4.23 seconds before next cycle...
```

## Troubleshooting

### Program Hangs After "ACTIVELY MONITORING"

**Cause**: No audio is playing through the selected device.

**Solution**:

- Make sure audio/game is playing through the correct output device
- Loopback devices only capture audio that is actively being output
- Try playing some music to verify the device is receiving audio

### "Invalid Sample Rate" Error

**Cause**: Device sample rate mismatch.

**Solution**: The latest version auto-detects and resamples. Update to the fixed code.

### Detection Not Working

**Possible causes**:

1. **Threshold too high** → Lower `THRESHOLD` (try 0.5 or 0.4)
2. **Wrong audio files** → Re-record target sounds with better quality
3. **Volume differences** → Ensure similar volume levels when recording
4. **Background noise** → Record in a quieter environment

### Wrong Device Selected

**Solution**: Run `list_devices.py` and verify you're using a loopback device (marked with ⭐).

### Keys Not Pressing

**Possible causes**:

1. **Application not in focus** → Make sure target application is the active window
2. **Admin privileges** → Some apps require running Python as administrator
3. **Wrong key** → Verify 'k' is the correct key for your application

## File Structure

```
project/
│
├── main.py                 # Main program
├── list_devices.py         # Device listing utility
├── target.wav             # Target sound file
├── out-of-range.wav       # Out-of-range sound file
└── README.md              # This file
```

## Technical Details

### Audio Detection Method

The program uses **cross-correlation** to detect audio patterns:

1. Records audio in 0.3-second chunks
2. Maintains a 3-second rolling buffer
3. Compares buffer against target audio using normalized cross-correlation
4. Detects match when correlation score exceeds threshold

### Advantages

- Handles volume variations automatically (normalization)
- Robust to background noise
- Fast real-time detection (300ms chunks)
- Sample rate agnostic (auto-resamples)

## Advanced Configuration

### Adjusting Detection Sensitivity

- **More sensitive** (more false positives): `THRESHOLD = 0.6`
- **Less sensitive** (fewer false positives): `THRESHOLD = 1.8`
- **Default balanced**: `THRESHOLD = 1.2`

### Changing Wait Time Ranges

```python
# Faster cycles (more aggressive)
WAIT_AFTER_TARGET_FOUND = (1, 2)

# Slower cycles (more conservative)
WAIT_AFTER_TARGET_FOUND = (5, 10)
```

### Different Key Bindings

Modify the `press_key()` calls in the main loop:

```python
press_key('e')  # Change from 'k' to 'e'
```

## Safety Notes

- This program simulates keyboard input - use responsibly
- Ensure you comply with the terms of service of any application you use this with
- The program can be stopped at any time with `Ctrl+C`
- Test thoroughly in a safe environment before production use
