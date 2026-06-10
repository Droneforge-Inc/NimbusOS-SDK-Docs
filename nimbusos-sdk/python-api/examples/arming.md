# Arming

Arming the drone is telling the flight controller "You are allowed to spin the motors now"

This is necessary for the drone to take in any commands.

Arming the drone in the SDK is a single line of code

```python
from __future__ import annotations

import time

from nimbusos_sdk import NimbusClient


def main() -> None:
    with NimbusClient() as client:
        print("Publishing arm", flush=True)
        client.publish_arm_state(True)       # <---- This the single line

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nStopped by Ctrl+C", flush=True)
        sys.exit(130)
```

Once the drone is armed, you are ready for takeoff.

