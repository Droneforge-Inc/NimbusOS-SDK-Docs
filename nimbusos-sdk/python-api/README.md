# Python API

## Table of Contents

* [Setup](setup.md)
* [Quick Start](quick-start.md)
* [Publishing](publishing.md)
* [API](api.md)
* [Subscriptions](subscriptions.md)
* [CLI](cli.md)



The NimbusOS Python SDK is a small public wrapper around the supported NimbusOS V1 ZeroMQ pub/sub topics.

Use it when a Python application, demo, smoke test, or integration needs to observe NimbusOS state or send supported high-level commands.

## What you get

The package includes:

* `NimbusClient`, the main class for publishing commands and subscribing to messages.
* Typed Python dataclasses for common receive workflows.
* Command-line tools for smoke testing and manual command publishing.

## Supported subscriptions

Use typed helpers for application code. They return stable Python dataclasses.

| Topic             | Helper                        | Description                                                      |
| ----------------- | ----------------------------- | ---------------------------------------------------------------- |
| `telemetry`       | `client.telemetry()`          | Battery, attitude, and link telemetry.                           |
| `state`           | `client.state()`              | Local-frame position, velocity, attitude, and orientation state. |
| `camera`          | `client.camera_frames()`      | Core-selected JPEG camera stream.                                |
| `live_camera`     | `client.live_camera_frames()` | Raw live camera JPEG stream before core camera-source selection. |
| `waypoint_status` | `client.waypoint_status()`    | Active waypoint progress and reached/held status.                |

## Supported publications

Publishing methods encode NimbusOS command messages and send them to the configured publish endpoint.

| Topic              | Method                              | Description                                                  |
| ------------------ | ----------------------------------- | ------------------------------------------------------------ |
| `arm_state`        | `client.publish_arm_state()`        | Publish armed or disarmed state.                             |
| `guidance_request` | `client.publish_guidance_request()` | Request `go`, `land`, `return_home`, or a relative waypoint. |
| `waypoint_command` | `client.publish_waypoint_command()` | Publish an override or queued waypoint command.              |
| `yaw_turn_command` | `client.publish_yaw_turn_command()` | Publish a relative yaw turn in radians.                      |

## Connection model

NimbusOS publishes and receives messages over ZeroMQ. By default, the SDK connects to a local NimbusOS instance.

| Direction | Default endpoint       |
| --------- | ---------------------- |
| Publish   | `tcp://127.0.0.1:7771` |
| Subscribe | `tcp://127.0.0.1:7772` |

Override these endpoints with environment variables or explicit `NimbusClient` constructor arguments.
