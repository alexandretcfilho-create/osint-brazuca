# OSINT.md — Pic OSINT Skill for Claude

A Claude skill that geolocates images from visual evidence alone — no metadata required.

## What it does

Upload a photo and type `/OSINT` to trigger a structured chain-of-clues analysis. Claude will scan the image for location-bearing details and narrow down where it was taken.

**Clue categories analysed:**
- Text (signage, license plates, phone numbers, URLs, menus, graffiti)
- Built environment (architecture, road markings, traffic signs, drain covers)
- Vehicles (make, model, plate format, driving side)
- Nature (vegetation, terrain, sky/sun angle, coastline)
- Reflections and mirrored signage
- In-frame metadata (timestamps, prices, receipts)

**Output includes:**
- Location (city, country, address if pinned)
- Clue chain — each piece of evidence and what it revealed
- Confidence level with the weakest link named
- Map display via `places_map_display_v0`

## Activation

The skill triggers **only** when your message contains the literal token `/OSINT` (case-insensitive). Without it, Claude will not run the OSINT workflow even if an image is attached and you ask "where was this taken?"

If `/OSINT` is present but no image is attached, Claude will ask you to attach one.

## Installing the skill

1. Download `pic-osint.skill` from this repository.
2. In Claude, open **Settings → Skills → Install from file** and select the downloaded file.
3. The skill is now active in your sessions.

## Usage

```
[attach an image]
/OSINT where is this?
```

Or simply:

```
[attach an image]
/OSINT
```

## Ethics & refusals

This skill will **not** help to:
- Identify or locate a private individual from their photo
- Find someone's home address from a personal image they didn't intend to share
- Track a person across multiple images

Geolocating a public scene, a business, or a tourist photo you took yourself is fine. The line is targeting a **person** vs. identifying a **place**.

## License

See [LICENSE](LICENSE).
