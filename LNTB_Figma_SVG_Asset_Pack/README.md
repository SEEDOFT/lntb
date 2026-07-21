# LNTB Figma SVG Asset Pack

This pack contains two SVG asset types.

## Native vector SVG

The icon, navigation, symbol, design-token and UI-component files are real
vector SVGs. They remain editable as vector paths and text in Figma.

Folders:

- `icons/agriculture_iot/`
- `icons/symbols/`
- `icons/navigation/`
- `components/`
- `tokens/`

## Raster-backed SVG

The source design sheet was generated as a PNG. Complex backgrounds and
illustrations cannot be reconstructed as their original vector paths
automatically. Those items are provided as cropped PNG files and SVG wrappers
that embed each PNG.

Folders:

- `backgrounds/`
- `illustrations/`

These import into Figma, but the illustration inside each SVG remains raster.

## Import into Figma

1. Extract the ZIP.
2. Open a Figma Design file.
3. Drag the required `.svg` files onto the canvas.
4. Native vector files can be edited as vectors.
5. Raster-backed assets should be used as image layers or manually redrawn when
   fully editable illustration paths are required.

## Fonts

- Latin: Poppins
- Khmer: Khmer OS Battambang

The fonts are not included in this package.
