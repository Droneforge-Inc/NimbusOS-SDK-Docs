---
description: Install nimbusos-sdk and verify the current Python imports and command-line tools.
---

# Setup

This page shows how to install the SDK and configure its ZeroMQ endpoints.

## Requirements

`nimbusos-sdk` requires:

* Python `>=3.10,<4.0`.
* A running NimbusOS instance for live publish/subscribe workflows.

## Install from PyPI

{% tabs %}
{% tab title="pip" %}
Install the package with `pip`:

```bash
pip install nimbusos-sdk
```
{% endtab %}

{% tab title="uv" %}
If your project uses `uv`, add it as a dependency instead:

```bash
uv add nimbusos-sdk
```
{% endtab %}
{% endtabs %}

## Verify the install

Run the import check and confirm that the command-line tools are available:

```bash
python -c "from nimbusos_sdk import NimbusClient; print(NimbusClient)"
nimbusos-subscribe --help
nimbusos-arm --help
nimbusos-autonomy-request --help
nimbusos-waypoint-speed --help
nimbusos-yaw-turn-command --help
```

## Examples

Check these examples out to learn the fundamental actions to make your drone autonomy better.

{% content-ref url="examples/" %}
[examples](examples/)
{% endcontent-ref %}
