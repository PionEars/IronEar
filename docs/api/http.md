# HTTP API

Firmware 9.0.0. Every board serves the API on port 80 of its address. Bodies and answers are JSON unless a row says otherwise.

| Item | Value |
|---|---|
| Base | `http://<board address>` |
| Session | `X-Auth-Token: <token>` header or `ie_tok` cookie, see [auth.md](auth.md) |
| Body | `Content-Type: application/json` |
| 400 | body missing, not JSON, or a value out of range; the answer names the reason |
| 401 | no session or an expired one |
| 404 | unknown route or store |
| 409 | a run of this kind is already in progress |

## Status and health

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/status | GET | none | `{version, region, build, slot, uptime_s, free_heap, gps{fix, sats, siv, snr, src, age_s}, classify{idx, score}, heading, mesh, wifi{connected, rssi}, batt{v, pct, chg, vbus}, selftest}` |
| /api/state | GET | none | `{gps{fix, lat, lon, sats, hdop, alt, src, age_s, duty}, compass{heading}, uptime_s}` |
| /api/health | GET | none | `{uptime_s, heap, heap_min, heap_int, heap_int_min, heap_int_max_blk, hb{capture, infer, system}, reset_reason, counters{...}, boot_heap{...}, ws, gps, stores, house, lora}`; `hb` are loop counts of the three main tasks, `counters` hold classify, qdrops, bc_fired, bc_supp, nmea_acc, nmea_rej, cluster_fires, cluster_evals, mesh_text_rx, mqtt_cmd, mqtt_cmd_cool, mqtt_drop, wifi_wd_hard and the auth counters |
| /api/node | GET | none | `{serial, name, role, cfg_hash}`; `role` is `remote` or `master`, `cfg_hash` covers pipeline_config and layers |
| /api/node | POST | `{"name", "role"}` with `role` 0 (remote) or 1 (master) | `{ok}` |
| /api/reboot | POST | none | `{ok, rebooting}` |
| /api/coredump | GET | none | the last core dump as an ELF file, binary |

## Sound and pipeline

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/gate | GET | none | `{effective_db, floor_db, margin_db}`; the energy gate opens at `floor_db + margin_db` |
| /api/gate | POST | `{"margin_db"}` | the same object |
| /api/calibrate | POST | none | `{ok}`; equalises the three microphone gains over about 5 s in a quiet room and keeps the result |
| /api/calibrate-bg | POST | `{"seconds"}` | `{ok, started, seconds}`; records the room's background profile, 409 while one runs |
| /api/calibrate-bg | GET | none | `{active:false}` or `{active:true, remaining_s}`; the first call after the window stores the profile and answers `{ok, labels, floor_db}` |
| /api/history | GET | none | `{uptime_s, events[{t_s, db, inf_ms, classes[[name, score]]}]}`; the last classified windows, kept in RAM |
| /api/alerts/recent | GET | none | `{alerts[{age_s, e{label, conf, kind, mesh, ha}}]}` |
| /api/infrasound | GET | none | `{db, floor, over, peak, age_ms, valid, band, hist[]}`; 10 to 40 Hz band level from microphone 1 in dBFS, `hist` one value per second for two minutes |
| /api/infrasound/low | GET | none | the same for 10 to 20 Hz |
| /api/compass | GET | none | `{heading, mx, my, mz, hi[3], calibrating, remaining_s, radius, samples, rejected}` |
| /api/compass/calibrate | POST | none | `{ok}`; 30 s hard-iron calibration, rotate the board through two turns; `rejected` on GET when the rotation was too small |
| /api/battery/calibrate | GET | none | `{ok, scale, v, pct}` |
| /api/battery/calibrate | POST | `{"v"}` measured pack volts | `{ok, scale}` |

## Stores

A store is one JSON document on the board. GET returns it, POST replaces it after validation and the pipeline reloads. The cap is 24 KB. `?prev=1` on GET returns the version before the last save for the two learned stores.

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/store/{name} | GET | none | the document |
| /api/store/{name} | POST | the document | `{ok, cfg_hash}`; 400 with `{ok:false, error, label}` when a trigger or filter names a label the board does not know |
| /api/store/{name}?prev=1 | GET | none | the previous document, `learned_sounds` and `learned_scenes` only |

| Store | Content |
|---|---|
| pipeline_config | `energy_gate_db, gate_db, gain_max, threshold_default, thresholds{class: min score}, voting{class: [hits, window]}, cooldown_default, cooldown_overrides{class: seconds}, clusters[{target, members[], boost, min}], bg_labels[], energy_xval{class: [min dB, max dB]}, alert_scenes` |
| layers | `{energy_gate, class_grouping, confusion_cluster, background_sub, bg_calibration, thresholds, energy_xval, voting, cooldown}`, each `true` or `false` |
| broadcast_config | `triggers[{name, labels[], score, votes, cd, lora, ha, gps, need_motion}], filtered[{name, labels[], lora, ha, gps}], scenes[{name, sim, matches, win, lora, ha, gps}], caps{lora_per_min, ha_per_min, fleet_merge}, password{enabled, name, labels[2..4], score, lora, ha, gps, need_motion}` |
| learned_sounds | one entry per taught sound: `{name: {emb[1024], min_match}}`, written by `/api/learn`, at most 5 |
| learned_scenes | one entry per recorded scene, written by the Learning page |
| bg_profile | `{class: base score}` written by `/api/calibrate-bg` |
| pipeline_presets | named copies of pipeline_config |

A label in `triggers` or `password` is a class name from the model's class map, a group name, a taught sound, a learned motion or `motion`. A label in `filtered` is a class or a group.

## Alerts

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/broadcast/log | GET | none | `{log[{up_s, kind, names, line, lora, ha}], password{on, step, of, left_s}}`; the last 10 broadcasts and the sound phrase's progress |
| /api/broadcast/test | POST | `{"kind", "name"}` with `kind` trigger, scene, filtered or password | `{ok}`; pushes a synthetic event through the real routing |

## Teaching

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/learn | POST | `{"name", "seconds"}` | `{ok, started, seconds}`; opens a capture window for a new sound, 409 while one runs |
| /api/learn | GET | none | `{active:false}` or `{active:true, remaining_s}`; the first call after the window stores the sound and answers `{ok, name, windows}` or `{ok:false, error}` |
| /api/presence/learn | POST | `{"name", "seconds"}` | `{ok}`; records a motion pattern |
| /api/presence/learn | GET | none | `{state, name, seconds_left, error}` with `state` idle, recording, learned or failed |
| /api/presence/learned | GET | none | `{motion[{name, sim}]}` |

## Mesh and fleet

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/mesh/channel | GET | none | `{name, psk_b64, private, mc_key_hex}`; the shared channel key, `mc_key_hex` is its first half for the MeshCore app |
| /api/mesh/channel | POST | `{"generate":true, "name"}` or `{"name", "psk_b64"}` with a 16 or 32 byte key | the same object |
| /api/mesh | POST | `{"backend":"lora"}`, `{"backend":"meshcore"}`, `{"backend":"off"}` or `{"proto":"meshtastic"}`, `{"proto":"meshcore"}` | `{ok, backend}`; the radio stays off without a private channel |
| /api/mesh/region | GET | none | `{region, freq_hz, live, proto, phy, backend, switched}` |
| /api/lora-test | POST | none or `{"text"}` up to 160 characters | `{ok, sent, channel, private}`; sends a text to the channel |
| /api/fleet | GET | none | `{self{serial, name, role, cfg_hash}, nodes[{serial, id, name, role, batt_v, uptime_min, gps_fix, wifi, ip, cfg_hash, lat, lon, rssi, hb_age_s, link, has_lora, house_rssi, house_age_s, house_ch, via, seen_s, last_alert{name, kind, conf, snr, db, age_s}, scenes[{name, on, age_s}]}], decision{label, room, serial, conf, snr, db, spl, n, margin_db, ambiguous, candidates[]}}` |
| /api/fleet/sync | POST | `{"ip", "pass"}` or `{"ip", "token"}` | `{ok, started}`; starts a push of this board's pipeline_config and layers into the board at `ip` on a background task, after signing in to it with the target's device password (used once, wiped after the login) or a session token of it; 409 while a push runs, refused for this board's own address |
| /api/fleet/sync | GET | none | `{busy, ok, synced, status[2], error, age_s}` of the last push |
| /api/fleet/place | POST | `{"lat", "lon"}`, `{"id", "lat", "lon"}`, `{"mode":"auto"}` or `{"id", "mode":"auto"}` | `{ok, id, mode, via}`; places this board or a fleet board on the map, `auto` returns it to GPS; 409 when the other board is not on the house link |
| /api/house | GET | none | `{enabled, up, channel, associated, linked_peers, tx, rx, drops, auth_fail, replay, relay_lora, relay_mqtt, probes, time{src, node, age_s}, presence, csi{on, frames, drops, source}, index, floor, level, motion, motion_age_s, motion_events, peers[{serial, rssi, age_s, lora, presence, ch}]}` |
| /api/house | POST | `{"enabled"}` or `{"presence"}` with `off`, `slow` or `fast` | the same object |
| /api/gps/resurvey | POST | `{"lat", "lon"}` or `{"mode":"auto"}` | `{ok}`; places the board by hand or returns it to GPS |
| /api/gps/debug | GET | none | last sentence per type, decode counts, satellites, receiver state |
| /api/gps/raw | GET | none | the last 64 NMEA sentences, text |

## Network

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/wifi/status | GET | none | `{connected, ip, host, ssid, ap_clients, ap_up, ap_seconds, ap_key_src, offgrid}` |
| /api/wifi/scan | GET | none | `[{ssid, rssi, auth}]` |
| /api/wifi/connect | POST | `{"ssid", "pass"}` | `{ok}`; saves the network and connects |
| /api/wifi/forget | POST | `{"confirm":true}` | `{ok, rebooting}`; drops the saved network and restarts as its own access point, settings kept |
| /api/wifi/offgrid | POST | `{"offgrid"}` | `{ok, offgrid}`; marks the board as used without a home network |
| /api/wifi/ap | POST | `{"minutes"}` 1 to 480 | `{ok}`; opens the setup access point for that long |
| /api/net/static | GET | none | `{mode, ip, gw, mask, auto}` |
| /api/net/static | POST | `{"ip", "gw", "mask"}` or `{"ip":""}` | `{ok}`; fixed address from the next boot, empty `ip` releases to DHCP |
| /api/mqtt/discover | GET | none | `{sta, candidates[{kind, name, ip, port}]}`; finds a Home Assistant instance or a broker on the home network |
| /api/mqtt | GET | none | `{enabled, host, port, user, pass_set, prefix, node, tls, connected}` |
| /api/mqtt | POST | `{"enabled", "host", "port", "user", "pass", "prefix", "node", "tls"}`, any subset | `{ok}`; the password changes only when sent |

## Settings

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/led | GET | none | `{mode}` |
| /api/led | POST | `{"mode"}` with `normal`, `dim` or `off` | `{mode}`; the light is dark on battery in every mode |
| /api/power/off | POST | `{"confirm":true}` | `{ok}`; ship mode, the board turns off until USB power is applied |
| /api/factory-reset | POST | `{"confirm":true}` or `{"dry":true}` | `{ok, removed, wifi_cleared}`; `dry` lists what would be removed. Removes the stores, settings, password and the saved network, then reboots as a new board |

## Updates

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/ota | POST | the app image, binary | `{ok, bytes, boot, rebooting}`; the image must carry a valid signature and a region tag, the board reboots into it and falls back when the new image fails its health check |
| /api/ota/url | POST | `{"url"}` | `{ok}`; pulls the image over HTTPS and flashes it |
| /api/ota/url | GET | none | `{busy, url, last, last_age_s}` |
| /api/ota/storage | POST | the page image, binary | `{ok, bytes, staged, stage_fail, rebooting}`; replaces the pages and baked stores, user stores are kept |

## Production test

| Route | Method | Body | Answer |
|---|---|---|---|
| /api/emc/worst-case | GET | none | `{on, unlimited, remaining_s, udp_frames, lora_tries, lora_sent}` |
| /api/emc/worst-case | POST | `{"on", "minutes"}`, `minutes` 0 for until stopped | the same object; Wi-Fi at 110 kbit/s, one LoRa transmission every 15 s, the radio switched on for the run |
