# WLED Effects & Palettes Reference

Curated list of WLED effects and palettes commonly used in light recipes.

## Key Effects (FX ID: Name)

### Static & Simple (0-9)
| ID | Name | Best For |
|---|---|---|
| 0 | Solid | Single color display |
| 1 | Blink | Alert/pulse effects |
| 2 | Breathe | Gentle breathing animation |
| 3 | Wipe | Color wipe transitions |
| 5 | Random Colors | Party/ambient |
| 7 | Dynamic | Color shifting |
| 8 | Colorloop | Rainbow color cycle |
| 9 | Rainbow | Classic rainbow flow |

### Nature & Atmospheric (10-25)
| ID | Name | Best For |
|---|---|---|
| 10 | Forest | Green nature scenes, leaves, forest |
| 12 | Fade | Smooth color transitions |
| 14 | Rivendell | Cool forest, mystical |
| 15 | Breeze | Light blue-green, ocean breeze |
| 17 | Twinkle | Starry night, sparkling |
| 18 | Dissolve | Soft color melting |
| 21 | Sunset 2 | Sunset/dusk scenes |
| 22 | Beach | Beach, sand, ocean shore |
| 25 | Landscape | Natural landscapes |
| 26 | Beech | Light green nature |

### Warm & Fire (35-40)
| ID | Name | Best For |
|---|---|---|
| 13 | Sunset | Sunset, warm glow |
| 35 | Fire | Fireplace, campfire |
| 36 | Icefire | Cool-warm contrast |
| 37 | Cyane | Cyan glow |
| 38 | Light Pink | Soft pink, floral |
| 39 | Autumn | Autumn leaves, orange-red |

### Cool & Water (9, 50-55)
| ID | Name | Best For |
|---|---|---|
| 9 | Rainbow | Rainbow, color spectrum |
| 50 | Aurora | Northern lights |
| 51 | Atlantica | Ocean deep, atlantic |

### Party & Vibrant (6, 11-12, 56-70)
| ID | Name | Best For |
|---|---|---|
| 6 | Party | Party, celebration |
| 11 | Rainbow Bands | Striped rainbow |
| 12 | Rainbow | Full spectrum |
| 19 | Splash | Bold color splashes |
| 20 | Pastel | Soft muted colors |
| 27 | Sherbet | Sweet pastel colors |

## Built-in Palettes (Pal ID: Name)

### Special (0-5)
| ID | Name | Description |
|---|---|---|
| 0 | Default | Auto-selected by effect |
| 1 | Random Cycle | Changes every few seconds |
| 2 | Color 1 | Primary color only |
| 3 | Colors 1&2 | Gradient primary-secondary |
| 4 | Color Gradient | Mix all 3 colors |
| 5 | Colors Only | Discrete 3-color |

### Nature (10, 14, 22, 25-26)
| ID | Name | Color Theme |
|---|---|---|
| 10 | Forest | Green nature |
| 14 | Rivendell | Cool forest green-blue |
| 22 | Beach | Sand and sea |
| 25 | Landscape | Natural earth tones |
| 26 | Beech | Light green |

### Warm (13, 21, 35-36)
| ID | Name | Color Theme |
|---|---|---|
| 13 | Sunset | Orange-red gradient |
| 21 | Sunset 2 | Warm sunset |
| 35 | Fire | Red-orange-yellow |
| 36 | Icefire | Ice and fire contrast |

### Cool (9, 50-51)
| ID | Name | Color Theme |
|---|---|---|
| 9 | Ocean | Blue-green water |
| 50 | Aurora | Northern lights |
| 51 | Atlantica | Deep ocean |

### Floral (38, 49)
| ID | Name | Color Theme |
|---|---|---|
| 38 | Light Pink | Soft pink floral |
| 49 | Sakura | Cherry blossom pink |

## Custom Palettes

WLED 0.14+ supports up to 10 custom palettes (ID 0-9), stored as:
- `/palette0.json` through `/palette9.json`

Upload via `/upload` endpoint. Appear as "~ Custom N ~" in palette list after reboot.

Use `upload_palette.py` to upload from this skill.

## Effect Parameter Tuning

### Speed (SX)
- 0-50: Very slow, meditative
- 50-128: Gentle, ambient
- 128-200: Moderate, lively
- 200-255: Fast, energetic

### Intensity (IX)
- 0-50: Subtle, sparse
- 50-128: Balanced
- 128-200: Rich, dense
- 200-255: Intense, saturated

### Brightness (BRI)
- 0-80: Dim, nightlight
- 80-150: Soft, ambient
- 150-200: Normal, comfortable
- 200-255: Bright, vivid

## Theme-to-Effect Mapping

| Theme | Hue Range | Recommended FX | Recommended Pal |
|---|---|---|---|
| Warm/Sunset | 0-60°, 330-360° | 13, 21, 35 | 13, 21, 35 |
| Nature/Forest | 60-180° | 10, 14, 25 | 10, 14, 25 |
| Cool/Ocean | 180-270° | 9, 50, 51 | 9, 50, 51 |
| Party/Night | 270-330° | 6, 11, 12 | 6, 11, 12 |
| Floral | Various | 10, 38 | 38, 49 |

## Auto-Selection Logic (analyze_image.py)

The script uses these rules to auto-select effect:
1. Calculate avg hue weighted by color ratio
2. Map hue to theme (warm/nature/cool/party)
3. Pick first effect from theme's effect list
4. Tune speed by saturation (high sat = slower, dramatic)
5. Tune intensity by hue range (more diverse = higher intensity)
6. Tune brightness by avg value

Override auto-selection by passing explicit `--fx`, `--sx`, `--ix` to export_recipe.py.
