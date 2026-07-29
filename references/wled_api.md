# WLED JSON API Reference

Complete reference for WLED JSON API used by this skill.

## Base URL

```
http://<WLED_IP>/json
```

## Endpoints

### GET /json - Full State
Returns complete device state with `state`, `info`, `effects`, `palettes`.

### GET /json/info - Device Info
Returns device capabilities (version, LED count, max segments, effect/palette counts).

Key fields:
- `ver`: Firmware version
- `leds.count`: Total LED count
- `leds.rgbw`: RGBW support
- `leds.maxseg`: Max segments
- `fxcount`: Effect count
- `palcount`: Palette count
- `live`: UDP realtime mode active

### GET /json/state - Current State
Returns current light state (on, bri, segments, etc).

### POST /json/state - Update State
Send partial state object to update. Only specified fields are changed.

### GET /json/eff - Effects List
Array of effect names (index = effect ID).

### GET /json/pal - Palettes List
Array of palette names (index = palette ID).

## State Object Structure

```json
{
  "on": true,                    // Master power
  "bri": 200,                    // Master brightness (0-255)
  "transition": 0,               // Transition time (0.1s units, 0=instant)
  "ps": -1,                      // Current preset ID (-1=none)
  "pl": -1,                      // Current playlist ID
  "live": false,                 // UDP realtime mode (set false to take control)
  "mainseg": 0,                  // Main segment ID
  "seg": [...]                   // Array of segment objects
}
```

## Segment Object

```json
{
  "id": 0,                       // Segment ID (0-31)
  "start": 0,                    // First LED (inclusive)
  "stop": 16,                    // Last LED (exclusive)
  "len": 16,                     // Length (alternative to stop)
  "on": true,                    // Segment power
  "bri": 255,                    // Segment brightness
  "fx": 10,                      // Effect ID
  "sx": 128,                     // Speed (0-255)
  "ix": 200,                     // Intensity (0-255)
  "pal": 0,                      // Palette ID
  "col": [[R,G,B], [R,G,B], [R,G,B]],  // 3 color slots
  "cct": 127,                    // Color temperature (0-255)
  "rev": false,                  // Reverse direction
  "mi": false,                   // Mirror
  "grp": 1,                      // Grouping
  "spc": 0,                      // Spacing
  "of": 0,                       // Offset
  "frz": false                   // Freeze effect
}
```

## Preset Operations

### Save Current State as Preset
```json
POST /json/state
{"psave": 100, "n": "Preset Name"}
```

### Recall Preset
```json
POST /json/state
{"ps": 100}
```

### Get All Presets
```
GET /presets.json
```

## Custom Palette Upload

Upload palette file to `/palette{N}.json` (N=0-9) via multipart POST to `/upload`:

File content format:
```json
{
  "palette": [0, 255, 0, 0, 85, 0, 255, 0, 170, 0, 0, 255, 255, 255, 255, 255]
}
```

Format: `[position, R, G, B, position, R, G, B, ...]`
- Position: 0-255 (gradient position)
- RGB: 0-255

After upload, reboot WLED for palette to appear as "~ Custom N ~".

## Common API Patterns

### Take Control from UDP Realtime
```json
POST /json/state
{"live": false, "on": true}
```

### Set Single Color (All LEDs)
```json
POST /json/state
{"on": true, "seg": [{"id": 0, "fx": 0, "col": [[255, 100, 50]]}]}
```

### Set Multi-Segment with Different Effects
```json
POST /json/state
{"seg": [
  {"id": 0, "start": 0, "stop": 16, "fx": 10, "col": [[100,150,80]]},
  {"id": 1, "start": 16, "stop": 28, "fx": 10, "col": [[179,219,149]]}
]}
```

### Delete Segment
```json
POST /json/state
{"seg": [{"id": 1, "stop": 0}]}
```

### Toggle Power
```json
POST /json/state
{"on": "t"}
```

### Cycle Effects
```json
POST /json/state
{"seg": {"fx": "~"}}    // Next effect
{"seg": {"fx": "~-"}}   // Previous effect
{"seg": {"fx": "r"}}    // Random effect
```

## HTTP API (Alternative)

Simpler URL-based API at `/win`:
```
http://<IP>/win&T=1&A=200&FX=10&SX=128&IX=200&R=100&G=150&B=80
```

- T: 0=off, 1=on, 2=toggle
- A: brightness (0-255)
- FX: effect ID
- SX: speed, IX: intensity
- R/G/B: primary color
- PS: save preset, PL: load preset

## Error Handling

- HTTP 200: Success
- HTTP 400: Bad request (invalid JSON)
- HTTP 500: Internal error
- Timeout: Device offline or unreachable

Always set timeout (3-5 seconds) and handle exceptions gracefully.
