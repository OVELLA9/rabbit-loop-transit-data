# RL Transit data

This repo holds the data files RL Transit downloads. They are published as GitHub Release assets. No app source code lives here.

| Release | Files | Used by | Built by | How often |
|---|---|---|---|---|
| `gtfs-data` | `manifest.json`, `snapshot.db`, `gtfs.zip` | The app's departure boards and stop search. `gtfs.zip` also feeds the journey planner graph. | A service on the Rabbit Loop server (`rabbit-loop-transit-gtfs`, source in [rabbit-loop-transit](https://github.com/OVELLA9/rabbit-loop-transit) `server/`) | Every 8 hours |
| `osm-data` | `western-australia-latest.osm.pbf` | The journey planner graph | The "Publish OSM Snapshot" workflow in rabbit-loop-transit | Weekly |
| `otp-graph` | `graph.obj` | The journey planner (OpenTripPlanner) on the Rabbit Loop server | The "Build journey planner graph" workflow in this repo (`.github/workflows/build-graph.yml`) | Monday and Thursday, 3am Perth time |

## The journey planner graph

The journey planner needs one large file that combines the street map and the Transperth timetable. Building it takes more memory than the server has, so it is built here and the server only downloads it.

- The workflow downloads the street map and the timetable from the releases above, builds the graph, then tests it before publishing: it loads the graph with 1 GB of memory (what the server gives it) and plans a real trip for the next morning. If either fails, nothing is published and the server keeps the graph it has.
- The server (`otp/start.sh` in rabbit-loop-transit) downloads the graph when it starts, checks once an hour for a newer one, and restarts the planner on it. If a new graph will not load it goes back to the previous one.
- To rebuild it by hand: Actions tab, "Build journey planner graph", Run workflow. It takes about 4 minutes.
- `OTP_VERSION` must be the same in the workflow and in `otp/start.sh`. A graph only loads in the version that built it.

Why this exists: until 5 October 2026 the server built the graph itself. That build had been running out of memory since July and falling back to an old graph without any warning, and the timetable inside it expired.

## Cost

This repo is public, so its workflow minutes are free. The 8-hourly timetable build stays on the server on purpose, because running it on GitHub from the private app repo used up the plan's minutes.
