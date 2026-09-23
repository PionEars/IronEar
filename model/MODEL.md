# TinyEar-1.0

| | |
|---|---|
| File | `tinyear-1.0.tflite` |
| Base | YAMNet, TensorFlow Models, Google, Apache 2.0 |
| Classes | 521, AudioSet ontology, unchanged |
| Input | int8 log-mel patch, 96 frames x 64 bands, from 0.975 s of 16 kHz mono audio (see below) |
| Output | 521 class scores, 1024-dimension embedding |
| Format | TensorFlow Lite, quantized for the ESP32-S3 |
| Size | 4 119 712 bytes |
| Class map | `yamnet_class_map.csv` (index, AudioSet MID, display name) |

Modifications by IronEar Technologies LLC: conversion and quantization for the ESP32-S3, and the log-mel front end removed from the graph. The file contains the network from the mel patch onward. The class set and the layers after the front end are unchanged.

## Front end

The file expects the patch the IronEar engine computes on the device. To feed it from your own code:

| | |
|---|---|
| Sample rate | 16 000 Hz, mono, float in -1..1 |
| Frame | 400 samples (25 ms), Hann window, FFT 512, magnitude spectrum |
| Hop | 160 samples (10 ms); 96 frames per patch |
| Mel | 64 bands, YAMNet filterbank, 125 to 7500 Hz |
| Log | ln(mel + 0.001) |
| Quantization | int8 = round(log / 0.048020608723163605) + 16, clamped to -128..127 |

License: `../licenses/LICENSE-model.md`. Notices: `../licenses/NOTICE-model.md`.
