---
name: static-repose-angle-measurement
description: Measure static angle of repose and spreading distance from EDEM-like particle pile screenshots. Use when Codex needs to crop left/right soil slopes, binarize images, extract black soil boundary contours, fit slope lines, compute repose angle from tan(theta), measure spread distance using a known transparent cylinder diameter, update cumulative results spreadsheets, or create annotated display images.
---

# Static Repose Angle Measurement

## Core Workflow

Use this skill for screenshots where a black granular pile sits below a semi-transparent vertical cylinder and the user wants angle-of-repose measurements, binary masks, fitted line annotations, or spread-distance measurements.

Follow this order:

1. Inspect the workspace and confirm source images are under `initial_fig`.
2. Detect the semi-transparent cylinder by gray vertical pixels in the upper image and detect black soil by near-black pixels in the lower image.
3. Crop left and right slope regions into `local_fig`:
   - Left crop: soil left edge to cylinder left edge.
   - Right crop: cylinder right edge to soil right edge.
   - Use the cylinder bottom as the crop top.
   - Adjust each crop bottom so the lower outside corner contains black soil and no white corner gap.
4. Binarize crop images into `binary_fig` using the same filenames as the cropped images.
5. Extract the upper soil contour from each binary image by taking the first black pixel in each column.
6. Fit `y = kx + b` in mathematical image coordinates (`x` rightward, `y` upward).
7. Compute `repose_angle_deg = degrees(atan(abs(k)))`; the slope `k` is `tan(theta)` with sign preserved by side.
8. Measure spreading distance from the original image as black soil right edge minus left edge, converted to mm with the known cylinder outer diameter.
9. Write or update `results.xlsx` cumulatively:
   - Use `浼戞瑙抈 as the fit-data worksheet name.
   - Use `鎵╂暎璺濈` as the spread-distance worksheet name.
   - Do not overwrite previous image rows when adding a new image.
   - Insert `缁勫彿` as the first header in both worksheets.
   - Derive `缁勫彿` from the source stem before the first axis suffix, so `r1_x` and `r1_y` both use `r1`.
   - Sort rows by natural group order and image name, so `r10` follows `r9` instead of being placed after `r1`.
   - Merge adjacent cells in the `缁勫彿` column when rows share the same group.
10. Create or update `display_fig/<source_name>.png` by copying the original image and drawing:
   - Red dashed fitted lines.
   - Chinese labels for fit equation, repose angle, and `R虏`.
   - Red dashed spread-distance marker and known cylinder-diameter marker.

Do not delete source images. Respect any project-specific deletion policy; when deletion is requested, delete only explicitly confirmed files one at a time.

## Reusable Script

Prefer the bundled script for repeatable measurements:

```powershell
python D:\simulated_results_of_static_repose_angle\.codex\skills\static-repose-angle-measurement\scripts\measure_static_repose.py --root D:\simulated_results_of_static_repose_angle --image r1_x.png --cylinder-mm 95
```

Important options:

- `--root`: Project root containing `initial_fig`.
- `--image`: Source image filename inside `initial_fig`.
- `--cylinder-mm`: Known transparent cylinder outer diameter in millimeters.
- `--threshold`: Binary threshold for black soil; default `128`.
- `--prefix`: Output prefix; defaults to the source stem, for example `r1_x`.

Expected outputs:

- `local_fig/<prefix>1.png` and `local_fig/<prefix>2.png`
- `binary_fig/<prefix>1.png` and `binary_fig/<prefix>2.png`
- `display_fig/<source_image>`
- `results.xlsx`
- `results_data.json` as an intermediate audit file; keep prior rows and append/update new rows by image name.

## Measurement Conventions

- PIL crop boxes use right/bottom exclusive coordinates.
- Cylinder diameter in pixels is measured as an inclusive width from detected outer left edge to outer right edge.
- If the cylinder appears as separated vertical gray stripe runs, merge all central cylinder stripe runs into one outer-diameter range instead of using only the longest continuous run.
- Soil spreading distance is measured as the inclusive horizontal distance from the leftmost black soil pixel to the rightmost black soil pixel.
- Fitted equations in annotations use local crop coordinates with `y` upward.
- Draw labels in Chinese and place them in white background regions so they do not cover the black soil pile or the semi-transparent cylinder.

## Verification Checklist

Before finishing:

- Preview `local_fig` crops and confirm outside lower corners are not white.
- Preview `binary_fig` masks and confirm soil is black and background is white.
- Preview `display_fig` and confirm red dashed lines align with the slope trends.
- Confirm Chinese labels are readable and do not cover the soil or cylinder.
- Inspect `results.xlsx` to ensure:
  - Worksheets are named `浼戞瑙抈 and `鎵╂暎璺濈`.
  - The first header in both sheets is `缁勫彿`.
  - Prior image rows remain present after adding a new image.
  - Same-group cells in the `缁勫彿` column are merged.
  - Groups are ordered naturally as `r1`, `r2`, ..., `r10`, `r11`, `r12`.
  - No Excel formula errors are present.
