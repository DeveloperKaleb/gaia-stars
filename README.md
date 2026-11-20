# gaia-stars

The stars behind the "Our Cosmic Address" map on
[developerkaleb.github.io/PortfolioWebPage](https://developerkaleb.github.io/PortfolioWebPage/rubin/rubin.html):
every star in Gaia Data Release 3 whose distance is known to within 1%, 5,837,823 of
them, as small binary tiles the page loads as its view needs them.

Published with GitHub Pages at `developerkaleb.github.io/gaia-stars/`, on the same site
as the page, so a visitor's browser contacts nothing but the site's own host. Nothing here
is edited by hand: the tiles are built by the tools in the site repository.

## Where the data comes from

This work has made use of data from the European Space Agency (ESA) mission
[Gaia](https://www.cosmos.esa.int/gaia), processed by the Gaia Data Processing and
Analysis Consortium ([DPAC](https://www.cosmos.esa.int/web/gaia/dpac/consortium)). Funding
for the DPAC has been provided by national institutions, in particular the institutions
participating in the Gaia Multilateral Agreement.

The stars were selected from `gaiadr3.gaia_source` through the
[Gaia Archive](https://gea.esac.esa.int/archive/) with `parallax_over_error > 100`
(a parallax, and so a distance, known to within 1%), asked for in pieces, gently, once.

Gaia DR3: Gaia Collaboration, Vallenari et al. 2023, A&A 674, A1.

## Licence

Gaia data are released under the
[Creative Commons Attribution-ShareAlike 3.0 IGO](https://creativecommons.org/licenses/by-sa/3.0/igo/)
licence. These tiles are derived from them and are shared under the same licence, with the
attribution above.

## Layout

| Path | What |
|---|---|
| `dr3-1pct/index.json` | The tree: its frame, the root cube's half-width, stars per tile, and every tile as `[path, count]` |
| `dr3-1pct/r<path>.bin` | One tile per node of the tree: `r.bin` is the root, `r07.bin` its child 0's child 7 |
| `dr3-1pct/r<path>.ids` | Each tile's stars named, in the same order: added 2026-11-19, the tiles unchanged |
| `dr3-1pct/r<path>.names.json` | The friendlier names some of a tile's stars have, `{ k: name }` by place in the tile: added 2026-11-19 |
| `dr3-1pct/r<path>.first.json` | The oldest catalogue each star is in, `{ k: code }`, the codes those of `FIRST_RECORDS` in the site's `js/starnames.js`; a star not listed is in none older than Gaia: added 2026-11-19 |

A new selection or format goes in a folder of its own beside this one, so the page can
move to it without the old one changing under it.

## The format

The frame is the Sun's, in parsecs, along the ecliptic of J2000: x toward the March
equinox, z toward the ecliptic's north pole. The tree is an octree over a cube 2,048 pc
from the Sun each way. Each tile holds up to 4,096 stars, the most luminous of its cube
that its parent did not take, so a tile near the root holds the brightest stars of a
large region and its children the fainter ones.

A child's octant is a digit, 0 to 7: bit 1 is +x, bit 2 is +y, bit 4 is +z.

Each star is 8 bytes, little-endian:

| Bytes | What |
|---|---|
| 0-1, 2-3, 4-5 | x, y, z as uint16 across the tile's own cube (0 is its low edge, 65,535 its high edge) |
| 6 | Absolute G magnitude, in eighths from -8 (byte = (M + 8) × 8) |
| 7 | BP-RP colour, in fortieths from -1 (byte = (c + 1) × 40); 255 where Gaia has none |

Beside each tile, its ID file names its stars, in the tile's order, for the page to load
only about a star a reader has picked:

| Bytes | What |
|---|---|
| 8 per star | Its Gaia DR3 `source_id`, as uint64 |
| 4 | How many of the tile's stars have a Hipparcos number, as uint32 |
| 8 per such star | Its place in the tile, then its Hipparcos number, both uint32 |

Hipparcos numbers are those Gaia's `gaiadr3.hipparcos2_best_neighbour` gives. The tiles
were rebuilt with the IDs and checked byte for byte against those already here, so the
`.bin` files did not change.

A star's name file entry, where it has one, is the best of: a proper name, Bayer, Flamsteed,
Wolf or Ross, Gliese (GJ), a variable star's name, HD, LHS; the page falls back on the
Hipparcos number, then Gaia's. 62,682 stars have one. The names come from
[SIMBAD](https://simbad.u-strasbg.fr/) (Wenger et al. 2000, A&AS 143, 9), asked through
its mirror at IDIA while Strasbourg was offline, with data to 2026-10-14; and HD numbers
from the Hipparcos catalogue (ESA 1997, VizieR I/239). This research has made use of the
SIMBAD database, operated at CDS, Strasbourg, France.

The code that writes and reads them is `js/startiles.js` in the site repository, with
its tests; the tools are `tools/gaiastars/fetch.mjs` and `build.mjs`.
