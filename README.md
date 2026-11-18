# DreamShield Server v0.1.6

The DreamShield server ingests thermal frames and bedside temperature and humidity readings. It runs FusionNet-50 and a random forest locally to assess blanket coverage, collects evidence over time, and can request an independent vision-language model (VLM) review. Temperature-control rules combine the assessment with SHT31 readings before requesting an infrared command from the Freenove ESP32-S3. The server also hosts the desktop web console and the HTTPS gateway used by the mobile apps.

## Companion repositories

| Repository | Compatible release | Role |
| --- | --- | --- |
| DreamShield Server (this repository) | v0.1.6 | Local inference, review, temperature control, and the mobile gateway |
| [DreamShield Freenove firmware](https://github.com/Reizen721/dreamshield-freenove-firmware/tree/v0.9.3) | v0.9.3 | ESP32-S3 control node for SHT31 telemetry and infrared emission |
| [DreamShield Android](https://github.com/Reizen721/dreamshield-android) | v0.6.1 | Native Android app built with Kotlin, Java, and Jetpack Compose |
| [DreamShield iOS](https://github.com/Reizen721/dreamshield-ios) | v0.1.1 | Native SwiftUI app |

Both apps pair with the same HTTPS gateway. See the [mobile API contract](docs/mobile-api.md) for pairing payloads and routes, and the [compatibility matrix](docs/compatibility.md) for release tags and verification coverage.

## Run on Windows

From the repository root, create a 64-bit Python 3.12 environment, install the pinned dependencies, and start the server.

~~~powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\start_server.cmd
~~~

- [Mobile pairing](http://127.0.0.1:8016/mobile-setup)
- [Thermal console](http://127.0.0.1:8016/thermal-setup)
- Mobile HTTPS gateway port: `8746` by default. Set `DREAMSHIELD_MOBILE=0` before launch to run only the desktop console.
- Default data directory: `%LOCALAPPDATA%\DreamShield016Core\data`. Set `DREAMSHIELD_DATA_DIR` to an absolute path to override it.

Temporal review and automatic temperature control are off by default. To configure VLM review, set `VLM_PROVIDER` to `openai` or `deepseek` and set the corresponding API key environment variable. You can also point `DREAMSHIELD_VLM_KEY_FILE` to a key file outside the source tree.

## Source map

| Path | Responsibility |
| --- | --- |
| `backend/app/server.py` | HTTP service, device and configuration APIs, and task lifecycle |
| `backend/app/senxor_tcp.py`, `backend/app/thermal_capture.py` | Waveshare thermal input and frame management |
| `backend/model/server_worker.py`, `backend/model/predict_optimized.py` | Model process, preprocessing, and local inference |
| `backend/app/thermal_temporal.py`, `backend/app/thermal_review_scheduler.py`, `backend/app/thermal_closed_loop.py` | Time-weighted evidence, key frames, review budgets, and the control loop |
| `backend/app/control_policy.py`, `backend/app/ac_control.py` | Environmental rules, presets, cooldowns, event history, and device requests |
| `backend/app/mobile_gateway.py` | Mobile pairing, sessions, TLS, and allowed routes |

Model weights, the random forest, and the empty-bed thermal baseline live under [`backend/model/runs`](backend/model/runs). Use the bundled calibration with the original camera placement and bed region. See the [closed-loop assessment policy (Chinese)](%E9%97%AD%E7%8E%AF%E5%88%A4%E5%88%AB%E7%AD%96%E7%95%A5_CN.md) for the control logic.
