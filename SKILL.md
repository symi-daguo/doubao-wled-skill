---
name: "wled-light-recipe"
description: >-
  WLED smart light recipe designer. Generates dynamic light effects for WLED LED strips
  by searching images, analyzing color distributions, and applying recipes to LAN WLED devices.
  Invoke when user asks for scene-based lighting (e.g. jasmine blooms, sunset, ocean),
  light recipes, WLED control, or mentions flower/scene/scenery lighting effects.
  Trigger words: light recipe, WLED, light effect, jasmine, flower, sunset, ocean,
  light formula, color from image, scene lighting, 灯光效果, 光配方, 灯光场景, 茉莉花, 花开, 日落, 海洋.
  Do not invoke for non-lighting tasks or generic image search without WLED control intent.
---

# WLED Light Recipe Skill

Design dynamic light recipes for WLED LED strips from natural language scenes.
Turn "jasmine blooms" into a flowing green-white light effect on your LED strip.

## What This Skill Does

Transforms a scene description (like "jasmine blooms" or "sunset over ocean") into a
complete WLED light recipe by:
1. Searching relevant images using built-in web search
2. Analyzing dominant colors from the best image
3. Uploading a custom color palette to WLED
4. Applying the recipe with the best-matching effect
5. Saving as a WLED preset for reuse
6. Exporting a portable JSON recipe file for sharing

## Prerequisites

- A WLED device on the local network (auto-discovered via mDNS or configured in config.json)
- Python 3.8+ with Pillow (pip install Pillow) for image analysis
- Doubao built-in tools: web search, code execution

## Configuration

Before first use, edit `config.json` in the skill directory:
- `wled.ip`: WLED device IP (or leave empty for mDNS auto-discovery)
- `wled.mdns_name`: WLED mDNS hostname (e.g. wled-751f84)
- `layout.type`: LED strip layout - "tv_backlight", "linear", "ring", or "matrix"
- `layout.edges`: For TV backlight, specify top/bottom/left/right LED counts
- `layout.total_leds`: Total LED count (auto-detected if mismatched with device)

## Complete Workflow

When a user requests a light recipe, follow these steps in order:

### Step 1: Discover WLED Device

```bash
python3 scripts/discover_wled.py
```

- If success: note the IP address from the JSON output
- If failed: ask user for WLED IP and update config.json
- The script caches the IP, subsequent calls are fast
- Only re-scan if health check fails

### Step 2: Search for Scene Images

Use Doubao's built-in web search to find images for the scene:
- Search query: "{scene} photo high quality" (e.g. "jasmine flower bloom photo high quality")
- Collect 2-3 image URLs from search results
- Prefer landscape orientation images
- Pick the most representative image URL

### Step 3: Analyze Image Colors

```bash
python3 scripts/analyze_image.py "<image_url>" --json --num-colors 5
```

- The script downloads the image, extracts dominant colors via quantization
- Outputs JSON with: colors, palette_data, recipe_suggestion, wled_state_patch
- Save the full JSON output to a temp file for later use
- Note the `recipe_suggestion.rationale` to explain the choice to user

### Step 4: Export Recipe (Portable File)

```bash
python3 scripts/export_recipe.py \
  --name "{Scene Name}" \
  --analysis /tmp/analysis.json \
  --description "{scene description}" \
  --output recipes/{scene}_{timestamp}.json
```

- This creates a portable recipe file in the `recipes/` directory
- The file can be shared with others and imported on any WLED

### Step 5: Import and Apply Recipe (Full Pipeline)

```bash
python3 scripts/import_recipe.py recipes/{recipe_file} --save-preset "{Scene Name}"
```

This single command does three things:
1. Uploads custom palette to WLED (palette0.json)
2. Applies the recipe with segments configured for your layout
3. Saves the current state as a WLED preset (auto-assigned ID 100-250)

The light effect will start playing immediately and loop until changed.

### Step 6: Report to User

Tell the user:
- The scene name and recipe applied
- The dominant colors extracted (show hex codes)
- The effect name and parameters (FX, speed, intensity)
- The preset ID saved (for recall later)
- The recipe file path (for sharing)

## Common User Requests

### "Create a light recipe for [scene]"

Run the full workflow above. Example scenes:
- "jasmine blooms" -> search jasmine flower images
- "sunset over ocean" -> search sunset ocean images
- "cherry blossom" -> search sakura images
- "aurora borealis" -> search northern lights images
- "forest morning" -> search forest sunrise images

### "Apply recipe [name]"

List saved recipes and apply the matching one:
```bash
python3 scripts/export_recipe.py --list
python3 scripts/import_recipe.py recipes/{file} --save-preset "{name}"
```

### "Stop the lights" / "Turn off"

```bash
python3 -c "
import urllib.request, json
req = urllib.request.Request('http://{WLED_IP}/json/state',
    data=json.dumps({'on': False, 'live': False}).encode(),
    headers={'Content-Type': 'application/json'})
urllib.request.urlopen(req, timeout=3)
print('Lights off')
"
```

### "List all recipes"

```bash
python3 scripts/export_recipe.py --list
```

### "Recall preset [ID]"

```bash
python3 -c "
import urllib.request, json
req = urllib.request.Request('http://{WLED_IP}/json/state',
    data=json.dumps({'ps': PRESET_ID}).encode(),
    headers={'Content-Type': 'application/json'})
urllib.request.urlopen(req, timeout=3)
print(f'Preset {PRESET_ID} recalled')
"
```

## Layout-Aware Segments

The skill automatically creates segments based on `config.json` layout:

- **tv_backlight**: 4 segments (top, right, bottom, left) with rotating colors
- **linear**: 1 segment covering all LEDs
- **ring**: 1 segment (treated as linear)
- **matrix**: 1 segment (user should configure 2D separately)

For TV backlight, each edge gets a slightly rotated color set for visual variety.

## Effect Selection Logic

The `analyze_image.py` script auto-selects effects based on color analysis:

| Avg Hue Range | Theme | Effect Example |
|---|---|---|
| 0-60°, 330-360° | Warm (sunset/fire) | FX 13 Sunset, 35 Fire |
| 60-180° | Nature (forest/spring) | FX 10 Forest, 14 Rivendell |
| 180-270° | Cool (ocean/sky) | FX 9 Rainbow, 9 Ocean-like |
| 270-330° | Party (night/dream) | FX 6 Party, 11 Rainbow |

Speed and intensity are auto-tuned based on saturation and color diversity.

## Error Handling

- **WLED not found**: Run `discover_wled.py --force` or ask user for IP
- **Image download fails**: Try next image URL from search results
- **Palette upload fails**: Continue with built-in palette (non-fatal)
- **Preset save fails**: Recipe still applied, just not saved as preset
- **UDP realtime active**: Script auto-sends `{"live":false}` to take control

## Mobile Doubao Compatibility

This skill is designed to work on both desktop and mobile Doubao:
- All scripts use Python 3 standard library + Pillow only
- Image download uses urllib (no requests dependency required)
- mDNS discovery uses socket.gethostbyname (works on mobile)
- If mobile Doubao cannot execute Python, use the visual analysis fallback:
  1. Doubao analyzes the image directly using vision capability
  2. Output colors as JSON manually
  3. Call apply_recipe.py with the manual recipe JSON

## Recipe File Format

All recipe files use a portable JSON format (`doubao-wled-recipe-v1`):

```json
{
  "name": "Jasmine Blooms",
  "description": "Jasmine flower colors",
  "version": "1.0",
  "format": "doubao-wled-recipe-v1",
  "colors": [[100,150,80], [179,219,149], [240,250,219], [254,229,100]],
  "palette_data": [0,100,150,80, 85,179,219,149, 170,240,250,219, 255,254,229,100],
  "fx": 10,
  "sx": 176,
  "ix": 165,
  "bri": 218,
  "pal": 0,
  "theme": "nature",
  "rationale": "Green/yellow tones - nature/forest effect"
}
```

Files can be shared between users and imported on any WLED device running 0.14+.

## Directory Structure

```
wled-light-recipe/
├── SKILL.md                      # This file (skill instructions)
├── config.json                   # Device & layout configuration
├── scripts/
│   ├── discover_wled.py          # mDNS discovery + health check
│   ├── analyze_image.py          # Image color extraction
│   ├── apply_recipe.py           # Apply recipe to WLED
│   ├── save_preset.py            # Save current state as preset
│   ├── upload_palette.py         # Upload custom palette
│   ├── export_recipe.py          # Export recipe to JSON file
│   ├── import_recipe.py          # Import & apply recipe (full pipeline)
│   └── .wled_cache.json          # Cached WLED IP (auto-generated)
├── references/
│   ├── wled_api.md               # WLED JSON API reference
│   └── effects_palettes.md       # Effects & palettes list
├── recipes/                      # Exported recipe files
└── assets/                       # Test images
```

## Example Conversation

**User**: "I want jasmine bloom lighting effect"

**Assistant**:
1. Discovers WLED device on LAN
2. Searches "jasmine flower bloom photo high quality"
3. Analyzes top image: extracts green (37%), light green (36%), white (20%), yellow (7%)
4. Selects FX 10 (Forest) with speed 176, intensity 165
5. Uploads custom palette with these colors
6. Applies recipe with 4 TV-backlight segments
7. Saves as preset ID 100 "Jasmine Blooms"
8. Exports recipe to recipes/jasmine_blooms_20260729_110259.json

**Response**: "Jasmine Blooms light recipe applied! Colors: #649650 (green), #B3DB95 (light green), #F0FADB (white), #FEE564 (yellow). Effect: Forest (FX 10) at speed 176. Saved as preset 100. Recipe file: recipes/jasmine_blooms_20260729_110259.json"

## Tips for Best Results

- Use specific scene names ("jasmine blooms" vs just "flowers")
- For seasonal themes, add the season ("autumn forest" vs "forest")
- For time-of-day, specify ("sunset ocean" vs "ocean")
- The skill auto-loops the effect until you change it
- To recall a saved recipe: "apply jasmine blooms recipe"
- To share: send the recipe JSON file from recipes/ directory
