---
name: wled-light-recipe
description: WLED智能灯带光配方设计师。通过搜索多张图片、解析颜色分布、生成平顺丝滑的动态灯光效果并应用到局域网WLED设备。当用户需要基于场景的灯光效果（如茉莉花、日落、海洋）、WLED控制、光配方时调用此技能。
version: 1.0.2
author: symi-daguo
category: smart-home
permissions:
  - network
  - read
  - execute
triggers:
  - 灯光效果
  - 光配方
  - 灯光场景
  - WLED
  - 茉莉花
  - 花开
  - 日落
  - 海洋
  - 灯光
  - light recipe
  - light effect
  - WLED control
  - scene lighting
  - jasmine
  - sunset
  - ocean
  - 灯带
  - 灯珠
  - 调色板
---

# WLED Light Recipe Skill v1.0.2

Design smooth, continuously animating light recipes for WLED LED strips.
Turn "jasmine blooms" into a gentle flowing color gradient on your LED strip.

## What This Skill Does

Transforms a scene description (like "jasmine blooms" or "sunset over ocean") into a
smooth WLED light recipe by:
1. Searching 2-4 relevant images using built-in web search
2. Analyzing dominant colors from ALL images
3. Merging colors into a harmonious 3-color palette
4. Selecting a verified SMOOTH effect (Fade/Dissolve/Breathe/Forest/Sunset/Aurora)
5. Applying with SLOW speed (SX=70) and LOW intensity (IX=100) for non-flashing animation
6. Saving as a WLED preset for reuse
7. Exporting a portable JSON recipe file for sharing

## Color Calculation Logic (How Images Become Light Recipes)

### Step 1: Image Download & Preprocessing
- Download image from URL using urllib
- Load with PIL (Pillow), resize to 150x150 max for speed
- Convert to RGB mode (remove alpha channel)

### Step 2: Color Quantization (PIL quantize method)
- Use PIL `Image.quantize(colors=N)` to reduce colors to top N (default 5)
- This groups similar pixels and returns the most representative colors
- Fallback: 4x4x4 RGB histogram (64 buckets) if quantize fails

### Step 3: HSV Conversion & Filtering
Each extracted RGB color is converted to HSV and filtered:
- `min_brightness=20`: Skip colors with V<20 (too dark, invisible on LED strip)
- `min_saturation=30`: Colors with S<30 (gray/white) are separated into `gray_colors`
- If saturated colors < 3: gray_colors are added back as fallback (preserves white flowers, clouds, snow)

### Step 4: Color Ratio Calculation
- Each color's ratio = pixel_count / total_pixels
- Colors are sorted by ratio descending (most dominant first)
- Top N colors are selected and ratios normalized to sum=1.0

### Step 5: Multi-Image Color Merging
When analyzing 2-4 images, colors from ALL images are merged:
1. Collect all colors from all images into one pool
2. Sort by ratio descending
3. Pick top 3 DISTINCT colors (hue difference > 30 degrees, RGB difference > 50)
4. This ensures color variety (e.g., green from leaves + yellow from flower center + white from petals)

### Step 6: Theme Detection & FX Selection
Dominant theme is determined by averaging hue values weighted by ratio:
- Hue 0-60°, 330-360° = "warm" → FX 13 (Sunset, smooth warm gradient)
- Hue 60-180° = "nature" → FX 12 (Fade, smoothest color crossfade)
- Hue 180-270° = "cool" → FX 50 (Aurora, smooth northern lights)
- Hue 270-330° = "party" → FX 12 (Fade, default smooth)

### Step 7: Parameter Tuning
- SX (Speed) = 70: Slow, gentle movement (range 50-90, NEVER > 128)
- IX (Intensity) = 100: Subtle, balanced (range 80-120, NEVER > 150)
- BRI (Brightness) = 180: Comfortable (range 150-200)
- These values are verified on real WLED 17.0.0-devV5 to produce smooth, non-flashing animation

## Dynamic Animation Strategy (1-Minute Loop)

### How WLED FX 12 (Fade) Works (Source Code Analysis)

From WLED source code `wled00/FX.cpp` line 453:
```cpp
void mode_fade(void) {
  unsigned counter = (strip.now * ((SEGMENT.speed >> 3) + 10));
  uint8_t lum = triwave16(counter) >> 8;
  // Blend between color1 and color2 based on lum (0-255)
  for (unsigned i = 0; i < SEGLEN; i++) {
    SEGMENT.setPixelColor(i, color_blend(SEGCOLOR(1), color_from_palette(...), lum));
  }
}
```

### Fade Period Calculation (SX=70)
- `counter = now_ms * ((70 >> 3) + 10) = now_ms * 18`
- `triwave16()` has period 65536 (2^16)
- One complete fade cycle: 65536 / 18 = 3641ms = 3.6 seconds
- In 60 seconds: 60 / 3.6 = 16.7 fade cycles

### What This Means for the User
- Every 3.6 seconds: colors smoothly transition from color1 → color2 → color1
- In 1 minute: approximately 16-17 complete color transitions
- Effect is CONTINUOUSLY animated, never static
- Effect LOOPS INFINITELY until user changes it (no 1-minute timeout)
- No playlist needed - single FX with col array handles everything

### Why Not Playlist?
WLED 17.0.0-devV5 playlist auto-advance has a bug (playlist saves but does not auto-advance).
Single FX with col array is more reliable across all WLED versions.
The "1-minute" concept means: within any 1-minute window, user sees ~17 color transitions.

## CRITICAL RULES (NEVER Violate)

### Rule 1: ONLY Use Smooth Effects
NEVER use flashing/strobe effects. The following are FORBIDDEN:
- FX 77 (Pacifica) - causes strobe at high speed
- FX 75 (Flow) - causes strobe
- FX 35 (Fire) - flickers like fire
- FX 33 (Fireworks) - strobe flashes
- FX 1 (Blink) - obvious strobe
- FX 17 (Twinkle) - random flashes
- FX 6 (Party) - rapid strobe
- FX 11 (Rainbow Runner) - fast strobe
- FX 19 (Splash) - abrupt color jumps

ONLY use these verified SMOOTH effects:
- FX 12 (Fade) - smoothest color crossfade [DEFAULT]
- FX 18 (Dissolve) - soft color melting
- FX 2 (Breathe) - gentle breathing
- FX 10 (Forest) - smooth nature gradient
- FX 14 (Rivendell) - cool forest, smooth
- FX 13 (Sunset) - warm gradient, smooth
- FX 21 (Sunset 2) - warm sunset
- FX 50 (Aurora) - northern lights, smooth
- FX 26 (Beech) - light green nature
- FX 20 (Pastel) - soft muted colors

### Rule 2: NEVER Use High Speed/Intensity
- SX (Speed): MUST be 50-90 (slow, gentle)
- IX (Intensity): MUST be 80-120 (subtle, balanced)
- BRI (Brightness): MUST be 150-200 (comfortable)
- SX > 128 causes strobe/flashing - FORBIDDEN
- IX > 150 causes intense flashing - FORBIDDEN

### Rule 3: Multi-Image Fusion
Always analyze 2-4 images and MERGE colors. Single image = limited palette.
Multiple images = rich, harmonious color variety.

## Prerequisites

- A WLED device on the local network (auto-discovered via mDNS or configured in config.json)
- Python 3.8+ with Pillow (pip install Pillow) for image analysis
- Doubao built-in tools: web search, code execution

## Configuration

Before first use, edit `config.json` in the skill directory:
- `wled.ip`: WLED device IP (leave empty for mDNS auto-discovery)
- `layout.type`: LED strip layout - "tv_backlight", "linear", "ring", or "matrix" (default: "linear")
- `layout.total_leds`: Total LED count (set to 0 to auto-detect from device)

## Complete Workflow (STANDARD TEMPLATE)

When a user requests a light recipe, follow these steps EXACTLY.
**Any AI agent can follow this template to produce smooth light effects.**

### Step 1: Discover WLED Device

```bash
python3 scripts/discover_wled.py
```

- Note the `ip` field from JSON output, use as {WLED_IP} in later steps
- If failed: ask user "请告诉我WLED灯带的IP地址"

### Step 2: Search for 2-4 Real High-Quality Landscape Images (CRITICAL)

Use Doubao's built-in web search to find REAL photographs for the scene.

**MANDATORY Image Requirements:**
1. **REAL photographs only** - NO illustrations, NO cartoons, NO AI-generated art, NO clipart
2. **Landscape orientation** - width > height (portrait images will be REJECTED)
3. **High resolution** - minimum 1280px width (1920x1080 or 4K preferred)
4. **2-4 images** - need multiple images for rich color variety
5. **Diverse content** - different angles, lighting, backgrounds of the same scene

**Search Query Template:**
```
{scene} photo high quality landscape 4K
```

Examples:
- "jasmine flower bloom photo high quality landscape 4K"
- "sunset ocean photo high quality landscape 4K"
- "cherry blossom photo high quality landscape 4K"

**How to Get Image URLs:**
1. Use Doubao web search with the query above
2. From search results, find image URLs (https://...jpg/.png)
3. Good sources: wallpaper sites (wallpaperaccess.com), stock photo sites (pexels.com, unsplash.com)
4. Verify each URL is a direct image link (ends with .jpg, .png, or returns image content)
5. The script will automatically validate: landscape orientation + min 1280px width
6. Portrait/low-res images are auto-skipped, so collect 4-6 URLs to ensure 2-3 valid ones

**IMPORTANT:** Do NOT use locally generated or synthetic images. The skill validates image quality
and will reject images that don't meet landscape + HD requirements.

### Step 3: Apply Dynamic Recipe (ONE COMMAND - RECOMMENDED)

```bash
python3 scripts/apply_dynamic_recipe.py <url1> <url2> <url3> --scene "Scene Name" --save-preset --show-images --device {ADB_DEVICE}
```

This single command does EVERYTHING:
1. Downloads and validates all images (landscape + HD only, auto-skip invalid)
2. Analyzes colors from valid images using PIL quantize + white detection
3. Merges colors into 3-color palette (preserves white flowers, clouds, etc.)
4. Selects best SMOOTH effect (Fade/Dissolve/Breathe/Forest/Sunset/Aurora)
5. Applies with SX=70, IX=100 (slow, smooth, non-flashing)
6. Saves as WLED preset for reuse
7. **NEW**: Pushes source images to Android device screen via ADB (if --show-images)

**Parameters:**
- `--show-images`: Push source images to screen so customer sees where colors came from
- `--device 192.168.2.43:5555`: ADB device address (Android TV, amplifier with screen)
- `--duration 30`: How long to display images (seconds, default 30)
- `--local`: If no ADB device, open images on local computer instead
- `--save-preset`: Save as WLED preset for later recall

The light effect will start playing immediately with smooth, continuous animation.
Source images will display on screen simultaneously so customer understands the color origin.

### Step 4: Report to User

Tell the user:
- The scene name and recipe applied
- The merged colors extracted (show hex codes)
- The effect name (FX 12 Fade, FX 50 Aurora, etc.)
- The speed (SX=70 slow) and intensity (IX=100 subtle)
- The preset ID saved (for recall later)
- Confirm: "效果平顺丝滑，持续循环播放，无闪烁"

## Common User Requests

### "Create a smooth light recipe for [scene]"

Run the full workflow above with 2-4 images. Example:
```bash
python3 scripts/apply_dynamic_recipe.py \
  "https://example.com/jasmine1.jpg" \
  "https://example.com/jasmine2.jpg" \
  "https://example.com/jasmine3.jpg" \
  --scene "Jasmine Blooms" --save-preset
```

### "Apply recipe [name]"

List saved recipes and apply:
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

## Effect Selection Logic (Auto)

The `analyze_multi_images.py` auto-selects SMOOTH effects based on dominant theme:

| Dominant Theme | Selected FX | Description |
|---|---|---|
| warm (sunset/fire) | 13 (Sunset) | Smooth warm gradient |
| nature (forest/spring) | 12 (Fade) | Smoothest color crossfade |
| cool (ocean/sky) | 50 (Aurora) | Smooth northern lights |
| party (night/dream) | 12 (Fade) | Default smooth |

All effects use SX=70 (slow), IX=100 (subtle) for non-flashing animation.

## Layout-Aware Segments

The skill automatically creates segments based on `config.json` layout:

- **tv_backlight**: 4 segments (top, right, bottom, left) with rotating colors
- **linear**: 1 segment covering all LEDs
- **ring**: 1 segment (treated as linear)
- **matrix**: 1 segment (user should configure 2D separately)

## Error Handling

- **WLED not found**: Run `discover_wled.py --force` or ask user for IP
- **Image download fails**: Try next image URL from search results
- **UDP realtime active (live=true)**: Script auto-sends `{"live":false}` to take control
- **Preset save fails**: Recipe still applied, just not saved as preset

## Recipe File Format

All recipe files use a portable JSON format (`doubao-wled-recipe-v1`):

```json
{
  "name": "Jasmine Blooms",
  "description": "Jasmine flower colors",
  "version": "2.0",
  "format": "doubao-wled-recipe-v1",
  "colors": [[100,150,80], [179,219,149], [240,250,219]],
  "palette_data": [0,100,150,80, 85,179,219,149, 170,240,250,219, 255,254,229,100],
  "fx": 12,
  "sx": 70,
  "ix": 100,
  "bri": 180,
  "pal": 0,
  "theme": "nature",
  "rationale": "Merged 3 images, smooth FX 12 Fade for continuous animation"
}
```

## Directory Structure

```
wled-light-recipe/
├── SKILL.md                      # This file (skill instructions)
├── config.json                   # Device & layout configuration
├── config.example.json           # Config template with documentation
├── scripts/
│   ├── discover_wled.py          # mDNS discovery + health check
│   ├── analyze_image.py          # Single image color extraction + quality validation
│   ├── analyze_multi_images.py   # Multi-image analysis + color merging
│   ├── apply_recipe.py           # Apply recipe to WLED
│   ├── apply_dynamic_recipe.py   # ONE-SHOT pipeline (RECOMMENDED)
│   ├── show_recipe_gallery.py    # NEW: Push images to screen via ADB
│   ├── build_playlist.py         # Playlist builder (advanced)
│   ├── save_preset.py            # Save current state as preset
│   ├── upload_palette.py         # Upload custom palette
│   ├── export_recipe.py          # Export recipe to JSON file
│   ├── import_recipe.py          # Import & apply recipe
│   └── .wled_cache.json          # Cached WLED IP (auto-generated)
├── references/
│   ├── wled_api.md               # WLED JSON API reference
│   └── effects_palettes.md       # Effects & palettes list
├── recipes/                      # Exported recipe files
└── assets/                       # Test images (real photos only)
```

## Example Conversation

**User**: "我需要茉莉花盛开的灯光效果"

**Assistant**:
1. Discovers WLED device on LAN (e.g. 192.168.2.66)
2. Searches "jasmine flower bloom photo high quality" - gets 3 image URLs
3. Runs: `python3 scripts/apply_dynamic_recipe.py url1 url2 url3 --scene "Jasmine Blooms" --save-preset`
4. Script analyzes all 3 images, merges colors, selects FX 12 (Fade)
5. Applies with SX=70 (slow), IX=100 (subtle) - smooth, non-flashing
6. Saves as preset ID 100

**Response**: "茉莉花盛开光配方已应用！融合颜色：#8DA866 (绿), #B3DB95 (浅绿), #F0FADB (白)。效果：Fade (FX 12)，速度70（慢速），强度100（柔和）。效果平顺丝滑，持续循环播放，无闪烁。已保存为预设100。"

## Tips for Best Results

- Use 2-4 images for rich color variety (single image = limited palette)
- Use specific scene names ("jasmine blooms" vs just "flowers")
- For seasonal themes, add the season ("autumn forest" vs "forest")
- The skill auto-loops the effect until you change it
- To recall a saved recipe: "apply jasmine blooms recipe"
- To share: send the recipe JSON file from recipes/ directory

## Version History

- v1.0.2: 新增图片质量验证(landscape+1280px)、ADB推送图片到屏幕、白色检测优化
- v1.0.1: 修复暴闪问题，仅用平滑FX(Fade/Dissolve/Breathe)，SX=70/IX=100，多图融合
- v1.0.0: 初始版本，单图分析，存在闪烁问题
