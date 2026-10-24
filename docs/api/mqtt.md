# Home Assistant

Firmware 9.0.0. The board is an MQTT client. It publishes under `<prefix>/<node>/`, with `prefix` `ironear` and `node` the board's serial unless changed in the broker settings. Discovery messages under `homeassistant/` create one device per board. Nothing is sent while the broker is not configured.

## Topics the board publishes

| Topic | Retained | Payload | When |
|---|---|---|---|
| status/online | yes | `online` or `offline`; `offline` is also the will | on connect, on a clean stop, by the broker when the connection dies |
| alert | no | `{event_type, name, label, conf, snr, db, kind, room}` | when a trigger, filter or scene routed to Home Assistant fires |
| last_alert | yes | `{t, name, label, conf, snr, kind}` | with every alert |
| activity | yes | `ON` or `OFF` | sound above the room floor started or ended |
| sound | yes | `{class, conf, db, floor}` | class change, confidence moved by 10, level moved by 2 dB, or every 60 s; `class` is empty while the room is quiet |
| infrasound | yes | `{db, floor, over, peak}` | 10 to 40 Hz band, on change |
| infrasound_low | yes | `{db, floor, over, peak}` | 10 to 20 Hz band, on change |
| diag | yes | `{batt_pct, batt_v, charging, usb, rssi, uptime_s, fw, lora, channel, gps_fix, sats, gps_src, heap}` | every 60 s and when charging, USB or the radio change |
| position | yes | `{latitude, longitude, gps_accuracy}` | moved more than 5 m or every 10 min, only with a fix |
| motion | yes | `ON` or `OFF` | presence sensing detected or cleared motion |
| scene/{slug}/state | yes | `{on, score}` | a learned scene routed to Home Assistant turned on or off |
| ctrl | yes | `{lora, protocol, presence, light}` | after a command and within 5 s of a change made elsewhere |
| report | no | `{serial, name, id, t, label, kind, conf, snr, db, floor, idx, lat, lon, fix, rssi, motion, peer}` | with every alert, for other software |
| peer_report | no | the same record for an alert relayed from a board without a broker | per relayed alert |
| peer/{id} | yes | `{serial, id, batt_mv, uptime_min, rssi, t}` | a heartbeat from another board: every 60 s over LoRa, every 7 s over the house link |
| fleet_nodes | yes | `{n}` | master only, every 60 s |
| relay/{serial}/alert, health, position, scene/{slug} | yes | `{event_type, name, kind, conf, snr, db, spl, t, node}`, `{t, seen_s, batt_pct, batt_v, gps_fix, uptime_min}`, `{latitude, longitude, gps_accuracy}`, `{on, name}` | master only, for boards heard over the mesh that have no broker; discovery creates a device "<name> (via <master>)" with Last alert, Last alert time, Last heard, GPS fix, Battery, Position, an Alert event typed by the alert name like the master's own, and one binary sensor per scene |

## Entities created by discovery

| Entity | Type | Source topic |
|---|---|---|
| Alert | event | alert; `event_type` is the trigger or filter name |
| Last alert, Last alert time | sensor | last_alert |
| Sound activity | binary sensor | activity, turns off 30 s after the last message |
| Detected sound, Sound level est. | sensor | sound; the level is dBFS plus 120 |
| Infrasound 10-40 Hz est., Infrasound 10-20 Hz est. | sensor | infrasound, infrasound_low |
| Battery, Charging, USB power, Wi-Fi signal, Uptime, Firmware, LoRa mesh, Mesh channel set, GPS fix, GPS satellites | sensor and binary sensor | diag |
| Position | device tracker | position |
| Motion | binary sensor | motion, only while presence sensing is on, turns off after 60 s |
| one binary sensor per learned scene | binary sensor | scene/{slug}/state, turns off after 90 s |
| LoRa radio | switch | ctrl |
| Radio protocol, Presence sensing, Indicator light | select | ctrl |
| Setup access point, Radio test, Recalibrate mics, Reboot | button | none |
| Boards heard | sensor | fleet_nodes, master only |

## Commands the board accepts

Topic `<prefix>/<node>/cmd/<key>`, one payload per message. A message that arrives retained is ignored. A button acts at most once per 30 s.

| key | Payload | Effect |
|---|---|---|
| lora | `ON` or `OFF` | radio on or off; needs a channel |
| protocol | `Meshtastic` or `MeshCore` | protocol for the next switch-on |
| presence | `off`, `slow` or `fast` | presence sensing mode |
| light | `normal`, `dim` or `off` | indicator light |
| ap | `PRESS` | setup access point for 10 min |
| radio_test | `PRESS` | one test line on the channel |
| recalibrate | `PRESS` | microphone gain equalisation |
| reboot | `PRESS` | reboot after 2 s |

## Broker settings

| Field | Meaning |
|---|---|
| host, port | broker address, `/api/mqtt/discover` finds a Home Assistant instance on the home network |
| user, pass | broker credentials, the password is stored and never read back |
| tls | `true` for MQTT over TLS on port 8883 with the board's certificate bundle |
| prefix, node | topic prefix and board name in topics |
