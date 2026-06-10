# Waypoints

In the Droneforge SDK we speak to the drone in waypoints. These waypoints are in the NED coordinate convention (North, East, Down) and is RELATIVE to the drone body.&#x20;



In simple terms think the following

| Axis      | Movement         |
| --------- | ---------------- |
| North +/- | Forward/Backward |
| East +/-  | Right/Left       |
| Down +/-  | Down/Up          |

It's important to note that going UP requires a negative DOWN value.&#x20;

The simplest way to send a waypoint in the SDK is to call

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



Threshold gives a padding of success, the waypoint is an exact coordinate and it's rare the drone will ever be 100% where commanded, this gives it some tolerance.

Hold Time tells how long it should hold at that position before taking in the next waypoint, although this will be easiest to do with time.sleep in python.

Example combining [arming.md](arming.md "mention"), [takeoff.md](takeoff.md "mention"), the waypoint and a [landing.md](landing.md "mention")request&#x20;

```python
from __future__ import annotations

import time

from nimbusos_sdk import NimbusClient

TAKEOFF_WAIT_S = 10.0
ARM_WAIT_S = 3.0

def main() -> None:
    with NimbusClient() as client:
        print("Publishing arm", flush=True)
        client.publish_arm_state(True)

        time.sleep(ARM_WAIT_S)
        print("Publishing takeoff", flush=True)
        client.publish_autonomy_request("takeoff")

        print(f"Waiting {TAKEOFF_WAIT_S:.0f} seconds", flush=True)
        time.sleep(TAKEOFF_WAIT_S)

        print("Waiting 10 seconds after takeoff", flush=True)
        time.sleep(10.0)

        # Waypoint command to move ~1 meter forward.
        print(f"Publishing waypoint 1: {1.0:.1f} meters forward", flush=True)
        client.publish_relative_waypoint(
            mode="override",
            forward=1.0,
            right=0.0,
            down=0.0,
            threshold_m=0.25,
            hold_time_s=0.0,
        )

        print("Publishing land", flush=True)
        client.publish_guidance_request("land")


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nStopped by Ctrl+C", flush=True)
        sys.exit(130)

```
