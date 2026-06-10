# Landing

This tells the drone to go down towards the floor until it's about 10cm above the surface. The drone will then disarm, and the flight will be effectively stopped.

This can be called with a single line

```python
client.publish_guidance_request("land")
```

