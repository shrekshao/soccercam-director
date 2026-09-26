# Director camera-path format

This document describes schema **1.0** and the behavior of the current Soccer-cam player.
Use `director.json` as the filename, not `directed.json`. `directed.mp4` refers to a
separately rendered video and is not an input camera-path file.

## Files

| File | Purpose |
| --- | --- |
| `director.json` | Time-indexed camera poses and source/projection metadata. |
| `calibration.json` | Source field of view, orientation, and camera limits. Supply the matching file for faithful playback. |
| Source video | The matching original wide video. Not included in this repository. |

`source.json`, analysis observations, model files, and detection decisions are not
needed to play these examples. Do not mix calibration files between examples.

## Top-level fields

| Field | Type / value | Meaning |
| --- | --- | --- |
| `schema_version` | String, `"1.0"` | Required schema identifier. |
| `source_id` | String | Descriptive source identifier, usually a filename stem; not a YouTube ID or guaranteed unique ID. The current reader defaults to `"unknown"`. |
| `source_sha256` | Optional string | SHA-256 of the original full-resolution source file. Informational for YouTube playback. |
| `analysis_source_sha256` | Optional string | SHA-256 of the input used for analysis; it can differ when analysis uses a proxy. |
| `projection` | `"flat_uncalibrated"` or `"erp360"` | Required source projection. |
| `coordinate_system` | `"field_camera_yaw_right_pitch_up"` | Positive yaw points right; positive pitch points up. The reader defaults to this value. |
| `source_center_yaw` | Number, degrees | Source orientation offset for ERP sampling; defaults to zero. Do not add it to flat-mode sample yaw. |
| `calibration_status` | Optional string | Descriptive calibration provenance, not an executable instruction. |
| `time_unit` | `"seconds"` | Defaults to seconds if absent; other units are rejected. |
| `angle_unit` | `"degrees"` | Defaults to degrees if absent; other units are rejected. |
| `fov_axis` | `"horizontal"` | FOV is horizontal; other axes are rejected. Defaults to horizontal. |
| `angle_interpretation` | Optional string | Typically `"linear_image_angle_approximation"` for flat sources or `"spherical"` for ERP. |
| `output_aspect` | Two positive numbers, e.g. `[16, 9]` | Intended output aspect ratio; defaults to `[16, 9]`. Metadata does not force the browser viewport to this ratio. |
| `smoothing_sigma_seconds` | Optional object | Generator diagnostics. The samples are already smoothed; the player does not use this field. |
| `samples` | Nonempty array | Camera poses in strictly increasing timestamp order. |

Write explicit metadata when producing new files, even where the current reader has
defaults. Unrecognized generator metadata is not required for playback. Metadata such
as coordinate-system labels and output aspect is not fully validated by the current reader.

## Camera samples and units

Each sample contains four finite JSON numbers:

| Field | Unit | Meaning |
| --- | --- | --- |
| `t` | Seconds | Time from the source video's first decoded frame. |
| `yaw` | Degrees | Horizontal camera direction, positive right. |
| `pitch` | Degrees | Vertical camera direction, positive up. |
| `fov` | Degrees | Horizontal field of view; smaller values zoom in. |

Use positive FOV values appropriate to the source and view mode. Do not write `NaN`,
infinities, duplicate timestamps, or descending timestamps. Samples need not be equally
spaced and need not correspond one-to-one with decoded video frames.

## Timing and interpolation

Each clip or segment has its own zero-based timeline. Do not concatenate segment paths
without adjusting their timestamps. For YouTube, the current player samples at:

```text
pathTimeSeconds = videoTimeSeconds - timeOffsetMs / 1000
```

A positive time offset delays the path. Between adjacent samples `a` and `b`:

```text
alpha = (pathTimeSeconds - a.t) / (b.t - a.t)
value = a.value + alpha * (b.value - a.value)
```

Apply this independently to yaw, pitch, and FOV. Before the first sample, hold its pose;
after the last sample, hold the last pose. Do not add smoothing or shortest-angle
interpolation when reproducing the current player. Hash metadata does not synchronize
video playback; matching the correct video and timeline is essential.

## Projection types

### `flat_uncalibrated`

A flat wide source interpreted through an approximate angular mapping. Supply
`calibration.assumed_hfov`; the current player falls back to 180 degrees if it is absent.
These are approximate image angles, not a guarantee of physical lens calibration.

For the baseline Plane crop on a 16:9 source, with `H = assumed_hfov`:

```text
scale = min(1, fov / H)
centerX = yaw / H + 0.5
centerY = 0.5 - pitch / (H * 9 / 16)
```

Clamp the crop center so the crop stays inside the source. Positive pitch moves the
crop upward. `source_center_yaw` is not added in this projection. Cylinder and Sphere
view modes use different rendering geometry; changing view mode does not change the
declared source projection.

### `erp360`

A stitched equirectangular panorama, normally 2:1, spanning 360 degrees horizontally
and 180 degrees vertically. Angles are spherical. Apply `source_center_yaw` once when
mapping yaw to the source panorama; matching calibration can provide the effective
center yaw. Wrap horizontally at the panorama seam.

This format support does **not** imply support for YouTube's native 360-degree player.
The current extension targets ordinary flat uploads; both supplied examples are flat.

## Valid small example

This synthetic two-second path demonstrates the format. It is not a path for either
linked YouTube video; use the full files in `examples/` for those videos.

```json
{
  "schema_version": "1.0",
  "source_id": "synthetic-wide-clip",
  "projection": "flat_uncalibrated",
  "coordinate_system": "field_camera_yaw_right_pitch_up",
  "source_center_yaw": 0,
  "time_unit": "seconds",
  "angle_unit": "degrees",
  "fov_axis": "horizontal",
  "angle_interpretation": "linear_image_angle_approximation",
  "output_aspect": [16, 9],
  "samples": [
    { "t": 0, "yaw": -10, "pitch": -5, "fov": 90 },
    { "t": 1, "yaw": 0, "pitch": -5, "fov": 80 },
    { "t": 2, "yaw": 10, "pitch": -5, "fov": 90 }
  ]
}
```

## Calibration

The current player reads this subset of `calibration.json`:

| Field | Meaning |
| --- | --- |
| `projection` | Source projection metadata; keep consistent with the director file. The director determines playback projection. |
| `assumed_hfov` | Horizontal span of the flat source in degrees. |
| `center_yaw` | Reference yaw; ERP source orientation offset, or baseline for flat-mode user adjustments. |
| `center_pitch` | Reference pitch for user adjustments; not an extra offset to add to every sample. |
| `yaw_limits` | `[minimum, maximum]` yaw in degrees. |
| `pitch_limits` | `[minimum, maximum]` pitch in degrees. |
| `fov_limits` | `[minimum, maximum]` horizontal FOV in degrees. |

Example matching the synthetic path above:

```json
{
  "projection": "flat_uncalibrated",
  "assumed_hfov": 180,
  "center_yaw": 0,
  "center_pitch": -5,
  "yaw_limits": [-65, 65],
  "pitch_limits": [-25, 15],
  "fov_limits": [60, 110]
}
```

The original example snapshots also contain analysis regions, generator settings,
`annotation_source`, and `annotation_time_seconds`. The player ignores these fields;
`annotation_source` is a historical local path, not a file that users must provide.
When calibration limits are absent, the current player derives limits from sample
ranges. With no user overrides, calibration reference values do not shift the saved
flat camera path. User changes can remap FOV or constrain the view.
