## floatingsaunarenders

> Based on the Level 1 Floor Plan: 21' x 61' x 15' tall floating dock structure attached to the existing dock at Heritage Harbour, Vancouver, Canada with Reception (9.5'x15'), Changeroom (12'x20'), Outdoor Lounge (17'x6.5') with plunge pool, Indoor Lounge (17'x13.5'), Hallway, Rest Room 1 (5'x6'), Rest Room 2 (9'x6'), Maintenance & Mechanical (7.5'x6'), and Sauna (22'x22.5') with teak benches, electric sauna heaters, and floor-to-ceiling windows on the north and east walls that meet at the northeast corner in a nearly seamless glass-to-glass butt joint (no structural column at the corner). Indoor Lounge has floor-to-ceiling windows north and south. Reception entry via outward-opening French doors on the south side (only south entrance). The south side faces and attaches to the existing Heritage Harbour dock.

# Floating Sauna — Gemini (Nano Banana Pro) Prompts (2026)

Based on the Level 1 Floor Plan: 21' x 61' x 15' tall floating dock structure attached to the existing dock at Heritage Harbour, Vancouver, Canada with Reception (9.5'x15'), Changeroom (12'x20'), Outdoor Lounge (17'x6.5') with plunge pool, Indoor Lounge (17'x13.5'), Hallway, Rest Room 1 (5'x6'), Rest Room 2 (9'x6'), Maintenance & Mechanical (7.5'x6'), and Sauna (22'x22.5') with teak benches, electric sauna heaters, and floor-to-ceiling windows on the north and east walls that meet at the northeast corner in a nearly seamless glass-to-glass butt joint (no structural column at the corner). Indoor Lounge has floor-to-ceiling windows north and south. Reception entry via outward-opening French doors on the south side (only south entrance). The south side faces and attaches to the existing Heritage Harbour dock.

---

## Platform Settings

| Setting | Architectural Drawings | Photorealistic |
|---------|----------------------|----------------|
| **Resolution** | `"2K"` or `"4K"` via API (uppercase K) | `"4K"` via API |
| **Aspect ratio** | Set via `imageConfig.aspectRatio: "16:9"` | `"16:9"` or `"21:9"` |
| **Thinking mode** | `thinking_level="High"` for complex layouts | `thinking_level="High"` |
| **Key trick** | Narrative sentences > keyword lists | Upload floor plan as reference image |

> Nano Banana Pro accepts up to 14 reference images. Upload the floor plan + material swatches + style references together. No transparent background support. Expect 503 errors during peak hours.

---

## Prompt Writing Guide — Lessons Learned (Gemini Nano Banana Pro / Gemini 2.0 Flash)

These lessons were discovered while iterating on photorealistic architectural prompts with the **Gemini Nano Banana Pro** image generation model (based on Gemini 2.0 Flash). They may or may not apply to other Gemini models or other image generation platforms (Midjourney, DALL-E, Flux, etc.).

### 1. Material boundaries must be stated from BOTH sides
When two sections share a boundary (e.g., sauna glass meets outdoor lounge), describe the boundary explicitly in **both** section descriptions. Say where material A **stops** and say where material B **starts**. If you only describe it from one side, Gemini Nano Banana Pro will bleed the more visually dominant material (usually glass) across the boundary.

> **Example:** The sauna section says "the glass STOPS at the right edge... where it meets a thick SOLID pine wall." The outdoor lounge section says "the LEFT side wall is SOLID thermally modified pine — NO windows or glass."

### 2. Matching side walls need explicit description
If two side walls of an open section (like a deck) should look the same, say so directly. Don't assume the model will infer symmetry. Name each wall ("LEFT side wall", "RIGHT side wall") and state the material for each one individually.

### 3. Interior features visible through windows need layered depth cues
When something should be visible *through* a window (not on the window wall itself), describe it as a spatial sequence:
1. First describe the wall facing the camera and its windows
2. Then say "looking through these windows into the room"
3. Then describe what's on the **opposite interior wall**, using terms like "far back wall," "furthest from camera," "the room's OPPOSITE wall"

Never describe interior features without anchoring which wall they're on relative to the camera. Nano Banana Pro defaults to placing prominent features on the camera-facing surface.

### 4. Doors and openings: state which face they are NOT on
If doors belong on the back wall, explicitly say "there are NO doors on this [camera-facing] face" before describing where the doors actually are. Nano Banana Pro tends to place doors on whatever wall is most visible. A negative constraint on the wrong wall is more reliable than a positive placement on the correct wall alone.

### 5. Avoid lighting cues that pull features forward
Phrases like "bright natural daylight pours through" or "backlighting the doors" can cause Nano Banana Pro to move those elements to the camera-facing side (where the light source would logically be). When describing features on a far interior wall, use neutral lighting language — "warmly lit," "visible deep inside" — rather than dramatic directional lighting that implies proximity to the viewer.

### 6. Remove context that doesn't appear in the camera view
If an element isn't visible from the specified camera angle (e.g., a dock on the south side when the camera faces north), remove it from the prompt entirely. Nano Banana Pro will try to include everything mentioned, often placing invisible elements in incorrect locations or distorting the composition to force them in.

### 7. "Spanning the full width" is dangerous for bounded sections
Avoid phrases like "spanning the full width and height" for glass or any material that should stop at a boundary. Nano Banana Pro interprets "full width" as edge-to-edge with no interruption. Instead, describe the material without the "full" qualifier and explicitly describe the termination point — a column, a wall element, a material change.

### 8. Hyper-specific visual descriptions for small features
Generic architectural terms ("French doors," "privacy windows") are often ignored or rendered generically. Describe the visual appearance: "two tall matching door panels with multiple rectangular glass panes in a traditional grid pattern" gives the model far more to work with than "French doors." The more visually specific the description, the more likely Nano Banana Pro renders it.

### 9. Use "the most prominent feature" to prioritize
If a detail keeps getting dropped, add a sentence declaring it "the most prominent feature" or "the dominant visual element" in that section. Nano Banana Pro uses attention weighting — calling something prominent tells the model to allocate rendering priority to it.

### 10. Break complex facades into step-by-step spatial instructions
Per Google's own Gemini image generation docs: for complex scenes with multiple elements, describe the composition as a spatial sequence rather than a simultaneous description. "First... then... finally..." or a numbered left-to-right reading order (as used in the B-series prompts) helps Gemini maintain correct spatial relationships.

---

> **Orientation key (looking at the plan from above, north at top):**
> - **North elevation** (long side, 61') — faces the open water/harbour. Shows: reception windows, outdoor lounge opening, indoor lounge windows, sauna windows.
> - **South elevation** (long side, 61') — faces the existing Heritage Harbour dock (entry/access side). Shows: reception French doors + entrance landing, solid changeroom wall, indoor lounge windows, solid service corridor wall.
> - **East elevation** (short side, 21') — sauna end. Shows: sauna floor-to-ceiling windows.
> - **West elevation** (short side, 21') — reception end. Shows: reception floor-to-ceiling windows.

---

## A. Architectural Drawings — CAD-Level

**A1. Top-Down Floor Plan**
> Upload `FloorPlan.png` as the reference image alongside this prompt.

```
The attached image is the EXACT floor plan layout you must replicate. Match the room positions, wall placement, room proportions, door locations, dimension lines, and labeling style precisely. Do not add rooms, do not move rooms, do not add furniture or fixtures. Copy the layout as closely as possible, then add the window markings described below.

A top-down orthographic floor plan of a floating sauna facility, 21 feet wide (north-south) by 61 feet long (west-east), on a white background. Use thin single-line walls only — not thick filled-black walls. Rooms are empty labeled boxes with no furniture, no fixtures, and no interior detail. Windows are marked on specific walls as described below using standard architectural window notation (thin parallel lines breaking the wall line). Show door swing arcs at every door opening as quarter-circle arcs.

The layout is NOT a simple left-to-right row. It is a 2D arrangement with rooms stacked north-south in the eastern half. North is at the top. Here is the exact layout:

The western third of the building has two rooms that each span the full 21-foot width:
1. RECEPTION (9.5 feet wide east-west, 15 feet deep north-south) at the far west end. Two outward-opening French doors on its south wall (two quarter-circle arcs swinging south into an area labeled "Ext." outside the building outline). Large window bays on the north wall.
2. CHANGEROOM (12 feet wide, 20 feet deep) immediately east of Reception. One door on its north wall swinging east into the outdoor lounge area. High narrow windows on both the north and south walls (privacy windows — placed high on the wall to let in natural light while maintaining privacy).

The center section is 17 feet wide east-west, split into two rooms stacked north-south:
3. OUTDOOR LOUNGE (17 by 6.5 feet) in the north half. Empty room, no pool drawn. One door on the dividing wall connects to the indoor lounge below.
4. INDOOR LOUNGE (17 by 13.5 feet) in the south half, directly below the outdoor lounge. Floor-to-ceiling windows on both the north and south walls. One large door arc from the changeroom/outdoor lounge partition. A double door (two small arcs) on its south wall leads to the hallway. A single door on its east wall leads to the hallway.

The eastern section (approximately 22 feet wide east-west) is also split north-south:
5. SAUNA (22 by 22.5 feet) — a large, nearly square room occupying the NORTH portion of this section. It fills from the north exterior wall downward. Floor-to-ceiling windows spanning the entire north wall and the entire east wall, meeting at the northeast corner in a seamless glass-to-glass joint with no column to maximize the view. Empty room, no benches or heaters drawn.
6. Three small service rooms in a horizontal row in the SOUTH portion, directly below the sauna, each approximately 6 feet deep north-south: REST ROOM 1 (5 by 6 feet) on the west, then REST ROOM 2 (9 by 6 feet) in the middle, then MAINTENANCE AND MECHANICAL (7.5 by 6 feet) on the east. Each has a door on its south wall swinging into the hallway (quarter-circle arcs).
7. HALLWAY — a narrow east-west corridor running along the very south edge of this section, south of the service rooms, connecting the indoor lounge on the west to the sauna area on the east. Medium-sized windows on the south exterior wall, placed high (from about 8 feet up to the roofline).

Dimension lines: 61 FEET across the top. 21 FEET on the left side. Along the bottom edge, a dimension chain showing: 12', 17', 5', 9', 8.5' (the reception width is not dimensioned at the bottom). Room names in clean monospace capitals centered in each room, with dimensions below (e.g., "Reception" then "9.5'x 15'" on the next line). Draw a compass rose showing N/S/E/W in the bottom-right corner. No scale bar. Thin black line work on white background, simple and clean. No shading, no hatching, no color.
```

**A2. North Elevation (Long Side — Harbour-Facing, 61')**

```
Draw a simple 2D flat elevation of ONLY the north face of a 61-foot-long, 15-foot-tall floating sauna building. This is a single flat wall viewed head-on from the north. Only draw what is physically visible on this one exterior face — do not show rooms that are behind other rooms.

The building sits on a floating dock hull with a waterline marked. The roof is a very LOW-PITCH gable — the walls rise approximately 12 feet, then a shallow roof peak adds only about 3 more feet to reach 15 feet total. The ridge runs east-west along the full length. From this north elevation, the low-angle roof slopes gently DOWN toward the viewer with a small overhang. The roof is NOT steep — it is almost flat with a subtle peak. The roof material is standing-seam dark zinc.

IMPORTANT: You are standing NORTH of the building looking SOUTH at its north face. East is on your LEFT, west is on your RIGHT.

Only 4 rooms touch the north exterior wall. The indoor lounge, hallway, restrooms, and maintenance room are all on the SOUTH side of the building — they do NOT appear on this north elevation at all. Do not draw them.

Reading left to right across this face (east to west), there are exactly 4 sections:
— 22 feet: floor-to-ceiling glass window panels with slim aluminum mullions spanning the entire width and full height of this section (SAUNA)
— 17 feet: an open gap — no wall here, just glass-and-steel railings at deck level, open to the air (OUTDOOR LOUNGE)
— 12 feet: solid wall of vertical thermally modified Scandinavian pine cladding, warm honey-brown, with two high narrow privacy windows (CHANGEROOM)
— 9.5 feet: large window bays spanning most of this section (RECEPTION)

These 4 sections sum to approximately 61 feet. There are no other sections — only these 4 rooms are visible on the north face.

Label each section with its room name below the dimension line. Draw a dimension chain along the bottom showing each section width (22, 17, 12, 9.5) summing to 61 feet. Draw one height dimension on the right side reading 15 FEET (to peak). Mark the waterline. No compass, no scale bar. Black line work on white background. No shading, no color, no perspective. Simple flat 2D architectural elevation.
```

**A3. South Elevation (Long Side — Dock-Facing, 61')**

```
Draw a simple 2D flat elevation of ONLY the south face of a 61-foot-long, 15-foot-tall floating sauna building. This is a single flat wall viewed head-on from the south. Only draw what is physically visible on this one exterior face — do not show rooms that are behind other rooms. This side faces the Heritage Harbour dock.

The building sits on a floating dock hull with a waterline marked. The roof is a very LOW-PITCH gable — the walls rise approximately 12 feet, then a shallow roof peak adds only about 3 more feet to reach 15 feet total. The ridge runs east-west along the full length. From this south elevation, the low-angle roof slopes gently DOWN toward the viewer with a small overhang. The roof is NOT steep — it is almost flat with a subtle peak. The roof material is standing-seam dark zinc.

IMPORTANT: You are standing SOUTH of the building looking NORTH at its south face. West is on your LEFT, east is on your RIGHT.

The outdoor lounge and sauna are on the NORTH side of the building — they do NOT appear on this south elevation. Do not draw them.

Reading left to right across this face (west to east), there are exactly 6 sections:
— 9.5 feet: RECEPTION — outward-opening French doors (the only entrance on this wall). The reception section is SET BACK from the main facade line, with a teak entrance landing platform extending forward toward the viewer at deck level, with a handrail. Solid thermally modified pine wall above the doors. The setback and landing should be clearly visible.
— 12 feet: CHANGEROOM — solid wall of vertical thermally modified Scandinavian pine cladding, warm honey-brown, with two high narrow privacy windows. No doors.
— 17 feet: INDOOR LOUNGE — floor-to-ceiling glass curtain wall panels with slim aluminum mullions. At the left edge (between changeroom and indoor lounge), the outdoor lounge deck edge is visible as a horizontal line at deck level above.
— 5 feet: solid thermally modified pine wall (RR1 behind)
— 9 feet: solid thermally modified pine wall (RR2 behind)
— 8.5 feet: solid thermally modified pine wall (MAINTENANCE & MECHANICAL behind)

The last three sections (5 + 9 + 8.5 = 22.5 feet) form a continuous solid pine wall for the hallway/service corridor with medium-sized high windows placed from about 8 feet up to the roofline.

Label each section. Draw a dimension chain along the bottom showing: 9.5' + 12' + 17' + 5' + 9' + 8.5' = 61'. Draw one height dimension on the right side reading 15 FT HEIGHT (TO PEAK). Mark the waterline and floating hull. No compass, no scale bar. Black line work on white background. No shading, no color, no perspective. Simple flat 2D architectural elevation.
```

**A4. East Elevation (Short Side — Sauna End, 21')**

```
Draw a simple 2D flat elevation of ONLY the east face of a 21-foot-wide, 15-foot-tall floating sauna building. This is a single flat wall viewed head-on from the east. Only draw what is physically visible on this one exterior face.

The building sits on a floating dock hull with a waterline marked. The roof is a very LOW-PITCH gable — from this east end, a shallow triangle is visible at the top. The walls rise approximately 12 feet straight up, then the roof adds only about 3 more feet of peak at the center, reaching 15 feet total. The slopes are very gentle (approximately 15-20 degrees), NOT a steep A-frame. The building should look mostly rectangular with a subtle roof peak on top. The roof material is standing-seam dark zinc. No chimney — the sauna uses an electric heater.

IMPORTANT: You are standing EAST of the building looking WEST at the east face. South is on your LEFT, north is on your RIGHT.

This face has two distinct sections, reading from LEFT to RIGHT:

— LEFT (south side), approximately 6 feet wide: SOLID thermally modified Scandinavian pine cladding, warm honey-brown. This is the hallway and service corridor behind — no large windows, only medium-sized high windows placed from about 8 feet up. This solid section takes up nearly a THIRD of the facade width — it is NOT a narrow strip.

— RIGHT (north side), approximately 15 feet wide: the sauna room with floor-to-ceiling GLASS window panels spanning this section, with slim aluminum mullions dividing the glass. At the far RIGHT edge (north corner), the glass goes all the way to the corner with no visible column — this is where it meets the north wall glass in a seamless butt joint. Include a small detail inset box in the upper right corner labeled "Detail A" showing a close-up of this seamless glass-to-glass corner joint. Through the glass, the tiered sauna benches on the south interior wall and a modern electric sauna heater are faintly visible.

Draw a dimension line showing 21 FEET width across the top. Draw a height dimension on the right reading 15 FEET. Mark the waterline. No doors on this face. No compass, no scale bar. Black line work on white background. No shading, no color, no perspective. Simple flat 2D architectural elevation. Square format.
```

**A5. West Elevation (Short Side — Reception End, 21')**

```
Draw a simple 2D flat elevation of ONLY the west face of a 21-foot-wide, 15-foot-tall floating sauna building. This is a single flat wall viewed head-on from the west. Only draw what is physically visible on this one exterior face.

The building sits on a floating dock hull with a waterline marked. The roof is a very LOW-PITCH gable — from this west end, a shallow triangle is visible at the top. The walls rise approximately 12 feet straight up, then the roof adds only about 3 more feet of peak at the center, reaching 15 feet total. The slopes are very gentle (approximately 15-20 degrees), NOT a steep A-frame. The building should look mostly rectangular with a subtle roof peak on top. The roof material is standing-seam dark zinc.

IMPORTANT: You are standing WEST of the building looking EAST at the west face. North is on your LEFT, south is on your RIGHT.

This face is the reception end, but it is NOT a single flat plane — the south portion is set back. Reading from LEFT to RIGHT:

— On the LEFT (north side), approximately 15 feet of the west wall is the reception room's west face. This section has moderately sized window bays (not floor-to-ceiling — standard reception windows with solid wall above and below), framed on the LEFT edge with a narrow strip of solid thermally modified Scandinavian pine cladding, warm honey-brown.

— On the RIGHT (south side), the remaining approximately 6 feet is SET BACK from the main reception wall. This is the exterior entrance landing — an OPEN-AIR teak deck platform at floor level with a simple handrail along the edge. There are NO WALLS enclosing this landing area — it is completely open on the west face and south face. The only things visible here are the deck platform, the handrail, and the roof overhang above (the roof extends over the landing to provide weather cover, but the space below is open air). The reception's south wall (with French doors, not visible from this angle) forms the back wall of the landing recess. Draw this as an open covered porch/landing, not an enclosed room.

Draw a dimension line showing 21 FEET width across the top. Draw a height dimension on the side reading 15 FEET. Mark the waterline. No compass, no scale bar. Black line work on white background. No shading, no color, no perspective. Simple flat 2D architectural elevation. Square format.
```

**A6. Cross-Section Through Sauna (East-West Cut)**

```
Draw a simple 2D architectural cross-section cutting east-west through the sauna room of a floating sauna building. The building is 21 feet wide and 15 feet tall from waterline to roof peak.

The section reveals from bottom to top: the floating dock hull sitting in water (water shown as a light blue horizontal band — the only color), a structural floor platform, the interior of the sauna room (22 by 22.5 feet, nearly square) with two tiers of L-shaped teak wood bench seating along the walls, two electric sauna heaters on the floor, a tongue-and-groove cedar plank ceiling, and a very low-pitch gable roof clad in standing-seam dark zinc — the walls are approximately 12 feet tall, and the roof peak adds only about 3 more feet at the center ridge, reaching 15 feet total. The slopes are gentle (15-20 degrees), not steep.

The north wall (left in section) is shown as a floor-to-ceiling glass panel. The south wall (right in section) shows the wall layers in section: thermally modified pine cladding, air gap, insulation, vapor barrier, interior cedar paneling — each with standard architectural hatching.

Draw dimension lines showing approximately 12-foot interior ceiling height and 15-foot waterline-to-roof-peak total. No compass, no scale bar. Black-and-white line work, purely orthographic, white background.
```

**A7. Exploded Axonometric Diagram**

```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

Draw an exploded axonometric diagram of a floating sauna facility. The building is 61 feet long (east-west) and only 21 feet wide (north-south), making it approximately 3 TIMES LONGER than it is wide. It MUST look like a long, narrow rectangular barge — not a compact square or a 2:1 rectangle. If the width is drawn as 1 unit, the length must be drawn as 3 units. The building is 15 feet tall to the roof peak. Show it separated into 5 visible layers floating apart vertically on a white background.

The isometric view should show the building from a position looking at the NORTH and EAST faces (so you can see the sauna glass corner).

LAYER 1 (bottom): the flat floating dock hull sitting on a light blue water surface. The hull is a long narrow rectangle (3:1 ratio).

LAYER 2: the structural floor deck and framing, matching the hull shape.

LAYER 3: interior partition walls showing the room layout matching the attached floor plan exactly. From west to east: Reception and Ext. landing area (far left), Changeroom, then the building splits north-south — Outdoor Lounge on the north half and Indoor Lounge on the south half (stacked), then the eastern section with the large Sauna on the north and the service rooms (Rest Room 1, Rest Room 2, Maintenance & Mechanical) and Hallway on the south. Label each room.

LAYER 4: the exterior cladding shell. This layer shows ONLY exterior walls, not interior partitions. CRITICAL: each face has a specific pattern of glass, solid wall, and openings. Most of the exterior is SOLID PINE, not glass. Glass appears in only a few specific sections. All solid wall sections are thermally modified Scandinavian pine cladding, warm honey-brown.

NORTH FACE (61 feet long, reading left-to-right as seen from the viewing angle, east to west):
1. SAUNA section (22 feet): floor-to-ceiling GLASS panels with slim aluminum mullions spanning the full width and full height. This is the ONLY large glass section on the north face.
2. OUTDOOR LOUNGE section (17 feet): NO WALL HERE AT ALL. This section is completely open to the air — no glass, no pine, no cladding. Show only glass-and-steel railings at deck level. The gap must be clearly visible as an opening, not a wall.
3. CHANGEROOM section (12 feet): SOLID thermally modified pine wall with NO large windows. Only two small, high, narrow privacy windows. The wall is mostly solid wood.
4. RECEPTION section (9.5 feet): pine-framed wall with window bays (mullioned windows set into the pine wall, NOT a full glass curtain wall — pine framing is visible around and between the windows).

EAST FACE (21 feet wide, short end):
Two distinct sections. The south portion (lower left from viewing angle), approximately 6 feet wide — nearly a THIRD of the facade — is SOLID thermally modified pine cladding for the hallway and service corridor behind, with only medium-sized high windows. This is NOT a narrow strip. The north portion, approximately 15 feet wide, is sauna floor-to-ceiling GLASS panels with slim aluminum mullions, meeting the north face glass at the northeast corner in a seamless glass-to-glass butt joint with no column. No doors.

SOUTH FACE (61 feet long, reading left-to-right as seen from the viewing angle, east to west):
1. HALLWAY / SERVICE CORRIDOR section (22.5 feet): SOLID thermally modified pine wall — NO large windows, NO glass curtain wall. Only medium-sized high windows placed from about 8 feet up. This section covers Rest Room 1, Rest Room 2, Maintenance & Mechanical, and the hallway. It is mostly SOLID PINE.
2. INDOOR LOUNGE section (17 feet): floor-to-ceiling GLASS curtain wall panels with slim aluminum mullions. This is the ONLY large glass section on the south face.
3. CHANGEROOM section (12 feet): SOLID thermally modified pine wall — NO large windows. Only two small high narrow privacy windows. Mostly solid wood.
4. RECEPTION section (9.5 feet): outward-opening French doors (the building entrance) set into a solid pine wall. The reception is set back slightly with a teak entrance landing.

WEST FACE (21 feet wide, short end):
Two distinct sections. The north portion (right side from viewing angle), approximately 15 feet wide, is the reception room's west face — pine-framed wall with moderately sized window bays (mullioned windows set into pine framing, NOT a full glass curtain wall, NOT floor-to-ceiling). Narrow strip of solid pine at the north corner. The south portion (left side from viewing angle), approximately 6 feet wide, is SET BACK from the reception wall — this is the exterior entrance landing, an OPEN-AIR teak deck platform with a handrail. There are NO WALLS enclosing this landing — it is open on the west and south sides, with only the roof overhanging above for weather cover. No doors visible on this face.

LAYER 5 (top): the roof — a very LOW-PITCH gable (not flat, not steep). The ridge runs east-west along the full 61-foot length. The walls are approximately 12 feet tall, and the shallow roof peak adds only about 3 more feet at the center to reach 15 feet total. Standing-seam dark zinc material. The roof shape should be a long narrow rectangle with a very subtle peaked ridge, not a flat slab.

Color fills: warm honey-brown for pine cladding, blue-green tint for glass panels ONLY where specified above, light blue for water, matte dark grey for the zinc roof, white for interior walls. Do NOT color any section as glass unless it is explicitly listed as glass above. Annotation labels on leader lines pointing to each layer and key features. No compass, no scale bar. Clean and minimal line work.
```

---

## B. Photorealistic Exterior & Interior

### B1. Hero Shot - Golden Hour on the Harbour
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a modern floating sauna facility at Heritage Harbour in Vancouver, Canada, floating on calm harbour water at golden hour, captured in the style of professional editorial architectural photography. CRITICAL PROPORTION: the building is 61 feet long and only 21 feet wide — it is THREE TIMES longer than it is wide, a long narrow barge shape, NOT a compact square. It is 15 feet tall. It floats on the glassy harbour water on a floating dock base, with its full reflection mirrored perfectly below. The exterior is clad in thermally modified Scandinavian pine with a warm honey-brown tone, visible natural grain, and a matte oil finish. A low-pitch gable roof of standing-seam dark zinc runs the full 61-foot length, with the ridge running east-west — the walls rise approximately 12 feet and the shallow roof peak adds about 3 more feet to 15 feet total, giving a mostly rectangular profile with a subtle peaked ridge.

The camera is positioned on the water directly north of the building, looking STRAIGHT south at the harbour-facing north elevation — a perfectly flat, head-on, frontal view with NO perspective angle, NO three-quarter view, and NO visible side walls. The building should appear as a flat rectangle across the frame, like an architectural elevation photograph. There are exactly 4 distinct sections reading left to right (east to west), and each must be clearly visible and correctly proportioned:

1. SAUNA (22 feet wide, leftmost): floor-to-ceiling glass panels with slim aluminum mullions on the north face (facing camera) and east face (left side). At the far left corner (northeast corner), the glass wraps around in a seamless glass-to-glass joint with no column. IMPORTANT: the glass on the north face STOPS at the right edge of this 22-foot section, where it meets a thick SOLID thermally modified pine wall that divides the sauna from the outdoor lounge. This dividing wall is clearly visible as a vertical honey-brown pine column/wall element at the right edge of the glass — the glass does NOT extend past this wall. Through the glass, the interior is visible: two tiers of L-shaped teak bench seating run along the right wall (west, solid pine) and continue along the back wall (south, also solid pine), forming a continuous L-shaped seating arrangement in the far-right and back of the room. Electric sauna heaters with stones sit in the back-left corner (southeast), near the glass. Deep amber light radiates from within.

2. OUTDOOR LOUNGE (17 feet wide): the north facade has no wall here, just glass-and-steel railings at deck level, with weathered silver-gray teak decking and a small turquoise plunge pool glowing from underwater lights. The LEFT side wall (dividing this deck from the sauna) and the RIGHT side wall (dividing this deck from the changeroom) are both SOLID thermally modified pine — there are NO windows or glass on either side wall of the outdoor lounge. The sauna's floor-to-ceiling glass stops at the sauna's own 22-foot section and does NOT continue into the outdoor lounge. IMPORTANT: directly behind this open deck, forming its back wall (south side), is the indoor lounge's floor-to-ceiling glass north wall — a brightly lit glass curtain wall with slim aluminum mullions revealing warm interior lighting and modern Scandinavian furniture inside. This is NOT a see-through gap to the water beyond; the enclosed indoor lounge fills the space behind the outdoor deck.

3. CHANGEROOM (12 feet wide): a prominent SOLID wall section of thermally modified pine cladding — NO large windows, NO glass. Only two small, high, narrow privacy windows near the roofline. This section must read as a clear solid break between the open outdoor lounge and the reception windows.

4. RECEPTION (9.5 feet wide, rightmost): moderately sized window bays set into pine framing — SMALLER than the sauna's floor-to-ceiling glass, with visible pine wall above and below each window. Not a glass curtain wall. There are NO doors on this north-facing wall. The reception interior glows warmly, and through the windows you can see a pair of tall white-framed French doors with rectangular glass panes in a grid pattern standing against the far back wall inside the room. These interior French doors are brightly lit and clearly visible through the reception windows.

The scene is illuminated by low golden sunlight at 15 degrees above the horizon, casting long warm shadows and grazing light across the timber texture. The Vancouver skyline and North Shore mountains are visible across the harbour under a sky grading from warm peach to deep blue. Shot from water level, 10 meters away, with a 24mm tilt-shift lens at f/9, creating a serene and aspirational atmosphere. No text, no watermarks.
```

### B2. Aerial Drone View - Summer Day
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a modern floating sauna facility at Heritage Harbour in Vancouver, Canada, floating on deep blue harbour water and attached to an existing dock on the south side in bright midsummer daylight, captured from a drone at approximately 15 meters altitude looking down at a 45-degree angle from the northwest. CRITICAL PROPORTION: the structure is 61 feet long and only 21 feet wide — THREE TIMES longer than it is wide, a long narrow barge shape. From this aerial perspective the layout reads clearly from west to east with these exact proportions:

1. RECEPTION (9.5 feet, far left): low-pitch dark zinc gable roof over moderately sized window bays set into pine framing (NOT large glass), with the open-air teak entrance landing visible on the south side set back from the main facade.

2. CHANGEROOM (12 feet): solid dark zinc roof over solid thermally modified pine walls — NO large windows, only small high privacy windows. This section must be clearly visible as a solid break.

3. OUTDOOR LOUNGE (17 feet, center): open to the sky — NO roof over this section. Weathered silver-gray teak deck with a small turquoise plunge pool sparkling in the sun and two oiled teak lounge chairs with white cushions. Directly behind (south of) this open deck, the indoor lounge's floor-to-ceiling glass north wall is clearly visible from above, with warm interior lighting and Scandinavian furniture inside — the outdoor deck is NOT an empty gap through the building.

4. SAUNA (22 feet, far right — the largest section): dark zinc roof over floor-to-ceiling glass clearly visible on the north and east faces, wrapping around the northeast corner in a seamless glass-to-glass joint with no column.

Two people in white robes recline on the open deck. The warm honey-brown thermally modified pine cladding with visible natural grain contrasts with the deep navy blue harbour water showing gentle wind ripples that catch sunlight. Other harbour vessels, docks, and the Vancouver waterfront fill the background. The scene is bright and vivid with saturated summer colors and hard even daylight. No text, no watermarks.
```

### B3. Winter Scene - Snow and Steam
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a modern floating sauna building in a dramatic Pacific Northwest winter scene at blue-hour twilight at Heritage Harbour in Vancouver, in the style of cinematic architectural photography. CRITICAL PROPORTION: the building is 61 feet long and only 21 feet wide — THREE TIMES longer than it is wide, a long narrow barge shape. It is 15 feet tall. It floats on dark still harbour water with thin ice forming at the edges, attached to the existing dock on the south side with dark open water visible around its floating dock hull. Fresh powder snow covers the low-pitch gable roof of dark zinc standing-seam panels — the ridge runs the full 61-foot length east-west with gentle slopes — and blankets the surrounding docks and harbour infrastructure.

The camera faces the north elevation from the adjacent dock at eye level, 12 meters away. Reading left to right (east to west), there are exactly 4 distinct sections:

1. SAUNA (22 feet, leftmost — the largest section): floor-to-ceiling glass panels on its north and east faces, wrapping around the northeast corner in a seamless glass-to-glass joint with no column. Through the glass: the right wall (west) and back wall (south) inside are SOLID pine with two tiers of teak bench seating running along them. Electric sauna heaters with stones sit in the back-left corner (southeast). A door in the back-right corner (southwest) connects to the indoor lounge. Deep amber light radiates from within. Snow accumulates on the roof above.

2. OUTDOOR LOUNGE (17 feet): no wall on the north face, just glass-and-steel railings. Thick clouds of steam rise dramatically from the heated turquoise plunge pool on the teak deck as it meets the freezing air, backlit warmly by interior glow. Behind the open deck, the indoor lounge's floor-to-ceiling glass north wall glows with bright warm amber-orange light from interior fixtures, revealing Scandinavian furniture inside.

3. CHANGEROOM (12 feet): a prominent SOLID section of thermally modified pine cladding covered in delicate frost — NO large windows, only two small high privacy windows near the roofline. This solid wall section must read as a clear dark break between the glowing lounge and reception.

4. RECEPTION (9.5 feet, rightmost): moderately sized window bays set into pine framing (SMALLER than the sauna glass, with pine wall visible above and below). Through the windows, double outward-opening French doors are visible on the back wall (south face), leading to the exterior entrance landing. A soft warm glow inside.

The harbour surface shows thin ice patterns with dark patches near the warm structure. The sky is deep purple-blue with faint stars emerging. Shot with a 35mm lens at f/4, creating a moody atmosphere of cozy warmth against harsh Pacific Northwest winter. No text.
```

### B4. Interior - Outdoor Lounge and Plunge Pool
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict references for the spatial layout, room dimensions, wall sections, and window placement. The outdoor lounge is the open-air gap between the changeroom and sauna roof sections — match this precisely.

A photo of the outdoor lounge deck of a modern floating sauna facility, captured from a standing position on the open-air teak deck looking east toward a small cold plunge pool. The deck is 17 feet wide and 6.5 feet deep, completely open to the sky — no roof overhead, just open air between the changeroom roof section to the west and the sauna roof section to the east. The rectangular pool (4 feet by 8 feet) is filled with crystal-clear turquoise water and set flush into the deck surface of weathered teak boards showing a natural silver-gray patina with visible grain pattern. Beyond the pool on the north side, minimal glass-and-steel railings open to a panoramic view of calm Heritage Harbour waters with the Vancouver waterfront and North Shore mountains in the distance. Two modern Scandinavian teak lounge chairs with natural white linen cushions are positioned beside the pool. Through a door to the south, the indoor lounge room is visible below with natural light pouring through its floor-to-ceiling south-facing glass curtain wall. To the east, the warm honey-brown thermally modified pine wall of the enclosed sauna wing rises with its smooth heat-treated surface, its floor-to-ceiling glass wrapping around the northeast corner. Bright, low-angle afternoon golden sunlight pours in unobstructed from the open sky, casting warm light across every teak grain and creating sharp shadows from the railings on the deck. Captured with a 20mm wide-angle lens at f/8, at standing eye level, emphasizing the luxurious textures of oiled teak and the contrast between the intimate deck space and the vast harbour panorama. No text, no watermarks.
```

### B5. Interior - Sauna Room
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict references for the spatial layout, room dimensions, and window placement. The sauna has floor-to-ceiling glass on its north and east walls meeting at the northeast corner in a seamless glass-to-glass joint — match this precisely. The south and west walls are solid.

A photo of the sauna room inside a modern luxury floating sauna, captured from a low seated position on the lower bench in the southwest corner looking northeast across the room toward the glass corner. The space is 22 feet wide (east-west) and 22.5 feet long (north-south), with two tiers of L-shaped bench seating made of smooth heat-treated teak displaying a rich deep amber patina with visible grain pattern, running along the south and west walls (the solid interior walls). On the floor sit two modern electric sauna heaters with stones on top glowing with intense radiant heat, with visible wisps of steam rising. The south and west walls and ceiling are clad in smooth tongue-and-groove Western red cedar planks with a warm golden tone and subtle natural knot patterns. The north and east walls are entirely floor-to-ceiling glass panels with slim aluminum mullions framing panoramic views of the harbour water outside — at the northeast corner, the glass meets in a seamless glass-to-glass butt joint with no structural column, creating an uninterrupted wraparound panoramic view of the harbour and mountains, a cool visual counterpoint to the warm interior. The room is illuminated by warm ambient light from recessed LED strips hidden under the upper bench tier, casting pools of golden light on the cedar surfaces. The air is gently hazy with light steam that softens and diffuses the light. A traditional Finnish wooden sauna bucket and birch ladle rest on the lower bench. The image should feel warm, enveloping, meditative, and luxurious, captured with a 16mm wide-angle lens at f/5.6 from bench level, in the style of professional wellness architecture photography. No text.
```

### B6. Night Scene - Floating Light
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a modern floating sauna facility at night on a perfectly still, dark Heritage Harbour waters, in the style of long-exposure architectural photography. CRITICAL PROPORTION: the building is 61 feet long and only 21 feet wide — THREE TIMES longer than it is wide, a long narrow barge shape. It is 15 feet tall. It floats on the black harbour water, its warm Scandinavian pine cladding nearly invisible in the darkness so that the structure is defined entirely by its bright interior lighting.

The camera faces the north elevation from water level. Reading left to right (east to west), there are exactly 4 distinct sections:

1. SAUNA (22 feet, leftmost — the largest and most dramatic light source): deep amber-orange light radiates through floor-to-ceiling glass panels on its north and east faces. At the far left corner (northeast), the glass wraps around the corner in a seamless glass-to-glass joint with no column — you can see the glass continuing around the building's east face, glowing intensely. Through the glass: the right wall (west) and back wall (south) inside are SOLID pine with two tiers of teak bench seating along them, lit by warm amber light. Electric sauna heaters glow in the back-left corner (southeast). A door in the back-right corner (southwest) connects to the indoor lounge.

2. OUTDOOR LOUNGE (17 feet): subtle recessed deck lights and a turquoise plunge pool lit from underwater. The north facade has no wall here, just glass-and-steel railings. Directly behind this open deck, the indoor lounge's brightly lit floor-to-ceiling glass north wall is visible, warmly illuminated with Scandinavian furniture inside (this is NOT a see-through gap to the water beyond).

3. CHANGEROOM (12 feet): a dark void — the solid thermally modified pine cladding is nearly invisible in the darkness, with only faint light escaping through two small high privacy windows near the roofline. This dark solid section must read as a clear break between the glowing lounge and reception.

4. RECEPTION (9.5 feet, rightmost): a warm soft glow through moderately sized window bays set into pine framing — SMALLER than the sauna glass, with dark pine wall visible above and below each window. Through the windows, double outward-opening French doors are faintly visible on the back wall (south face), leading to the exterior entrance landing.

Every warm light creates a perfect mirror reflection stretching down into the absolutely still black water, doubling the luminous composition. The low-pitch gable roof is a dark silhouette against the sky. The sky above is clear and filled with thousands of visible stars with the faint band of the Milky Way, and a thin crescent moon hangs low on the horizon. There are no other light sources — pure wilderness darkness surrounds the single glowing structure. The image is captured from water level, 15 meters away, with a 24mm lens at f/2.8 in a 15-second exposure. The mood is magical, serene, and luxuriously isolated. No text.
```

---

---

## C. Conceptual / Marketing

### C1. Lifestyle - Couple Enjoying the Experience
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict references for the building's exterior appearance, window placement, wall sections, and material transitions. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a couple in their early 30s enjoying a luxury floating sauna experience, captured in warm editorial lifestyle photography style. They are seated on the edge of a small turquoise plunge pool set into the weathered silver-gray teak deck of a modern floating sauna building, their feet dangling in the cool water. Both wear simple white linen robes and look relaxed and happy, gazing out at a calm Heritage Harbour waters surrounded by the harbour and Vancouver skyline in warm golden late-afternoon light. Behind them, the building is visible in soft focus — the solid thermally modified pine wall of the changeroom section and, further back, the indoor lounge's floor-to-ceiling glass curtain wall with warm interior light glowing through. The building is a long narrow barge (61 feet long, 21 feet wide). A wooden tray with two glasses of sparkling water sits on the deck beside them. The image is shot with an 85mm lens at f/2.8, creating a shallow depth of field where the couple is sharp against a soft golden bokeh of the harbour and forest beyond. The mood is serene, intimate, and aspirational, in the style of a Kinfolk or Cereal magazine editorial spread. No text, no watermarks.
```

### C3. Material Palette Mood Board
```
A photo of a flat-lay material mood board for a modern floating sauna building, arranged on a clean white Carrara marble surface in an organized grid with generous spacing. The material samples include: a square piece of thermally modified Scandinavian pine showing warm honey-brown color with visible natural grain and a smooth matte oil finish, a rectangle of oiled honey-toned teak decking with visible grain pattern and a warm sheen, a strip of smooth Western red cedar tongue-and-groove paneling with its golden tone and subtle knots, a cluster of dark smooth river rocks as used in a Finnish sauna heater, a swatch of matte black powder-coated steel flat bar, a small folded piece of standing-seam dark zinc roofing showing the seam profile, and a small square of low-E architectural glass with a subtle blue-green tint. Each sample casts a soft natural shadow on the marble. Small minimalist labels in a clean sans-serif font sit beside each sample. The image is captured from directly above in bright, even, diffused natural daylight with no harsh shadows. The style is a professional materials presentation for an architecture studio portfolio — clean, minimal, sophisticated. No watermarks.
```

---

---

## D. Special Angles and Details

### D1. Three-Quarter View - Approaching from the Dock
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a modern floating sauna facility at Heritage Harbour in Vancouver, on calm harbour water, attached to the existing dock on the south side, captured from the Heritage Harbour dock approaching from the southwest at a three-quarter angle that reveals both the short west end (reception entrance) and the full long south facade simultaneously. CRITICAL PROPORTION: the building is 61 feet long and only 21 feet wide — THREE TIMES longer than it is wide, a long narrow barge shape. It is 15 feet tall.

On the near short west end (21 feet wide), the building reveals two distinct zones: the north portion (about 15 feet) shows the reception's moderately sized window bays set into warm honey-brown pine framing (NOT floor-to-ceiling glass), and the south portion (about 6 feet) is set back to form the open-air teak entrance landing with a simple handrail — open to the air with only the roof overhang above for weather cover.

Along the long south facade (61 feet), the building's program reads clearly from left to right (west to east) with these exact proportions:

1. RECEPTION (9.5 feet): outward-opening French doors leading to the entrance landing, set back from the main facade line.

2. CHANGEROOM (12 feet): SOLID thermally modified pine wall — NO large windows, only two small high privacy windows. Must read as a clear solid break.

3. INDOOR LOUNGE (17 feet): floor-to-ceiling glass curtain wall with slim aluminum mullions revealing Scandinavian modern furniture and warm interior light. This is the ONLY large glass section on the south face.

4. SERVICE CORRIDOR (22.5 feet — the longest section): continuous SOLID thermally modified pine wall stretching to the far east end with only medium-sized high windows near the roofline. NO large glass.

The low-pitch gable roof of dark zinc standing-seam panels runs the full length with a subtle peaked ridge. Late afternoon sunlight hits the building at a low grazing angle, dramatically emphasizing the warm timber texture and casting the pine cladding in rich golden tones. The weathered wooden planks of the Heritage Harbour dock are visible in the foreground, slightly soft in focus. Dark calm harbour water with gentle golden-lit ripples surrounds the scene, and the North Shore mountains and harbour infrastructure fill the background in soft focus. Shot with a 35mm lens at f/4, creating an atmospheric and inviting editorial travel photograph. No text.
```

### D2. Detail - Steam Rising from Sauna Stones
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as references for the sauna room's interior layout — glass walls on north and east, solid cedar-clad walls on south and west, teak bench seating along the solid walls.

An extreme close-up photo of an electric sauna heater inside a luxury floating sauna room, captured at bench level with shallow depth of field. The heater is a modern electric sauna heater with stones piled on top, radiating intense heat that creates visible air shimmer above. A thin stream of water is being poured from a traditional birchwood ladle onto the top stones, producing an explosive burst of white steam that billows upward and catches warm amber light emanating from between the rocks below. Individual water droplets sizzle and bounce on the searing stone surfaces, frozen mid-motion. Behind the heater, out of focus in warm golden bokeh, the smooth teak bench seating and cedar wall paneling of the sauna room are faintly visible. Dramatic underlighting from the glowing rocks creates deep shadows and bright specular highlights on each stone's polished surface. Shot with an 85mm lens at f/2.8, the focus razor-sharp on the moment of contact between water and stone. The mood is primal, elemental, and meditative. No text.
```

### D3. Dusk Wide Shot - Wilderness Context
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a modern floating sauna facility at dusk, composed as an ultra-wide landscape that places the building as a small luminous element within a vast Pacific Northwest harbour panorama. CRITICAL PROPORTION: the building is 61 feet long and only 21 feet wide — THREE TIMES longer than it is wide, reading as a long narrow horizontal bar, not a compact shape. It sits in the lower third of the frame, floating in the vast waters of Heritage Harbour in Vancouver stretching to distant blue-grey mountain ridgelines on the horizon. From this distance the building reads as a thin, elongated horizontal band with a subtle low-pitch gable ridge, defined by warm amber light from the sauna's floor-to-ceiling glass corner at the east end, the indoor lounge glass curtain wall in the center, and the faint glow of the reception window bays at the west end, all suspended above a perfect mirror reflection in the perfectly still dark water. The sky fills the upper two-thirds of the image with a dramatic gradient from deep indigo overhead to burnt orange at the horizon, layered with thin altocumulus clouds catching the last pink and gold light of the day. Dark silhouettes of pine-forested islands break up the middle distance. The composition emphasizes the powerful contrast of scale: one small warm human refuge in immense wild nature. Captured from a distant shoreline approximately 60 meters away with a 16mm ultra-wide lens at f/8, in the style of fine art landscape and architectural photography. The mood is contemplative, majestic, and quietly luxurious. No text.
```

### D4. Isometric 3D Model View
```
The attached images are architectural elevation drawings and a floor plan of the EXACT building described below. Use them as strict dimensional and layout references — match the window placement, wall sections (solid pine vs. glass), room proportions, roof profile, and material transitions shown in these drawings precisely. Do not invent windows, doors, or openings that are not shown in the reference drawings.

A photo of a physical architectural scale model (maquette) of a modern floating sauna facility, placed on a clean white studio table with soft even studio lighting. CRITICAL PROPORTION: the model is 21 feet wide by 61 feet long — THREE TIMES longer than it is wide, a long narrow barge shape. At approximately 1:50 scale in an isometric three-quarter view showing the north and east faces (to reveal the sauna glass corner).

The model is meticulously crafted. Reading along the north face from east to west, 4 distinct sections are visible:

1. SAUNA (22 feet — the largest section): clear acrylic representing floor-to-ceiling glass on the north and east walls, wrapping around the northeast corner in a seamless joint with no column. Through the transparent glass: miniature two-tier L-shaped teak bench seating along the west and south walls (both solid pine), tiny electric sauna heaters in the southeast corner, and a tiny door in the southwest corner connecting to the indoor lounge.

2. OUTDOOR LOUNGE (17 feet): open gap between the changeroom and sauna rooflines — no roof over this section. Small rectangular pool insert tinted turquoise with tiny teak deck boards and glass-and-steel railings along the north edge. Behind this open deck, the indoor lounge's clear acrylic north wall is visible with tiny furniture inside.

3. CHANGEROOM (12 feet): entirely pine veneer — SOLID, no large windows, only tiny high windows. Must read as a clear solid break.

4. RECEPTION (9.5 feet): pine-framed wall with moderately sized window bays (SMALLER than the sauna glass). Tiny double French doors visible on the south wall leading to the open-air exterior entrance landing.

Most exterior walls are clad in real thermally modified Scandinavian pine veneer strips in warm honey-brown. The low-pitch gable roof is dark zinc-colored card with a subtle peaked ridge running the full length. The floating dock hull base sits on a sheet of reflective blue acrylic representing water. The solid hallway/service corridor walls on the south face are entirely pine-clad with only tiny high windows. The lighting is bright and even from above, casting minimal shadows. The style is a professional architecture studio model photograph with a clean white background and shallow depth of field softening the edges. No text, no watermarks.
```

---

---

## Usage Tips

- Write **narrative sentences, not keyword lists** — Gemini's strength is language understanding
- Set **resolution to "4K" (uppercase K)** via API — default is only 1K
- Enable **thinking mode** (`thinking_level="High"`) for complex architectural compositions
- **Upload the floor plan as a reference image** alongside the text prompt for best spatial accuracy
- Use **inline negative prompting**: "no text, no watermarks," "avoid curtains"
- **No transparent background support** — request white backgrounds for compositing
- The model **adopts the aspect ratio of the last uploaded image** — use this to your advantage
- Expect **503 errors during peak hours** (9AM-5PM PT); paid tier allows ~2-3 Nano Banana Pro images/day

### General
- **Tactile material descriptions** dramatically outperform generic ones: "thermally modified Scandinavian pine cladding with warm honey-brown tone, visible natural grain, and matte oil finish" >> "wood cladding"
- Always specify: subject, materials with texture detail, camera angle + lens + aperture, lighting direction and color temperature, mood, and style reference
- For **multi-turn consistency**, keep the same chat session (GPT Image 1.5) or re-upload reference images (Nano Banana Pro)
- **Neither platform produces dimensionally accurate output** — these are for design communication and marketing, not construction documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bottobot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bottobot)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
