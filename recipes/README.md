# Recipes Directory

This directory stores exported light recipe JSON files.
Each file is a portable recipe that can be shared and imported to any WLED device.

## File Format
```json
{
  "name": "Recipe Name",
  "description": "Description",
  "version": "1.0",
  "created_at": "ISO timestamp",
  "format": "doubao-wled-recipe-v1",
  "colors": [[R,G,B], ...],
  "palette_data": [pos,R,G,B, ...],
  "fx": 10,
  "sx": 176,
  "ix": 165,
  "bri": 200,
  "pal": 0,
  "theme": "nature",
  "rationale": "Why this effect was chosen"
}
```

## Usage
- Export: `python3 scripts/export_recipe.py --name "Name" --analysis analysis.json`
- Import: `python3 scripts/import_recipe.py recipes/name.json --save-preset "Name"`
- List: `python3 scripts/export_recipe.py --list`
