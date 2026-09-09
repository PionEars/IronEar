# Third-party software notices

The IronEar firmware is built with the following third-party software. Each
component keeps its own license; the texts are available from the linked
projects.

| Component | Use | License |
|---|---|---|
| ESP-IDF (Espressif) | operating system, drivers, network stack, mbedTLS, lwIP | Apache 2.0 (mbedTLS: Apache 2.0; lwIP: BSD 3-Clause) |
| esp-tflite-micro (Espressif, TensorFlow) | TensorFlow Lite Micro runtime | Apache 2.0 |
| esp-nn (Espressif) | neural network kernels, with a local alignment fix | Apache 2.0 |
| esp-dsp (Espressif) | FFT for the mel spectrogram front end | Apache 2.0 |
| esp-mqtt (Espressif) | MQTT client for Home Assistant | Apache 2.0 |
| mdns (Espressif) | local network discovery | Apache 2.0 |
| led_strip (Espressif) | status LED driver | Apache 2.0 |
| cJSON (Dave Gamble and contributors) | JSON parsing | MIT |
| argtable3 (test firmware only) | console argument parsing | BSD 3-Clause |
| PHY certification test commands from ESP-IDF examples (test firmware only) | RF test console | Unlicense or CC0 1.0 |

The Meshtastic compatible framing in `firmware/main/mesh/` is an independent
implementation of the Meshtastic packet format written for interoperability
and contains no code from the Meshtastic project.

## Third-party trademarks

Meshtastic is a registered trademark of Meshtastic LLC. Home Assistant is a
registered trademark of the Open Home Foundation. Espressif and ESP32 are
trademarks of Espressif Systems. IronEar is not affiliated with or endorsed
by any of them. The names are used only to describe compatibility.

