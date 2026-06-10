---
description: Publish body-frame relative waypoints and waypoint speed with the current NimbusOS SDK.
---

# Waypoints

The current SDK publishes relative waypoints through the autonomy request API. The helper method is `client.publish_relative_waypoint(...)`.

Relative waypoint offsets are in the drone body frame:

| Field     | Positive movement | Negative movement |
| --------- | ----------------- | ----------------- |
| `forward` | Forward           | Backward          |
| `right`   | Right             | Left              |
| `down`    | Down              | Up                |

Going up requires a negative `down` value.

The simplest way to send a relative waypoint in the SDK is:

```python
client.publish_relative_waypoint(
    mode="override",
    forward=1.0,
    right=0.0,
    down=0.0,
    threshold_m=0.25,
    hold_time_s=0.0,
)
```

Use `mode="override"` to replace the active waypoint. Use `mode="queue"` to append after the active waypoint.

`threshold_m` is the distance from the target that counts as reached. `hold_time_s` is how long NimbusOS should hold after reaching the waypoint.

Waypoint path speed is configured separately:

```python
client.publish_waypoint_speed(0.45)
```

The accepted speed range is `0.05` to `0.75` meters per second.

Example combining [arming.md](arming.md "mention"), [takeoff.md](takeoff.md "mention"), a waypoint, and a [landing.md](landing.md "mention") request:

```python
from __future__ import annotations

import sys
import time

from nimbusos_sdk import NimbusClient

ARM_WAIT_S = 3.0
TAKEOFF_WAIT_S = 10.0
WAYPOINT_SPEED_MPS = 0.45


def main() -> None:
    with NimbusClient() as client:
        print("Publishing arm", flush=True)
        client.publish_arm_state(True)

        time.sleep(ARM_WAIT_S)
        print("Publishing takeoff", flush=True)
        client.publish_autonomy_request("takeoff")

        print(f"Waiting {TAKEOFF_WAIT_S:.0f} seconds", flush=True)
        time.sleep(TAKEOFF_WAIT_S)

        print(f"Publishing waypoint speed: {WAYPOINT_SPEED_MPS:.2f} m/s", flush=True)
        client.publish_waypoint_speed(WAYPOINT_SPEED_MPS)

        print("Publishing waypoint: 1.0 meters forward", flush=True)
        client.publish_relative_waypoint(
            mode="override",
            forward=1.0,
            right=0.0,
            down=0.0,
            threshold_m=0.25,
            hold_time_s=0.0,
        )

        print("Publishing land", flush=True)
        client.publish_autonomy_request("land")


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nStopped by Ctrl+C", flush=True)
        sys.exit(130)
```

The command-line equivalents are:

```bash
nimbusos-arm
nimbusos-autonomy-request takeoff
nimbusos-waypoint-speed 0.45
nimbusos-autonomy-request relative_waypoint --mode override --forward 1.0 --right 0.0 --down 0.0 --threshold 0.25 --hold-time 0.0
nimbusos-autonomy-request land
```
