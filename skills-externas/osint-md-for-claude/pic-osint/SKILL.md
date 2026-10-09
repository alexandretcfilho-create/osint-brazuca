---
name: pic-osint
description: Geolocate / identify the location where an image was taken using only visual clues (signage, language, architecture, vegetation, vehicles, road markings, license plates, sun angle, reflections, phone numbers, web URLs, etc.). ACTIVATION IS STRICTLY GATED. Only trigger this skill when the user's message contains the literal token "/OSINT" (case-insensitive). Do NOT trigger on generic phrasings like "where was this taken" or "find the location of this photo" - those must be answered without this skill. If the user uploads an image without "/OSINT", do not run OSINT analysis. When activated, follow the structured chain-of-clues workflow inside.
---

# Pic OSINT

Geolocate an image from visual evidence alone.

## Activation

Trigger ONLY when the user's message contains the literal token `/OSINT` (case-insensitive, anywhere in the message). Without this token, do not execute this workflow even if an image is attached and the user asks "where was this taken".

If `/OSINT` is present but no image is attached, ask the user to attach one.

## Workflow

### 1. Inventory clues

Scan the image and list every potentially location-bearing clue. Group them:

- **Text**: signage, posters, shop names, slogans, phone numbers, URLs, license plates, street signs, graffiti, menus, labels. Note the language and script.
- **Built environment**: architecture style, building materials, balcony / window / roof types, electrical poles, road markings, traffic signs, signal types, curb style, drain covers, bollards.
- **Vehicles**: makes, models, plate format and colour, side of road they drive on.
- **Nature**: vegetation, terrain, mountains, coastline, sky / sun angle, weather, snow.
- **People**: clothing, uniforms (only as a regional cue, never to identify individuals).
- **Reflections / windows**: mirrored signage, reflected buildings.
- **Metadata in frame**: timestamps on screens, prices in a currency, store receipts.

Do not skip this step. Write the inventory out before guessing.

### 2. Narrow region

From the strongest clues (language, plate format, phone country code, brand presence, driving side, plug sockets) narrow to country, then region. State which clues drove the narrowing.

### 3. Identify candidates

Use web_search to look up:
- Brand names visible on signage + their locations
- Slogans / taglines (often unique to one chain)
- Phone number prefixes (country / region codes)
- Street name patterns
- Architectural quirks tied to a specific city / district

Search in the local language when relevant - it returns better results than English for non-English regions.

### 4. Verify with places_search

Once you have a candidate address or business, call `places_search` to confirm it exists and get coordinates. Cross-check the returned phone number / address against the clues in the image.

### 5. Output

Structure the response as:

- **Location**: city, country (and address if pinned)
- **Clue chain**: bulleted list, each clue -> what it told you
- **Confidence**: high / medium / low, with the weakest link named
- **Map**: call `places_map_display_v0` with the resolved coordinates

If multiple plausible candidates remain, list them all with the disambiguating clue you'd need.

## Refusals

Refuse if the request is to:
- Identify or locate a private individual (face / clothing / context, not a public figure in a public role)
- Find someone's home address from a personal photo they didn't intend to share
- Track a person across multiple images

Geolocating a public scene, a business, or a tourist photo the user themselves took is fine. The line is targeting a person vs. identifying a place.

## Notes

- Do not include people's names in image_search or web_search queries.
- Do not speculate beyond the evidence. If a clue is ambiguous, say so.
- "Unknown" is a valid output if clues don't converge.
