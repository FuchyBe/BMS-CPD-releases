# BMS-CPD graphics file format (`.cpd`) — version 1

This document specifies the text format that BMS-CPD reads to draw tactical
graphics on the TSD (Tactical Situation Display) of the tablet: lines, zones,
circles, orbits (racetracks), points and free text. It is meant for mission
planners, scripts and third-party applications that want to produce graphics
for BMS-CPD.

- Status: **stable, version 1** (BMS-CPD 0.9.8 and later).
- Audience: anyone writing or generating `.cpd` files.
- A file can be written by hand in any text editor.

Contents: 1. Overview · 2. File rules · 3. Statements · 4. Positions ·
5. Attributes · 6. Item reference · 7. Limits · 8. Errors and warnings ·
9. How BMS-CPD uses a file · 10. Complete example · 11. Writing a generator ·
12. JSON form (parser output) · 13. Versioning

---

## 1. Overview

A `.cpd` file is plain UTF-8 text. One **statement** describes one graphic:

```
line  "FLOT"  color=red width=3 dash=dash
      N38 05.0 E126 40.0 ; N38 12.0 E127 30.0 ; N38 20.0 E128 10.0
```

A statement starts with a **keyword** at the start of a line (`line`), an
optional **name** in double quotes, then **attributes** (`key=value`) and
**positions**, in any order. A statement continues on the following lines as
long as they start with a space or a tab.

The whole file is one **layer** on the TSD: the pilot shows, hides, shares or
deletes it as a unit.

## 2. File rules

| Rule | Detail |
| --- | --- |
| Encoding | UTF-8 (a UTF-8 BOM is accepted and ignored). Line ends LF or CRLF. |
| File extension | `.cpd` (MIME type `text/plain`). |
| Comments | `#` up to the end of the line, except inside double quotes. |
| Blank lines | Ignored. |
| Keywords and attribute names | Case-insensitive (`LINE`, `Line`, `line`). |
| Statement start | A line whose first character is not a space, a tab or `#`. |
| Continuation | A line starting with a space or a tab continues the previous statement. |
| Names and text | Between double quotes. `\"` is a quote, `\\` a backslash inside quotes. |
| Separators | Spaces between words; `;` between positions; `->` between the two ends of an orbit. |
| Numbers | Decimal point `.` (never a comma). |

The first statement may be a `cpd` version line (recommended for generated
files):

```
cpd 1
```

## 3. Statements

| Keyword | Draws | Positions |
| --- | --- | --- |
| `layer` | nothing: names the layer and sets default attributes | none |
| `point` | a symbol with a label | exactly 1 |
| `line` | an open polyline | 2 to 200 |
| `zone` | a closed polygon (closed automatically) | 3 to 200 |
| `circle` | a circle | 1 (centre), with `radius=` |
| `orbit` | a racetrack (CAP, tanker track, holding) | 2, written `A -> B` |
| `text` | free text on the map | exactly 1 |
| `cpd` | nothing: format version (`cpd 1`) | none |

General form:

```
<keyword> ["Name"] [attribute=value ...] [position ; position ; ...]
```

### `layer`

```
layer "Package VIPER"  color=cyan width=2
```

- The name is the layer name shown in BMS-CPD. Without a `layer` statement,
  the layer takes the file name.
- Attributes on `layer` become the **defaults** for every item **after** it.
  A later `layer` statement replaces the defaults (and the name if the file
  had none yet). Most files have a single `layer` line at the top.

## 4. Positions

Three notations, which can be mixed in one file and even in one statement.

| Notation | Example | Meaning |
| --- | --- | --- |
| Decimal degrees | `37.0906 127.0297` | latitude then longitude, negative = south / west |
| Degrees and minutes (DED style) | `N37 05.44 E127 01.78` | hemisphere letter, degrees, decimal minutes |
| Degrees, minutes, seconds | `N37 05 26.4 E127 01 46.8` | hemisphere letter, degrees, minutes, decimal seconds |
| Bullseye | `BULL 045/32` | bearing (degrees) / range (NM) from the bullseye |

Details:

- **Hemisphere letters**: `N` `S` for latitude, `E` `W` for longitude. The
  letter may be glued to the number (`N37`) or separated (`N 37`).
- **Ranges**: latitude −90…90, longitude −180…180, minutes and seconds 0…<60.
- **Bullseye**: `BULL bearing/range`; the bearing is in degrees from the
  theater grid north (the same reference as the bullseye readouts of the CPD
  TSD), 0…360; the range is in nautical miles, ≥ 0. `BULLS` and `BE` are
  accepted for `BULL`. A bullseye position is resolved on the tablet with the
  bullseye of the current BMS mission: if BMS gives no bullseye, the items
  that use it are not drawn.
- **Separating positions**: `;` between positions of a `line` or a `zone`
  (`P1 ; P2 ; P3`), `->` between the two ends of an `orbit` (`A -> B`).
  Positions may run over several continuation lines; a `;` at the end or the
  start of a line is allowed.

## 5. Attributes

Written `key=value`, without spaces around `=`. A value containing spaces
goes between double quotes. Unknown attributes produce a **warning** and are
ignored (forward compatibility).

### 5.1 Style

| Attribute | Values | Default | Applies to |
| --- | --- | --- | --- |
| `color` | a colour (see 5.4) | from `side`, else `white` | all |
| `width` | stroke width in pixels, 1 to 8 (decimals allowed) | `2.5` | point, line, zone, circle, orbit |
| `dash` | `solid`, `dash`, `dot` | `solid` | line, zone, circle, orbit |
| `fill` | `none`, or a colour, optionally `:opacity` in % (`amber:20`, `#FF0000:35`) | `none` | zone, circle, orbit |
| `size` | text size in pixels, 10 to 40 | `18` (text: `20`) | all labels, text |
| `label` | `on`, `off` (show the name on the map) | `on` | point, line, zone, circle, orbit |
| `align` | `left`, `center`, `right` | `center` | text |
| `box` | `on`, `off`: dark backing behind the text | `off` | text |

When `fill` has a colour without opacity, the opacity is 25 %.

### 5.2 Meaning

| Attribute | Values | Effect |
| --- | --- | --- |
| `side` | `friend`, `hostile`, `neutral`, `unknown` | default colour: friend cyan, hostile red, neutral green, unknown yellow |
| `symbol` | `dot`, `circle`, `square`, `triangle`, `diamond`, `cross`, `star`, `ip`, `target`, `wp` | point symbol (default `dot`) |
| `alt` | `FL250`, `25000`, `FL250-FL300`, `0-15000`, `SFC-FL100` | altitude or altitude block, shown under the name (feet or flight levels; `SFC` = surface) |
| `note` | a text in quotes | free note, shown in the item details (not on the map) |

### 5.3 Geometry

| Attribute | Values | Applies to |
| --- | --- | --- |
| `radius` | distance, NM by default: `15`, `15nm`, `28km`, `9000m`, `30000ft` | circle (required) |
| `center` | a position (`center=BULL 035/52`, or `center="N37 05.4 E127 01.8"`) | circle, alternative to a bare position |
| `width` with a unit | racetrack width (distance between the two legs): `8nm`, `15km` | orbit |
| `turn` | `right`, `left` (or `R`, `L`) | orbit: side of the turns, seen from A towards B; default `right` |
| `at` | a position, same as a bare position | text, point |

On an `orbit`, `width=` **with a distance unit** (`nm`, `km`, `m`, `ft`) is
the racetrack width; **without a unit** it is the stroke width. Default
racetrack width: 5 NM.

### 5.4 Colours

Either `#RRGGBB` (hexadecimal, `#RGB` accepted) or a name:

| Name | Value | | Name | Value |
| --- | --- | --- | --- | --- |
| `red` | `#FF3B3B` | | `blue` | `#4D8DFF` |
| `amber` | `#FFB000` | | `magenta` | `#FF5CF0` |
| `orange` | `#FF8A1F` | | `white` | `#F2F2F2` |
| `yellow` | `#FFE14D` | | `gray` / `grey` | `#9AA5A0` |
| `green` | `#00FF66` | | `black` | `#000000` |
| `cyan` | `#2EE6FF` | | | |

The named colours are the CPD palette; prefer them so that graphics match
the rest of the display.

## 6. Item reference

### `point`

```
point "IP ALPHA"  N37 30.0 E127 05.0  symbol=ip  color=white
point "TGT 1"  BULL 032/48  symbol=target  side=hostile  alt=FL0
```

One position. Name optional (40 characters max).

### `line`

```
line "FLOT"  color=red width=3 dash=dash
     N38 05.0 E126 40.0 ; N38 12.0 E127 30.0 ; N38 20.0 E128 10.0
```

2 to 200 positions. The name is drawn next to the first position.

### `zone`

```
zone "KILL BOX AB12"  color=amber fill=amber:20 side=hostile
     BULL 010/45 ; BULL 020/45 ; BULL 020/60 ; BULL 010/60
```

3 to 200 positions, closed automatically (do not repeat the first position;
if it is repeated, the duplicate is removed). The name is drawn at the
centre.

### `circle`

```
circle "SA-11 POP-UP"  BULL 035/52  radius=15nm  color=red dash=dot
circle "ROZ 3"  center=BULL 180/20  radius=5  fill=green:15  color=green
```

One centre (bare position or `center=`) and `radius=` (required, > 0, at
most 500 NM).

### `orbit`

```
orbit "CAP NORTH"  N37 50.0 E127 10.0 -> N37 50.0 E127 40.0
      width=8nm turn=right alt=FL250-FL300 color=cyan
```

Two positions `A -> B`: the inbound leg, flown from A to B. The racetrack is
the leg A→B, a half turn at B to the `turn` side, the parallel return leg at
`width` from the first, and a half turn back to A. The name and `alt` are
drawn at the middle of the A→B leg.

### `text`

```
text "FENCE IN"  BULL 360/30  size=18 color=amber
text "BRIDGE — DO NOT STRIKE"  N37 31.2 E126 58.9  box=on color=white align=left
```

One position and the text in quotes (80 characters max). No symbol.

## 7. Limits

| Limit | Value | Beyond |
| --- | --- | --- |
| File size | 2 MB | file refused |
| Items per file | 500 | items after the 500th ignored, warning |
| Positions per `line` / `zone` | 200 | reduced to 200 evenly spaced positions (first and last kept), warning |
| Name length | 40 characters | cut, warning |
| `text` length | 80 characters | cut, warning |
| `note` length | 200 characters | cut, warning |
| Line length | 4000 characters | statement ignored, error |
| Items on the TSD, all layers | 2000 | layers beyond are not drawn, the pilot is told |
| `circle` radius / `orbit` width | 500 NM / 100 NM | statement ignored, error |

## 8. Errors and warnings

BMS-CPD reads the whole file and reports problems with their line number:

- **error**: the statement is ignored (unknown keyword, bad position, missing
  radius…). The rest of the file is still read.
- **warning**: the statement is kept, something was ignored or adjusted
  (unknown attribute, name cut, positions reduced…).

A file with no valid item is refused. Example of report:

```
line 12: error: zone needs at least 3 positions (got 2)
line 20: warning: unknown attribute "colour" (ignored)
```

## 9. How BMS-CPD uses a file

- **Import**: on the PC, admin page **TSD graphics** (*Graphismes TSD* in
  French) → *Import*; on the tablet, a `.cpd` received from the squadron (or
  found in the local documents) opens an import prompt in DOC.
- **Display**: each imported file is a layer, drawn on the TSD under the
  flight plan and the tactical objects, above the background map and the
  threat circles. The TSD key **OBJ** (L5) shows or hides all layers; each
  layer can also be switched on or off in the admin page.
- **Re-import**: importing a file with the same layer name replaces the
  previous version of that layer (same id), so a generator can simply
  re-import its file to update the graphics.
- **Sharing**: *Envoyer au groupe* sends the file itself, encrypted, to the
  squadron (tagged with the current mission). Members get it in DOC and
  confirm the import.
- **Storage**: the original text is kept unchanged in
  `<data folder>\layers\` (the data folder is shown in the admin page,
  Avancé).

## 10. Complete example

A second, ready-to-import example (positions from the bullseye) is
[example.cpd](example.cpd).

```
cpd 1
# Mission OSAN STRIKE — TSD graphics
layer "Package VIPER"

line     "FLOT"  color=red width=3 dash=dash
         N38 05.0 E126 40.0 ; N38 12.0 E127 30.0 ; N38 20.0 E128 10.0

zone     "KILL BOX AB12"  color=amber fill=amber:20 side=hostile
         BULL 010/45 ; BULL 020/45 ; BULL 020/60 ; BULL 010/60

circle   "SA-11 POP-UP"  center=BULL 035/52  radius=15nm  color=red dash=dot

orbit    "CAP NORTH"  N37 50.0 E127 10.0 -> N37 50.0 E127 40.0
         width=8nm turn=right alt=FL250-FL300 color=cyan

orbit    "TEXACO"  37.40 127.90 -> 37.10 128.20  width=10nm turn=left
         color=green dash=dash alt=FL220

point    "IP ALPHA"  N37 30.0 E127 05.0  symbol=ip  color=white
point    "TGT 1"  BULL 032/48  symbol=target  side=hostile  note="Bridge, 2 spans"
text     "FENCE IN"  BULL 360/30  size=18 color=amber
```

## 11. Writing a generator

Recommendations for applications that produce `.cpd` files:

1. Start with `cpd 1` and one `layer "Name"` line; keep the same layer name
   between exports so that a re-import replaces the old layer.
2. Prefer decimal degrees with 4 to 6 decimals (`37.090600 127.029700`):
   unambiguous and easy to produce.
3. Write one statement per line when it fits; long lines and zones can use
   continuation lines (indent by at least one space).
4. Quote every name and text; escape `"` as `\"`.
5. Use the named colours; set `side` for tactical meaning.
6. Stay within the limits (section 7); simplify long borders before export.
7. Validate with BMS-CPD: the admin page shows every error and warning with
   its line number.

Minimal generator (Python):

```python
def cpd_file(layer, items):
    out = ["cpd 1", f'layer "{layer}"']
    for it in items:
        pos = " ; ".join(f"{lat:.6f} {lon:.6f}" for lat, lon in it["pos"])
        out.append(f'{it["kind"]} "{it["name"]}" color={it.get("color", "white")} {pos}')
    return "\n".join(out) + "\n"

print(cpd_file("Package VIPER", [
    {"kind": "line", "name": "FLOT", "color": "red",
     "pos": [(38.0833, 126.6667), (38.2, 127.5), (38.3333, 128.1667)]},
]))
```

## 12. JSON form (parser output)

BMS-CPD turns a file into the JSON below (`GET /api/layers` on the PC,
local only). It is documented for tools that want to check their output;
it is **not** an input format.

```json
{
  "id": "package-viper",
  "name": "Package VIPER",
  "file": "osan-strike.cpd",
  "enabled": true,
  "items": [
    {
      "kind": "orbit", "name": "CAP NORTH", "line": 14,
      "pos": [{"lat": 37.8333, "lon": 127.1667}, {"lat": 37.8333, "lon": 127.6667}],
      "widthNm": 8, "turn": "right",
      "alt": "FL250-FL300",
      "style": {"color": "#2EE6FF", "width": 2.5, "dash": "solid", "fill": "", "fillOpacity": 0,
                "size": 18, "label": true}
    },
    {
      "kind": "point", "name": "TGT 1", "line": 21,
      "pos": [{"bull": true, "brg": 32, "nm": 48}],
      "symbol": "target", "side": "hostile", "note": "Bridge, 2 spans",
      "style": {"color": "#FF3B3B", "width": 2.5, "dash": "solid", "fill": "", "fillOpacity": 0,
                "size": 18, "label": true}
    }
  ],
  "diags": [{"line": 20, "level": "warning", "msg": "unknown attribute \"colour\" (ignored)"}]
}
```

Item fields: `kind`, `name`, `line` (line number in the file), `pos` (list
of `{lat, lon}` or `{bull: true, brg, nm}`), `radiusNm` (circle), `widthNm`
and `turn` (orbit), `text`, `align`, `box` (text), `symbol`, `side`, `alt`,
`note`, `style`.

## 13. Versioning

- This is version **1** (`cpd 1`). A file without a `cpd` line is read as
  version 1.
- Later versions only **add** keywords and attributes. A version-1 reader
  ignores unknown attributes (warning) and unknown keywords (error on that
  statement only), so newer files stay readable by older BMS-CPD versions
  as far as possible.
- A file declaring a version higher than the reader supports is still read,
  with a warning.
