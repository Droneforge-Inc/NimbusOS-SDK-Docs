# Takeoff

"Takeoff" in the Droneforge ecosystem means "take off and hover". This requires [arming.md](arming.md "mention")the drone first, and once armed you request the autonomy system to send the drone up!

This defaults to 1 meter off the surface the drone took off from.&#x20;

```python
from __future__ import annotations

import time
import sys

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

