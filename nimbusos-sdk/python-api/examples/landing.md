---
description: Request landing through the current NimbusOS autonomy request API.
---

# Landing

Landing is requested through the autonomy request API.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_autonomy_request("land")
```

The command-line equivalent is:

```bash
nimbusos-autonomy-request land
```

Use this only against a safe vehicle, simulator, or controlled NimbusOS test environment.
