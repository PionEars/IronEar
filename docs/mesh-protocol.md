# Mesh protocol

Firmware 9.0.0. Boards share alerts, positions and heartbeats over LoRa and, on the same Wi-Fi channel, over a direct Wi-Fi link called the house link. All of it is keyed by one private channel: a name and a 256-bit key created on one board and joined by the others through the Mesh page or `/api/mesh/channel`; a 128-bit key of an existing Meshtastic channel can be joined too. Without a channel the radio stays off. The channel key is the whole trust boundary: every board that holds it is trusted for what it sends.

## LoRa, Meshtastic framing

The board is a Meshtastic node on the channel. Any Meshtastic device or app that joins the same channel reads the text messages. The Meshtastic frame is encrypted with the channel key but carries no authentication tag, so the leg has confidentiality and no proof of origin. Fleet lines are accepted from port 256 only, never from chat, and a board drops a fleet line whose sequence number does not advance past the last one it accepted from that sender within an hour.

| Item | US | EU |
|---|---|---|
| Frequency | 906.875 MHz | 869.525 MHz |
| Modulation | SF11, 250 kHz, CR 4/5 (LongFast) | same |
| Power | 14 dBm | 14 dBm |
| Region | fixed per firmware image | fixed per firmware image |

| Port | Use | Content |
|---|---|---|
| 1 TEXT_MESSAGE_APP | alert lines, plain text | see below |
| 3 POSITION_APP | this board's position, every 30 s while it holds a fix | standard protobuf |
| 4 NODEINFO_APP | this board's node info, with every tenth position | standard protobuf |
| 256 PRIVATE_APP | fleet lines IE2, IE4 and IE5, plain text, not shown as chat | see below |

Text lines:

| Line | Format | Meaning |
|---|---|---|
| alert | `<board name>: <label> <percent>%` | port 1; a trigger, filtered alert, scene or the sound phrase routed to LoRa; ` <level> dB` is appended when the rule's dB is ticked, ` · <lat>,<lon>` when GPS is ticked; each alert line is followed by an IE4 line |
| IE1 | `IE1 <id> <lat> <lon> <seq>` | position, six decimals; MeshCore framing only, Meshtastic uses port 3 instead |
| IE2 | `IE2 <id> <mac24> <role> <batt_mv> <uptime_min> <flags> <cfg_hash> <ip> <lat> <lon> <seq>` | heartbeat every 60 s; `mac24` six hex digits, `role` 0 remote 1 master, `flags` bit0 GPS fix, bit1 Wi-Fi, bit2 live GPS, bit3 has LoRa, bit4 presence on; `cfg_hash` and `ip` eight hex digits; `-` for a missing coordinate |
| IE4 | `IE4 <id> <kind> <conf> <snr> <db> <seq> <name>` | an alert or a scene state for the fleet pages |
| IE5 | `IE5 <id> <mac24> <seq> <name>|<scene>|<scene>` | a board's name and the scenes it routes over LoRa |

Rate caps: 4 LoRa alert lines per minute per board by default, each with its IE4 line; heartbeat every 60 s; position every 30 s with a fix. A board never transmits Wi-Fi and LoRa at the same time.

## LoRa, MeshCore framing

The same channel can run MeshCore framing instead; one protocol per board, chosen on the Mesh page.

| Item | Value |
|---|---|
| Frequency | 910.525 MHz US, 869.618 MHz EU |
| Modulation | SF7 US, SF8 EU, 62.5 kHz, CR 4/5 |
| Sync word | 0x1424 |
| Group key | the first 16 bytes of the channel key, `mc_key_hex` from `/api/mesh/channel` |
| Routing | flood, type 0x01 |
| Text | GRP_TXT 0x05, the same alert lines as above |
| Data | GRP_DATA 0x06 with data type 0x4945 (`IE`), the IE text lines inside |
| MAC | 2 bytes, payload up to 184 bytes |

## House link

Boards on the same Wi-Fi channel also talk directly, frame by frame, without the router. It carries the binary mesh records below, the alert report record, presence probes, fleet lines and a time stamp. A board without LoRa or without a broker has its alerts relayed by a neighbour that has them.

| Item | Value |
|---|---|
| Transport | ESP-NOW; unicast with retry for alert records, alert reports and placement; broadcast for heartbeat, position, probes, fleet lines and time |
| Key | 128 bits derived from the channel key; boards without the key fail the tag check |
| Cipher | AES-128-GCM, header and sender MAC as associated data, 8 byte tag, nonce from four bytes of the sender MAC, the boot counter and the sequence |
| Header, 16 bytes | magic `IH`, ver u8 (1), type u8, node_id u8, flags u8, ttl u8, len u8, boot u32, seq u32 |
| Types | 1 mesh record, 2 alert report, 3 presence probe, 4 fleet time, 5 place a board, 6 fleet line as text |
| Flags | 0x01 originator has no LoRa, relay it; 0x02 originator has no broker, publish it |
| Replay | per sender, boot and sequence must advance, a 64-frame window rejects repeats; the high-water mark per sender is saved every 60 s and survives a reboot |
| Heartbeat | every 7 s |
| Probes | 10 per second while any peer has presence sensing on |
| Channel rule | a board on the home network uses the router's channel, a board without one uses channel 1; boards on different channels do not link |

Binary mesh records inside house link frames of type 1, little-endian, packed:

| Type | Byte 0 | Fields |
|---|---|---|
| alert | 0x01 | priority u8, class_id u16, confidence u8, energy_db i8, distance_mm u16, scene_state u8, node_id u8 |
| position | 0x06 | node_id u8, lat_e7 i32, lon_e7 i32, gps_fix u8; 0x7FFFFFFF means no fix |
| fleet heartbeat | 0x07 | node_id u8, mac24[3], role u8, batt_mv u16, uptime_min u16, flags u8, cfg_hash u32, ip u32, lat_e7 i32, lon_e7 i32; flags bit0 GPS fix, bit1 Wi-Fi, bit2 live GPS, bit3 LoRa on, bit4 presence on |
