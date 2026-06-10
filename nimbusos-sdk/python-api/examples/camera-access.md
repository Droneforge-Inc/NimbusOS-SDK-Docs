---
description: Example of how to access the drones camera feed through NimbusOS SDK
---

# Camera Access

In version 0.1.7 and later, the camera is able to be accessed easily by importing&#x20;

`from nimbusos_sdk import CameraFrame`

Looping through `client.latest_live_camera_frames()` you can access the latest analog camera frame published by the connected drone.



Here is an example of display the latest camera frame in an OpenCV window.

```python
from __future__ import annotations

import time

import cv2
import numpy as np

from nimbusos_sdk import CameraFrame
from nimbusos_sdk import NimbusClient


def decode_bgr(frame: CameraFrame) -> np.ndarray | None:
    jpeg = np.frombuffer(frame.jpeg, dtype=np.uint8)
    return cv2.imdecode(jpeg, cv2.IMREAD_COLOR)


def main() -> None:
    last_status_at_s = 0.0

    try:
        with NimbusClient() as client:
            for frame in client.latest_live_camera_frames():
                image_bgr = decode_bgr(frame)
                if image_bgr is None:
                    continue

                cv2.imshow("Nimbus Camera", image_bgr)

# Want to calculate some data about the camera frame speed? Uncomment me!

#                now_s = time.monotonic()
#                if now_s - last_status_at_s >= 1.0:
#                    print(f"seq={frame.seq} {frame.width}x{frame.height}", flush=True)
#                    last_status_at_s = now_s

                if cv2.waitKey(1) & 0xFF in (ord("q"), 27):
                    break
    finally:
        cv2.destroyAllWindows()


if __name__ == "__main__":
    main()


```
