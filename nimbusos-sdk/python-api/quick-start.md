# Quick Start

This page gives short examples for the most common SDK tasks.

{% hint style="info" %}
Live examples require a running NimbusOS instance connected to the configured ZeroMQ endpoints.
{% endhint %}

## Create a client

Use `NimbusClient` as a context manager when possible:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_guidance_request("go")
```

Exiting the `with` block closes the publisher socket owned by that client.

## Publish a basic command

Arm NimbusOS and request normal guided operation:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_arm_state(True)
    client.publish_guidance_request("go")
```

## Subscribe to telemetry

Telemetry includes battery, attitude, and link information:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for telemetry in client.telemetry(timeout_sec=5.0):
        print(f"battery={telemetry.battery.voltage:.2f}V")
        print(f"yaw={telemetry.attitude.yaw_deg:.1f}deg")
```

`timeout_sec` is the total lifetime of the iterator. If no message arrives before the timeout, the iterator ends.

## Subscribe to state

State includes local-frame position, velocity, attitude, and orientation:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for state in client.state(timeout_sec=5.0):
        if not state.valid:
            continue
        print(state.position.x_m, state.position.y_m, state.position.z_m)
```

## Read camera frames

Save the first core-selected camera frame as a JPEG file:

```python
from pathlib import Path

from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for frame in client.camera_frames(timeout_sec=5.0):
        Path("frame.jpg").write_bytes(frame.jpeg)
        print(frame.width, frame.height)
        break
```

Use `client.live_camera_frames()` to read the raw live camera stream instead of the core-selected `camera` stream.

## Publish a waypoint command

Send an override waypoint in the local frame:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_waypoint_command(
        mode="override",
        forward=1.5,
        right=0.0,
        down=-1.0,
        threshold_m=0.15,
        hold_time_s=0.0,
    )
```

`threshold_m` is the distance from the target that counts as reached. `hold_time_s` is the time to hold after reaching the waypoint.

## Use the command line

The package also installs CLI tools for quick smoke tests and manual commands:

<pre class="language-bash"><code class="lang-bash"><strong>nimbusos-subscribe telemetry --limit 1 --timeout 5
</strong>nimbusos-arm
nimbusos-guidance-request go
nimbusos-waypoint-command --mode override --forward 1.5 --right 0.0 --down -1.0
nimbusos-yaw-turn-command 0.52
</code></pre>
