---
description: Arm and disarm NimbusOS with the Python SDK or nimbusos-arm CLI.
---

# Arming

Arming the drone tells the flight controller that it is allowed to spin the motors.

This is necessary before the drone can accept flight commands.

Arming the drone in the SDK is a single line of code:

```python
from __future__ import annotations

import sys

from nimbusos_sdk import NimbusClient


def main() -> None:
    with NimbusClient() as client:
        print("Publishing arm", flush=True)
        client.publish_arm_state(True)  # This is the arm command.


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nStopped by Ctrl+C", flush=True)
        sys.exit(130)
```

The command-line equivalent is:

```bash
nimbusos-arm
```

To disarm instead, publish `False` or use the CLI flag:

```python
client.publish_arm_state(False)
```

```bash
nimbusos-arm --disarm
```

Once the drone is armed, you can request takeoff with `client.publish_autonomy_request("takeoff")`.
