# Verification report

## Passed

- Downloaded the official 102-subject TotalSegmentator subset and matched the published MD5.
- Verified each subject has `ct.nii.gz` and `segmentations/` before selection.
- Chose s0720 by maximum summed physical volume among qualifying subjects; complete serial results are in `pipeline/selection.json` and `selection.log`.
- Asserted canonical RAS laterality: liver centroid i > spleen centroid i.
- Inspected the axial, coronal, and sagittal PNGs through the liver centroid. Liver is on screen left in axial and coronal; axial spine is at the bottom. The sagittal liver slice is lateral to the spine, so a separate sagittal spine-centroid image verifies the spine on screen right.
- Verified all eight navigation targets are inside their own label in each of the three displayed planes.
- Verified voxel/world round trips and inverse-transformed mesh surfaces against labels. 99th-percentile surface-to-label distance is 2 mm for every full-resolution mesh, within one web voxel after smoothing.
- Full meshes all have <=40,000 faces.
- Final `web/public/data`: 18,488,762 bytes, below 20 MB.
- Production Next.js builds passed, including TypeScript checks.
- Browser desktop: software 3D meshes render; slice planes intersect the selected target consistently; left-kidney selection gives crosshair (52, 50, 100) and data label 5.
- Browser mobile: verified the responsive layout in a 390 px iframe viewport, captured all four stacked panels, and confirmed left-kidney selection yields label 5.
- Browser mesh click: clicking liver changed selected structure and all coordinates to the clicked surface point.
- Browser slice slider, canvas drag, and mouse wheel changed shared crosshair values correctly. Canvas dragging across a labeled structure selected it.
- Browser visibility: hiding kidney_left changed the eye state; selecting kidney_left restored visibility and moved to its interior target.

## Issues encountered and fixes

1. Requested local data was absent. With the user's authorization, downloaded the official public subset rather than fabricating scans.
2. Default sandbox download could not reach the network proxy. The approved download execution succeeded; the final archive checksum matches the official record.
3. Supervised preview passes Vite-style `--host` / `--strictPort` flags, which Next.js rejects. Added `scripts/dev.mjs` to translate `--host` to `--hostname` and omit `--strictPort`; the requested Next.js stack is retained.
4. Native 1.5 mm assets totaled 39 MB, contradicting the 20 MB limit. Resampled CT and labels together to a documented 2 mm RAS grid. The original affine and spacing remain in metadata.
5. Left-kidney (and several other structures') true centroids are outside their curved masks. Preserved the actual centroids and used nearest interior voxels when needed for selection. The target is independently checked against all three slice mappings.
6. Cloud test browser cannot create a WebGL context. Added a tested software renderer using real simplified GLB meshes, shared transforms, orbit controls, and raycasting. WebGL2 browsers still use React Three Fiber and the full meshes.
7. Concurrent development and production builds initially shared `.next`, causing a temporary preview 500. Configured `.next-dev` for development and `.next` for production; reloaded successfully.
8. Material replacement on selection could leave GPU materials allocated. Added cleanup for replaced cloned materials.
9. The extra spine QA image initially inherited the `_liver` filename suffix; corrected the exporter and saved filename.

## Limits and deviations

- Actual iPhone Safari was not available. The mobile check is a 390 px browser viewport, not a physical-device or Safari test.
- Hardware WebGL rendering could not be visually exercised in this browser. The canvas compatibility renderer was visually and interactively verified; full GLB geometry was checked numerically against the label volume.
- The true left-kidney centroid cannot simultaneously be the navigation target and lie within this mask. Nearest-interior navigation is the documented resolution.
- A sagittal slice at the liver centroid does not intersect the spine in this scan. Posterior spine orientation was verified on an additional midline sagittal slice.
- Native 1.5 mm uncompressed volumes cannot meet the 20 MB limit. The documented 2 mm derivatives meet it without changing which subject was selected.
- Optional WebMCP registration is feature-detected, but no supported tool registry was available in the test document; its runtime path could not be validated. The complete UI does not depend on WebMCP.

## Delivery status

The app and static production export are complete. Publishing was blocked by automatic approval review: uploading the project source and assets to the Sites deployment destination requires explicit user authorization. No deployment or source push occurred. The downloadable bundle includes source, prebuilt `out/`, processed CT data, scripts, and verification evidence. The reserved Site identity is retained locally for continuation after approval.
