# Generalia — repeatable terrain and range workflow

Atlas 06: Short-beaked echidna, 24 September 2026.

## What changed

The whole region now shares one geographic transform. The color image is a NASA geographic composite, not an Australia cutout positioned by eye. There is no constant green or synthetic biome fallback. Height stays in floating-point metres until the display transformation, avoiding the old 8-bit terraces. The web material decodes the sRGB texture correctly, avoiding washed-out colors. Coastlines are cut by a separate high-resolution alpha mask. Range is a separate texture, clipped by the same land mask.

## Repeat this sequence for another region

1. Choose one extent, geographic projection, grid and orientation. This build uses 110–155.5° E, 45.5° S–3° N. North is at the top; Tasmania lies southeast of mainland Australia. Include framing margin before downloading anything.
2. Acquire and preserve real DEM measurements and geographically referenced surface imagery for the entire extent. Inspect the unstyled, north-up color raster first. Never infer geographic coordinates from the bounding rectangle of a screenshot.
3. Resample elevation, color and coastlines into that one grid. Store raw metres separately. Keep image precision independent of mesh density: this build uses a 3000 × 3200 texture and 1500 × 1600 vertex grid. NASA color has approximately 2 km native sampling; resampling does not create new satellite detail. Cached terrain tiles have finer source sampling than the displayed mesh.
4. Create the terrain-only Blender master. Add modest, continuous relief compression and measured-DEM shading. Compare the full region and close views of New Guinea, Cape York, Tasmania and the Australian interior. Save the Blender scene with packed textures and export a terrain-only GLB.
5. Register the user's range reference. Extract only its red color. Fit the reference coastline to real coastline geometry; fit a region separately if a diagram has local distortion. Save the fitted transforms, residuals, original image and coastline overlay. Do not add external species range maps. Do not confuse clipping to land with correctly registering the range.
6. Apply the range in the HTML viewer after terrain construction. Use the same UV mapping as the terrain. Red must remain distinct while preserving relief. The swatch is filled only when the layer is active. Toggle the layer off to inspect the natural palette.
7. Review actual browser screenshots with range on and off. Check registration, orientation, full coverage, coastal seams, visible color variation, elevation spikes, camera margins, page errors, range toggle and reset. Inspect the reference overlay, not just a passed numeric check.
8. Package a new version with the source manifest, scripts, terrain-only Blender/GLB, portable HTML and review images. Retain the previous approved build.

## Coordinates and artistic treatment

`u = (longitude - 110) / 45.5`, `v = (3 - latitude) / 48.5`.

Web coordinates: `x = (u - 0.5) × 17.15`, `z = (v - 0.5) × 19.4`. Blender uses the same horizontal dimensions, with north along +Y and elevation along +Z. This is a regional equirectangular display with longitude scale adjusted for the region; it is not an equal-area measurement surface.

Displayed height: `0.95 × 900 × log(1 + elevation_metres / 900) / 4000`. Raw elevation is preserved in `source_data/elevation_metres.npy`. This nonlinear artistic exaggeration limits high peaks while retaining small ranges. It is not a uniform vertical scale for measurement.

Satellite colors receive a documented contrast/saturation adjustment and shadow lift. Elevations above 650 m gradually receive a restrained olive/stone relief tint, with a maximum 50% blend at 3050 m. The tint is an artistic interpretation of measured elevation, not a land-cover classification. Fine shading is derived from the DEM. Bright areas such as salt lakes and seasonal snow remain where they appear in the source.

## Sources and provenance

### Shoreline refinement and opening motion

The displayed shore is pinned to sea level within four texture samples of the coastline, then blended smoothly into the unchanged inland DEM over eight further samples (roughly an 18 km total transition at this continental scale). This is an artistic coastal treatment, not a claim that coastal cliffs are naturally flat. Original elevation measurements remain unchanged. The coast alpha mask is rasterized at double resolution and reduced for smoother edges, with multisample coverage in the browser. `qa/coastline_checks.json` verifies the zero-height edge. Slow orbit starts enabled on page load and pauses on direct map interaction.

- **Surface color:** NASA Earth Observatory, Blue Marble Next Generation, July 2004 base map, 21600 × 10800 global JPEG. This is a monthly composite rather than single-day imagery, with geographic bounds −180–180°, −90–90°. It is historical and seasonal, not a present-day vegetation survey. [Source page](https://science.nasa.gov/earth/earth-observatory/blue-marble-next-generation/base-map/). [Downloaded source](https://assets.science.nasa.gov/content/dam/science/esd/eo/images/bmng/bmng-base/july/world.200407.3x21600x10800.jpg). NASA imagery usage guidance applies; credit NASA Earth Observatory and do not imply endorsement. [NASA usage guidance](https://www.nasa.gov/nasa-brand-center/images-and-media/).
- **Elevation:** existing Mapzen / Tilezen Terrain Tiles Terrarium cache, zoom 8, 1360 tiles, from the prior project. These are real multi-source elevation data; this build does not claim a new Copernicus DEM acquisition. Preserve source-provider credits per [Terrain Tiles attribution](https://github.com/tilezen/joerd/blob/master/docs/attribution.md) and the [AWS registry](https://registry.opendata.aws/terrain-tiles/).
- **Coastlines:** existing Natural Earth 1:10m country geometry; public-domain Natural Earth data. Countries are unioned independently so polygon holes cannot erase other land. Coastline detail is limited by this dataset.
- **Range:** `short_beaked_red_reference.png`, supplied by the user. No other range map was consulted. Original author/publication/license was not provided; retain that provenance uncertainty before public redistribution. The red diagram is an approximate distribution illustration, not a survey boundary. Mainland Australia and Tasmania are filled to the common coastline as in the reference; small islands follow the reference sampling. Thin black political lines inside red are closed before sampling. Blue, green and point symbols do not define this species' range.
- **Renderer:** Three.js r128 and OrbitControls, MIT license, bundled in the HTML for offline use.

## Build and files

Run `scripts/build.py` with Python, NumPy, Pillow and SciPy. The script reads the original tile cache, coastline and red reference from the adjacent `tachyglossus_aculeatus_austral_pacific_v05_rebuilt/source_data` directory. Preserve that directory when moving this project, or change the `OLD` input location in the script. The NASA source and rendering libraries are in this branch's `source_data` directory.

Run Blender in background with `scripts/blender_master.py` to reproduce the packed scene and GLB. The script starts an independent scene and does not alter the open Blender session or the original long-beaked scene.

- `viewer/index.html`: portable interactive viewer, all rendering data and libraries embedded.
- `blender/short_beaked_atlas_v06.blend`: packed terrain-only master.
- `blender/short_beaked_terrain_v06.glb`: corresponding terrain-only model.
- `qa/reference_coast_registration.png`: cyan real coastline drawn on the user's source to inspect alignment.
- `qa/checks.json`: grid, raw elevation maximum, geographic range transforms and residuals.
- `qa/terrain_overview.jpg`, `qa/range_overview.jpg`: north-up raster checks.

For future species sharing this terrain, preserve the terrain and its UV coordinates. Create an independently registered mask for each species. If several species must be active together, extend the range shader with equal-weight diagonal stripes; the current delivered viewer has one species layer.
