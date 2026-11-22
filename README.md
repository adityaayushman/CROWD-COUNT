# CrowdCount

Real-time people counting and zone occupancy from video. YOLOv8 detects people, BoT-SORT tracks them, and a FastAPI dashboard shows live counts, heatmaps and threshold alerts, with CSV and PDF export.

## Features

- **Detection and tracking:** YOLOv8 person detection with BoT-SORT tracking IDs (`detector/integrated_detector.py`).
- **Polygonal zones:** per-zone occupancy from `zones.json`, with an interactive editor (`zone_manager.py`).
- **Live dashboard:** counts, zone occupancy, history and a heatmap over a WebSocket (`/ws`) and REST endpoints (`/count`, `/zones`, `/history`, `/heatmap`, `/summary`).
- **Threshold alerts:** global and per-zone limits, recorded when a threshold is crossed (`/alerts`).
- **Exports:** CSV and PDF reports (`/export/csv`, `/export/pdf`).
- **Access control:** JWT login with roles, plus admin endpoints for users, cameras, zones, thresholds, activity logs and retention.
- **Run metrics:** an optional logger writes per-frame timing, detection confidence and zone time series to CSV for analysis (`--metrics-out`).

## How it fits together

```
video source -> detector (YOLOv8 + BoT-SORT) -> shared state -> FastAPI -> dashboard (frontend/)
                                                      \-> logs, alerts, exports
```

## Quick start

Requires Python 3.10 or later.

```bash
python -m venv .venv
.venv\Scripts\activate          # macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python run_app.py --source path/to/video.mp4
```

The dashboard is served at `http://localhost:8080/static/index.html`. The YOLOv8 weights (`yolov8m.pt`) are downloaded by Ultralytics on first run. No sample video is included, so pass your own with `--source`.

Useful options: `--model`, `--zones`, `--conf`, `--min-area`, `--iou`, `--speed`, `--port`, `--no-display`, `--server-only`, `--detector-only`, `--metrics-out`, `--max-frames`. Run `python run_app.py --help` for the full list.

## Configuration

| Variable | Purpose |
|---|---|
| `CROWDCOUNT_ADMIN_PASSWORD` | Password for the first `admin` account. If unset, a random one is generated and printed once on first run. |
| `JWT_SECRET_KEY` | Signing key for login tokens. If unset, a random key is generated on each start, so sessions end when the server restarts. |

## Project layout

```
run_app.py                       starts the API server and the detector together
people_detector_bytetrack.py     standalone detector (uses BoT-SORT despite the file name)
detector/                        integrated detector and run-metrics logger
backend/                         FastAPI app, auth, admin API, activity logging
frontend/                        dashboard, admin and login pages
zone_manager.py, zones.json      zone editor and zone definitions
shared_state.py                  thread-safe state shared by detector and API
paper/                           scripts that generate figures from run logs
tests/                           API smoke tests
```

## Tests

```bash
python -m unittest discover -s tests
```

The API tests use Starlette's test client, which needs its HTTP client dependency installed (install the package named in the import error if it is missing).

## Limitations

- No accuracy figures are included. Counts depend on the video, camera angle and the `--conf` and `--min-area` settings.
- State is held in memory and users, logs and configuration are stored in JSON files, so it is built for a single instance, not a multi-server deployment.
- The detector expects a video file or camera index. No sample footage is bundled.

## License

MIT. See `LICENSE`.
