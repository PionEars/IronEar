# TinyEar-1.0

| | |
|---|---|
| File | `tinyear-1.0.tflite` |
| Base | YAMNet, TensorFlow Models, Google, Apache 2.0 |
| Classes | 521, AudioSet ontology, unchanged |
| Input | 16 kHz mono, 0.975 s window, 64-band log-mel patch 96 x 64 |
| Output | 521 class scores, 1024-dimension embedding |
| Format | TensorFlow Lite, quantized for the ESP32-S3 |
| Size | 4 119 712 bytes |
| Class map | `yamnet_class_map.csv` (index, AudioSet MID, display name) |

Modifications by IronEar Technologies LLC: conversion and quantization for the ESP32-S3. The class set and the network architecture are unchanged.

License: `../licenses/LICENSE-model.md`. Notices: `../licenses/NOTICE-model.md`.
