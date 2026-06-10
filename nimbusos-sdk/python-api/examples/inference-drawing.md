---
description: Publish camera overlay drawing instructions for inference results.
---

# Inference Drawing

The SDK has a camera overlay API for drawing inference results in the NimbusOS desktop application without republishing modified camera images.

Overlays are drawing instructions sent on the `camera_overlay` topic. Camera pixels stay on the `camera` topic.

Overlay coordinates are in image pixels. The origin is the top-left of the image, `x` moves right, and `y` moves down. The NimbusOS UI scales the overlay to the camera view using the `frame_width` and `frame_height` you publish with the overlay.

In simple terms think the following:

* `x=0, y=0` is the top-left corner.
* `x=frame_width, y=frame_height` is the bottom-right corner.
* A box uses `x`, `y`, `width`, and `height`.
* A line or arrow uses `x`, `y`, `x2`, and `y2`.
* A layer is a named group of drawings. Higher `z_index` values draw on top.

It's important to note that overlay coordinates must match the image size you pass as `frame_width` and `frame_height`. If inference runs on a resized image, either publish that resized width and height or scale the inference coordinates back to the original camera frame.

The current SDK supports primitives such as `rect`, `rotated_rect`, `line`, `arrow`, `circle`, `ellipse`, `polyline`, `polygon`, `contour`, `text`, `label`, `keypoints`, `skeleton`, `trail`, `heatmap`, `mask`, `depth_map`, `gauge`, `panel`, `pose_axes`, `uncertainty_ellipse`, and `prediction_cone`.

The simplest way to send an inference overlay in the SDK is to call:

```python
client.publish_camera_overlay(
    camera_seq=frame.seq,
    frame_width=frame.width,
    frame_height=frame.height,
    source="my_detector",
    ttl_ms=250,
    layers=[
        {
            "name": "detections",
            "z_index": 10,
            "primitives": [
                box(120, 80, 180, 240, label="person", confidence=0.91),
            ],
        }
    ],
)
```

`camera_seq` tells NimbusOS which camera frame the overlay belongs to.

`ttl_ms` tells NimbusOS how long the drawing should stay visible if no newer overlay arrives. Use a short value for live inference so stale boxes disappear quickly.

`source` identifies which app or model published the overlay.

`layers` gives the drawing commands. Each layer can contain boxes, lines, arrows, text, circles, polygons, masks, heatmaps, and other primitives supported by the SDK.

Example combining camera frames, a person label, and a skeleton, inspired by an air marshaling demo with [SAM-3D-BODY-DINOV3](https://huggingface.co/facebook/sam-3d-body-dinov3):

```python
from __future__ import annotations

import sys

from nimbusos_sdk import CameraOverlayLayerDict
from nimbusos_sdk import NimbusClient
from nimbusos_sdk import box

SKELETON_LINKS = (
    ("left_shoulder", "right_shoulder"),
    ("left_shoulder", "left_elbow"),
    ("left_elbow", "left_wrist"),
    ("right_shoulder", "right_elbow"),
    ("right_elbow", "right_wrist"),
    ("left_shoulder", "left_hip"),
    ("right_shoulder", "right_hip"),
    ("left_hip", "right_hip"),
)


def build_overlay_layers() -> list[CameraOverlayLayerDict]:
    # Example output from an inference model. Real model outputs should already
    # be in the same coordinate space as frame_width and frame_height.
    bbox_x = 220.0
    bbox_y = 90.0
    bbox_w = 210.0
    bbox_h = 360.0
    confidence = 0.93

    keypoints = {
        "left_shoulder": (275.0, 170.0),
        "right_shoulder": (360.0, 170.0),
        "left_elbow": (255.0, 245.0),
        "right_elbow": (385.0, 245.0),
        "left_wrist": (240.0, 320.0),
        "right_wrist": (405.0, 320.0),
        "left_hip": (295.0, 350.0),
        "right_hip": (345.0, 350.0),
    }

    skeleton_primitives = []
    for start_name, end_name in SKELETON_LINKS:
        start_x, start_y = keypoints[start_name]
        end_x, end_y = keypoints[end_name]
        skeleton_primitives.append(
            {
                "type": "line",
                "tone": "primary",
                "x": start_x,
                "y": start_y,
                "x2": end_x,
                "y2": end_y,
                "stroke_width_px": 2.0,
            }
        )

    return [
        {
            "name": "detections",
            "z_index": 10,
            "primitives": [
                box(
                    bbox_x,
                    bbox_y,
                    bbox_w,
                    bbox_h,
                    label="person",
                    confidence=confidence,
                    tone="success",
                ),
            ],
        },
        {
            "name": "skeleton",
            "z_index": 15,
            "primitives": skeleton_primitives,
        },
    ]


def main() -> None:
    with NimbusClient() as client:
        for frame in client.latest_camera_frames():
            print(f"Publishing overlay for camera frame {frame.seq}", flush=True)
            client.publish_camera_overlay(
                camera_seq=frame.seq,
                frame_width=frame.width,
                frame_height=frame.height,
                source="inference_example",
                ttl_ms=250,
                layers=build_overlay_layers(),
            )


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nStopped by Ctrl+C", flush=True)
        sys.exit(130)
```

The person box includes `label="person"`, so the UI can show the person text. The skeleton line primitives intentionally do not include `track_id`, because the skeleton should draw as clean lines without repeated number labels. Add `track_id` only to detection primitives when you want the UI to show an identity number.

You can smoke-test overlay messages from the command line:

```bash
nimbusos-subscribe camera_overlay --limit 1 --timeout 5
```

No command-line helper is installed for publishing camera overlays.
