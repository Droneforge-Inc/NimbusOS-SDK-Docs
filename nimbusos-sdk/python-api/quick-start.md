---
description: Short current examples for creating a NimbusClient, subscribing, publishing, camera access, and CLI smoke tests.
---

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
    for telemetry in client.telemetry(timeout_sec=1.0):
        print(telemetry.seq, telemetry.t_ns)
        break
```

Exiting the `with` block closes any publisher socket owned by that client.

## Publish a basic command

Arm NimbusOS and request takeoff:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_arm_state(True)
    client.publish_autonomy_request("takeoff")
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

## Subscribe to selected state

Selected state includes local-frame position, velocity, attitude, and orientation:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for state in client.selected_state(timeout_sec=5.0):
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

## Publish a relative waypoint

Set the waypoint path speed and send an override waypoint in the drone body frame:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_waypoint_speed(0.45)
    client.publish_relative_waypoint(
        mode="override",
        forward=1.5,
        right=0.0,
        down=0.0,
        threshold_m=0.15,
        hold_time_s=0.0,
    )
```

`threshold_m` is the distance from the target that counts as reached. `hold_time_s` is the time to hold after reaching the waypoint.

## Use the command line

The package also installs CLI tools for quick smoke tests and manual commands:

```bash
nimbusos-subscribe telemetry --limit 1 --timeout 5
nimbusos-subscribe selected_state --limit 1 --timeout 5
nimbusos-arm
nimbusos-autonomy-request takeoff
nimbusos-autonomy-request relative_waypoint --mode override --forward 1.5 --right 0.0 --down 0.0
nimbusos-waypoint-speed 0.45
nimbusos-yaw-turn-command 0.52
```
