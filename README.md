# IronEar

On-device sound recognition node. ESP32-S3, three PDM microphones, LoRa (Meshtastic compatible), GPS, Home Assistant over MQTT. No cloud.

## Contents

| Directory | Contents | License | Status |
|---|---|---|---|
| `model/` | TinyEar-1.0 weights and class map | Apache 2.0, class names CC BY 4.0 | published |
| `licenses/` | license texts and notices | | published |
| `docs/` | REST and WebSocket API, MQTT topics, mesh protocol | CC BY 4.0 | before campaign launch |
| `hardware/` | schematics, bill of materials; board layout, Gerbers, enclosure STEP | CERN-OHL-S 2.0 | schematics and BOM at campaign launch; layout, Gerbers, enclosure when units ship |
| `firmware/` | firmware source | GPLv3 with linking exception | before units ship |
| `library/` | TinyEar inference engine, prebuilt | `licenses/LICENSE-engine.md` | before units ship |

Closed: the inference engine source and the training data.

## Model

TinyEar-1.0 is YAMNet (521 AudioSet classes) converted and quantized for the ESP32-S3. Details in `model/MODEL.md`.

## Contributing

`CONTRIBUTING.md`. Contributors sign `CLA.md` once and sign off commits.

## Trademarks

`licenses/TRADEMARKS.md`.

## Links

Product: https://ironear.io
Campaign: https://www.crowdsupply.com/ironear/ironear
