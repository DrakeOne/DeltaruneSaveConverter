# Deltarune Save File Format Documentation

This document describes the save file formats used by DELTARUNE for both PC and Console versions.

## Overview

DELTARUNE uses different save formats depending on the platform:
- **PC**: Plain text files with one value per line
- **Console**: Compressed format with GameMaker Studio (GMS) encoded arrays
- **SAV Container**: JSON format used to bundle multiple save files

## File Structure

### Chapter 1 Save Files

#### Console Format (223 lines)
```
Line 1: Player name (string)
Line 2: Other names (GMS-encoded string array[6])
Line 3-5: Character selection (3 numbers)
Line 6: Gold
Line 7: XP
Line 8: Level
Line 9: Inventory count
Line 10: Inventory capacity
Line 11: Dark zone flag
Line 12-21: Character stats (GMS-encoded arrays)
Line 22-197: Item stats and spells
Line 198-201: Inventory items (GMS-encoded arrays)
Line 202-217: Light world stats
Line 218-219: Light world items (GMS-encoded arrays)
Line 220: Flags (GMS-encoded array[9999])
Line 221: Plot progress
Line 222: Current room
Line 223: Play time
```

#### PC Format (10,318 lines)
- Same data as console but arrays are expanded
- Each array element gets its own line
- No GMS encoding

### Chapter 2 Save Files

#### Console Format (308 lines)
- Similar structure to Chapter 1
- 5 character slots instead of 4
- More items (48 weapons/armor vs 13)
- 72 pocket items
- 2500 flags instead of 9999

#### PC Format (3,055 lines)
- Expanded version of console format

## GMS List Format

GameMaker Studio lists use a specific binary format:

### Header
- Bytes 0-3: Magic number (0x012E or 0x012F)
- Bytes 4-7: Item count (little-endian)

### Items
Each item consists of:
- Bytes 0-3: Type (0 = real number, 1 = string)
- For numbers: 8 bytes (double, little-endian)
- For strings: 4 bytes length + ASCII string data

### Valid Headers
- `2E010000`: Standard list
- `2F010000`: Alternative format
- Additional headers may exist for different console versions

## SAV Container Format

Console saves are bundled in a JSON container:
```json
{
  "default": "",
  "filech1_0": "save file contents...",
  "filech1_1": "save file contents...",
  "filech1_2": "save file contents...",
  "dr.ini": "ini file contents..."
}
```

## Data Validation

### Expected Line Counts
- Chapter 1 Console: 223 lines
- Chapter 1 PC: 10,318 lines
- Chapter 2 Console: 308 lines
- Chapter 2 PC: 3,055 lines

### Common Issues
1. **Nintendo Switch Format**: May use different GMS headers
2. **Corrupted Arrays**: Invalid GMS encoding causes decode errors
3. **Missing Lines**: Incomplete saves from interrupted writes
4. **Dog Screen**: Indicates corrupted save data

## Value Ranges

### Valid Ranges
- HP/MaxHP: 0-999
- Level: 1-99
- Gold: 0-9999
- Flags: 0 or 1 (some special flags may have other values)
- Room IDs: Specific to game areas
- Time: Seconds played

### Special Values
- Flag 241 (Ch1): Jevil fight status
- Flag 571 (Ch2): Spamton NEO status
- Flag 912: Language setting

## Error Recovery

When encountering corrupted data:
1. Check line count first
2. Validate GMS headers
3. Verify numeric values are in valid ranges
4. Use default values for missing data
5. Log specific errors for debugging