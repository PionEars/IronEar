# Live stream

Firmware 9.0.0. One WebSocket per client at `ws://<board address>/ws?tok=<token>`. Every frame is a JSON object `{"type", "data"}`. The board sends only; it ignores what a client sends. Eight clients at a time.

| type | When | data |
|---|---|---|
| classify | every classified window, about once per second | `{inf_ms, db, floor, classes[{name, idx, score}]}`, the five best classes |
| audio | 1 s | `{m1, m2, m3, g1, g2, g3, floor, stale}`, level per microphone in dBFS, gain per microphone, room floor |
| infrasound | 1 s | `{db, floor, over, peak}` for 10 to 40 Hz |
| infrasound_low | 1 s | the same for 10 to 20 Hz |
| filtered | when a window passes every pipeline layer | `{label, score}` |
| cluster | when a confusion cluster boosts a class | `{target, hits, boost, fires}` |
| alert | when a trigger, filter or scene fires | `{label, conf, kind, mesh, ha}` |
| scene | when a learned scene matches | `{scene, similarity, learned}` |
| learned_sound | when a taught sound matches | `{name, similarity, src}` |
| learned_motion | when a learned motion matches | `{name, similarity, src}` |
| presence | 1 s while presence sensing is on | `{index, floor, motion, source, frames, peer, bssid, idx_peer, idx_bssid, mode}` |
| report | with every alert | `{serial, name, id, t, label, kind, conf, snr, db, floor, idx, lat, lon, fix, rssi, motion, peer}`, the record the fleet shares |
| status | 5 s | `{batt_v, batt_pct, charging, full, chg, vbus, pg, chg_fault, heap, uptime_s, rssi, lora, channel, motion, fw}` |
| gps | 5 s | `{fix, lat, lon, sats, siv, snr, hdop, alt, src, age_s, clk, gps_sntp_ms, duty}` |
| compass | 5 s | `{heading, mx, my, mz, ox, oy, oz}` |
| net | 5 s | `{ip}` |
| mesh | 5 s | `{backend, tx, rx, drops, house{up, peers, tx, rx}, coex{on, req, grant, denied, wifi_tx_in_grant, max_wait_ms}}` |
| fleet | when the fleet list changes | `{changed, id}` |
| node | when a position arrives on the mesh | `{id, fix, lat, lon, rssi}` |
| remote | when a remote board's name, scenes or last alert change | `{id, serial, name, via, seen_s, scenes, last_alert}`, the board's row |
| location | when this board's position changes | `{lat, lon, mode, by}` |
| mesh_text | when a text arrives on the LoRa channel | `{proto, from, rssi, text}` |
| security | sign-in events | `{msg}` |

Levels are dBFS. The pages show them as estimated sound pressure by adding 120 dB.
