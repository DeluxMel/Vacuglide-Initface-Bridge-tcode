# Vacuglide-Initface-Bridge-tcode
small python bridge that connects vacuglide to intiface using the tcode-v03 protocol ,like diglet48 did it for restim. do with this code what you want, i'm 'happy' with what i got hopefully the official support will come soon.... 


# VacuGlide ↔ Intiface TCode Bridge

## Overview

This script acts as a **bridge between Intiface (Buttplug) and the VacuGlide API**.

It listens for **TCode commands** (e.g. `L074I400`) coming from Intiface via WebSocket and converts them into **VacuGlide speed commands** using the official API.

The core idea:

* TCode provides **position + duration**
* The script converts this into **motion speed**
* That speed is sent to the VacuGlide as a continuous control signal

---
Required Python Packages
Package	Purpose
aiohttp	Asynchronous HTTP client (for VacuGlide API calls)
websockets	WebSocket client (connects to Intiface)
requests	Synchronous HTTP requests (cluster discovery)
re	Regular expressions (built-in, no install needed)
asyncio	Async framework (built-in, no install needed)
json	JSON parsing (built-in, no install needed)
time	Time utilities (built-in, no install needed)
sys	System functions (built-in, no install needed)
signal	Optional for safety signals (built-in, no install needed)

"pip install aiohttp websockets requests"

## How It Works

### 1. Connection

* Connects to Intiface via WebSocket (`WS_URL`)
* Performs a simple handshake
* Continuously listens for incoming TCode messages

---

### 2. TCode Parsing

Incoming messages look like:

```
L074I400
```

Meaning:

* `L074` → position (0–100)
* `I400` → duration in milliseconds

The script:

1. Tracks the previous position
2. Computes movement distance
3. Uses duration to calculate speed:

```
speed = distance / time
```

---

### 3. Speed Conversion

The computed speed is:

* normalized to a 0–100 range
* sent to the VacuGlide as `targetSpeed`

---

### 4. VacuGlide Control

The script sends:

* `TARGET_SPEED_PLAYING` → when movement is active
* `TARGET_SPEED_PAUSED` → when stopping

---

### 5. Idle Safety

If no TCode is received for a short time:

```
IDLE_STOP_SECONDS = 0.5
```

→ The script automatically sends a STOP command

---

## Configuration

### Required

```python
VACUGLIDE_TOKEN = "YOUR_TOKEN"
```

---

### Connection

```python
WS_URL = "ws://127.0.0.1:54817"
```

---

### Speed Behavior

#### `MAX_EXPECTED_SPEED` (inside parsing function)

Controls how motion translates to speed.

* Lower value → more aggressive / faster response
* Higher value → smoother / slower response

Typical range:

```
150 – 400
```

---

#### `MIN_SPEED`

```python
MIN_SPEED = 5
```

Prevents the device from stalling at very low speeds.

---

#### `SPEED_THRESHOLD`

```python
SPEED_THRESHOLD = 2
```

Minimum change required before sending a new speed command.

Helps reduce API spam.

---

#### `THROTTLE_INTERVAL`

```python
THROTTLE_INTERVAL = 0.3
```

Minimum time between API updates.

Lower = more responsive
Higher = more stable / less network load

---

## Timing / Responsiveness

The system depends heavily on timing:

* TCode duration (`Ixxx`) controls motion speed
* WebSocket latency affects responsiveness
* API throttling affects smoothness

If motion feels:

* **laggy** → reduce `THROTTLE_INTERVAL`
* **too jumpy** → increase it

---

## Known Limitations

### 1. No built-in device safety timeout

The VacuGlide:

* continues running at last speed
* until explicitly stopped

If the script crashes:

* the device may keep running

---

### 2. Speed-only control

This script:

* does NOT control position directly
* only derives and sends speed

---

### 3. No smoothing/interpolation

Motion is based purely on incoming TCode steps:

* fast changes may feel abrupt

---

## Where to Modify Behavior

### Change speed sensitivity

Inside `handle_tcode_and_get_effective_speed`:

```python
MAX_EXPECTED_SPEED = 300.0
```

---

### Adjust idle stop timing

```python
IDLE_STOP_SECONDS = 0.5
```

---

### Adjust API rate limiting

```python
THROTTLE_INTERVAL = 0.3
```

---

### Adjust minimum usable speed

```python
MIN_SPEED = 5
```

---

## Typical Data Flow

```
Intiface (TCode)
    ↓
WebSocket
    ↓
This Script
    ↓ (convert position + time → speed)
VacuGlide API
    ↓
Device Motion
```

---

## Notes

* This script assumes TCode format: `LxxxIyyy`
* Position is expected in range `0–100`
* Duration is expected in milliseconds
* Speed is derived, not directly provided

---

## Future Improvements (optional)

* Motion smoothing / interpolation
* Direction-based valve control
* Heartbeat-based safety system
* External watchdog for crash safety

---

## Disclaimer

This script controls physical hardware.
Improper behavior or crashes may result in the device continuing to run.

Use with caution and consider adding additional safety mechanisms if needed.
