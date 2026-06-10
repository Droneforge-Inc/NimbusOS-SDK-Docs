---
description: Example of how to access the drone camera feed through NimbusOS SDK
---

# Camera Access

Import `CameraFrame` and `NimbusClient` from `nimbusos_sdk`:

```python
from nimbusos_sdk import CameraFrame
from nimbusos_sdk import NimbusClient
```

Use `client.latest_camera_frames()` for the core-selected `camera` stream. Use `client.latest_live_camera_frames()` for the raw `live_camera` stream before core camera-source selection.

Here is an example that displays the latest live camera frame in an OpenCV window.

```python
from __future__ import annotations

import cv2
import numpy as np

from nimbusos_sdk import CameraFrame
from nimbusos_sdk import NimbusClient


def decode_bgr(frame: CameraFrame) -> np.ndarray | None:
    jpeg = np.frombuffer(frame.jpeg, dtype=np.uint8)
    return cv2.imdecode(jpeg, cv2.IMREAD_COLOR)


def main() -> None:
    try:
        with NimbusClient() as client:
            for frame in client.latest_live_camera_frames():
                image_bgr = decode_bgr(frame)
                if image_bgr is None:
                    continue

                cv2.imshow("Nimbus Camera", image_bgr)

                print(f"seq={frame.seq} {frame.width}x{frame.height}", flush=True)

                if cv2.waitKey(1) & 0xFF in (ord("q"), 27):
                    break
    finally:
        cv2.destroyAllWindows()


if __name__ == "__main__":
    main()
```

You can smoke-test camera topics from the command line:

```bash
nimbusos-subscribe camera --limit 1 --timeout 5
nimbusos-subscribe live_camera --limit 1 --timeout 5
```
