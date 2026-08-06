# Firmware bundle — simple-sensor

Pre-built firmware for the ultrasonic water-level sensor kit. These are the
bins the setup page (`index.html`) serves for flashing.

## Files

| File | Board | Role |
|------|-------|------|
| `rook_sender_ultrasonic_v3.uf2` | Rook v4 (nRF52840) | Sender — drag onto NICENANO drive |
| `heltec_v3_receiver_ultrasonic_v3.bin` | Heltec WiFi LoRa 32 V3 | Receiver — app only (safe update) |
| `heltec_v3_receiver_ultrasonic_v3_merged.bin` | Heltec V3 | Receiver — full flash (tick "Erase device") |
| `heltec_v4_receiver_ultrasonic_v3.bin` | Heltec WiFi LoRa 32 V4 (ESP32-S3) | Receiver — app only |
| `heltec_v4_receiver_ultrasonic_v3_merged.bin` | Heltec V4 | Receiver — full flash |

## Provenance

Built from the [`MeshCore-simple-sensor`](https://github.com/edgecollective/MeshCore-simple-sensor)
repo at:

- **SHA:** `3e9796ff`
- **Tag:** `snapshot/2026-08-06_v3-ultrasonic`
- **Source snapshot dir:** `firmware/2026-08-06_v3-ultrasonic/` in that repo
  (includes the same bins alongside a build-flags README)

To reproduce, clone that repo, `git checkout snapshot/2026-08-06_v3-ultrasonic`,
and run:

```bash
pio run -e Rook_companion_sensor_v3_ultrasonic
pio run -e Heltec_v3_companion_sensor_receiver_v3_ultrasonic
pio run -e heltec_v4_companion_sensor_receiver_v3_ultrasonic -t mergebin
```

## Version bump procedure

When publishing a new firmware version:

1. Build in `MeshCore-simple-sensor` and copy the fresh bins into
   `firmware/YYYY-MM-DD_v3-ultrasonic/` in that repo, alongside a README
   noting the source SHA, envs, and build flags.
2. Tag the source commit: `git tag -a snapshot/YYYY-MM-DD_v3-ultrasonic -m ...`
   Push tags with `git push --tags`.
3. Copy the bins here (`firmware/simple-sensor/`) with a bumped `_vN` suffix
   in the filename.
4. Update the `name` and `title` fields in `simple-sensor.json` to match.
5. Update the **Provenance** section above (SHA + tag).

Keeping the snapshot-dir + git-tag pair in the source repo means the
served bins are always reproducible from source, even after main has moved on.
