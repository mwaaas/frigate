---
id: faqs
title: Frequently Asked Questions
---

### Fatal Python error: Bus error

This error message is due to a shm-size that is too small. Try updating your shm-size according to [this guide](../frigate/installation.md#calculating-required-shm-size).

### How can I get sound or audio in my recordings? {#audio-in-recordings}

By default, Frigate removes audio from recordings to reduce the likelihood of failing for invalid data. If you would like to include audio, you need to set a [FFmpeg preset](/configuration/ffmpeg_presets) that supports audio:

```yaml
ffmpeg:
  output_args:
    record: preset-record-generic-audio-aac
```

### I can't view events or recordings in the Web UI.

Ensure your cameras send h264 encoded video, or [transcode them](/configuration/restream.md).

You can open `chrome://media-internals/` in another tab and then try to playback, the media internals page will give information about why playback is failing.

### What do I do if my cameras sub stream is not good enough?

Frigate generally [recommends cameras with configurable sub streams](/frigate/hardware.md). However, if your camera does not have a sub stream that a suitable resolution, the main stream can be resized.

To do this efficiently the following setup is required:
1. A GPU or iGPU must be available to do the scaling.
2. [ffmpeg presets for hwaccel](/configuration/hardware_acceleration.md) must be used
3. Set the desired detection resolution for `detect -> width` and `detect -> height`.

When this is done correctly, the GPU will do the decoding and scaling which will result in a small increase in CPU usage but with better results.

### My mjpeg stream or snapshots look green and crazy

This almost always means that the width/height defined for your camera are not correct. Double check the resolution with VLC or another player. Also make sure you don't have the width and height values backwards.

![mismatched-resolution](/img/mismatched-resolution-min.jpg)

### "[mov,mp4,m4a,3gp,3g2,mj2 @ 0x5639eeb6e140] moov atom not found"

These messages in the logs are expected in certain situations. Frigate checks the integrity of the recordings before storing. Occasionally these cached files will be invalid and cleaned up automatically.

### "On connect called"

If you see repeated "On connect called" messages in your logs, check for another instance of Frigate. This happens when multiple Frigate containers are trying to connect to MQTT with the same `client_id`.

### Error: Database Is Locked

SQLite does not work well on a network share, if the `/media` folder is mapped to a network share then [this guide](../configuration/advanced.md#database) should be used to move the database to a location on the internal drive.

### Unable to publish to MQTT: client is not connected

If MQTT isn't working in docker try using the IP of the device hosting the MQTT server instead of `localhost`, `127.0.0.1`, or `mosquitto.ix-mosquitto.svc.cluster.local`.

This is because, by default, Frigate does not run in host mode so localhost points to the Frigate container and not the host device's network.

### How do I know if my camera is offline

A camera being offline can be detected via MQTT or /api/stats, the camera_fps for any offline camera will be 0.

Also, Home Assistant will mark any offline camera as being unavailable when the camera is offline
