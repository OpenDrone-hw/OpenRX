OpenRX-Gemini firmware (ExpressLRS 4.0, dual-LR1121) — 2026-07-08
Bind phrase: stan123   |   CRSF baud: 420000   |   WiFi AP after 60s idle: SSID stan123 / pw stan12345
DEBUG_LOG off (required — debug shares UART0 with CRSF).

Your TX must use bind phrase "stan123" (or ask for a rebuild with a different phrase).
Antennas: single-band is fine for testing — put a 2.4GHz (or 900) antenna on ONE U.FL; that chain links,
the diversity uses the better radio. Attach the antenna BEFORE powering (PA keys telemetry on link).

FIRST FLASH (virgin board — UART, hold BOOT/U9 low at a clean 5V power-on, release):
  esptool --chip esp32c3 --port <PORT> --baud 460800 --before no_reset --after no_reset \
    write_flash -z --flash_mode dio --flash_freq 40m --flash_size detect \
    0x0 bootloader.bin  0x8000 partitions.bin  0xe000 boot_app0.bin  0x10000 firmware.bin
  Then full power-cycle (caps drained) to run.

ALREADY RUNNING ELRS: OTA just firmware.bin via the WiFi web UI (or
  curl -H "X-FileSize:<bytes>" -F "file=@firmware.bin" http://<ip>/update ).

Note: this is fork firmware (bring-up bridge). Shipping plan is a hardware respin to run stock ELRS —
power the TCXO from +3V3, and (Gemini) move radio-1 off GPIO0/GPIO2 or respin on ESP32-S3. See BRINGUP.md.
