---
description: Arm NimbusOS and request takeoff through the current autonomy request API.
---

# Takeoff

"Takeoff" in the Droneforge ecosystem means "take off and hover". This requires [arming.md](arming.md "mention") the drone first. Once armed, request takeoff through the autonomy API.

The current SDK call is `client.publish_autonomy_request("takeoff")`.

```python
from __future__ import annotations

import sys
import time

from nimbusos_sdk import NimbusClient


ARM_WAIT_S = 3.0


def main() -> None:
    with NimbusClient() as client:
        print("Publishing arm", flush=True)
        client.publish_arm_state(True)

        time.sleep(ARM_WAIT_S)
        print("Publishing takeoff", flush=True)
        client.publish_autonomy_request("takeoff")


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
```
