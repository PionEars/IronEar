# Notices for the TinyEar-1.0 model

TinyEar-1.0 is derived from YAMNet, an audio event classifier published by
Google in the TensorFlow Models repository
(https://github.com/tensorflow/models/tree/master/research/audioset/yamnet).

Copyright 2019 The TensorFlow Authors. Licensed under the Apache License,
Version 2.0. Modifications by IronEar Technologies LLC: conversion and
quantization of the network for execution on the ESP32-S3 microcontroller,
and removal of the log-mel front end from the graph (it runs outside the
model). No changes were made to the class set.

The class names in `model/yamnet_class_map.csv` come from the AudioSet
ontology (https://research.google.com/audioset/), Copyright Google LLC,
licensed under the Creative Commons Attribution 4.0 International License
(https://creativecommons.org/licenses/by/4.0/).
