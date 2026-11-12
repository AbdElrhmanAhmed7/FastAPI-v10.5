# Verification run · 2026-11-12

A dated snapshot of a **full `make check` run plus live-integration probes** on
the committed backend. It is evidence, not a gate: regenerate it any time with
`make check` (and `scripts/playthrough.py --json`). Nothing in the app reads this
file.

## Environment

| | |
| --- | --- |
| Python | 3.11.2 |
| fastapi | 0.162.5 |
| pydantic | 2.14.3 |
| SQLAlchemy | 2.0.57 |
| uvicorn | 0.54.0 |
| httpx | 0.28.1 |
| alembic | 1.20.1 |
| aiosqlite | 0.22.1 |
| pytest | 9.2.0 |
| ruff | 0.17.4 |
| mypy | 2.4.1 |

Installed from `requirements-dev.txt` into a fresh virtualenv; database = SQLite.
`make` itself was unavailable in the verification sandbox, so every target was
executed verbatim, one command at a time.

## Gates

```
== ruff check ==       All checks passed!
== ruff format ==      108 files already formatted
== mypy ==             Success: no issues found in 80 source files
== content ==          15 missions / 95 actions checked – 0 errors, 0 warnings
== audit ==            routes_total 75 · http_published 67 · operations 67
                       websockets 1 · mounts 1 · guarded_operations 15
                       schemas 136 · findings {'info': 24}
                       0 error(s), 0 warning(s)
== contract ==         ✓ docs/openapi.json is up to date (67 operations)
== pytest ==           437 passed, 1 warning in 176.62s
                       TOTAL 6075 statements · 410 missed · 93% coverage
== playthrough ==      ✓ campaign completed (exit 0)
```

The single warning is a Starlette deprecation notice about `httpx` in
`fastapi.testclient`, emitted by the dependency itself.

## Campaign (public HTTP only, `--difficulty standard --seed 7`)

```
MISSION                        SCORE  GRADE  MISTAKES  STEPS
training_prep                 100.0%      S         0      7
analog_systems_check          100.0%      S         0      4
analog_device_fault           100.0%      S         0      6
analog_emergency              100.0%      S         0      8
analog_power_link             100.0%      S         0      6
analog_comms_blackout         100.0%      S         0      5
analog_water_leak             100.0%      S         0      4
analog_water_recycling        100.0%      S         0      6
mars_arrival                  100.0%      S         0      8
mars_rover_traverse           100.0%      S         0      6
crew_health                   100.0%      S         0      9
freefall_fire                 100.0%      S         0      8
martian_guide                 100.0%      S         0      7
lunar_landing                 100.0%      S         0      7
earth_trends                  100.0%      S         0      6

Missions completed : 15 / 15      Readiness : 100.0 (S)
Average score      : 100.0%       Playtime  : 259.5 s
Actions            : 93 ok / 0 mistakes
```

## Probes against a running server

`uvicorn app.main:app --host 0.0.0.0 --port 8000`

| Probe | Result |
| --- | --- |
| `GET /api/v1/health` | `status ok`, version `4.0.0` |
| `GET /api/v1/ready` | `ready` · database ✓ · content ✓ · 15 missions · 26 items |
| `GET /` (Mission Control) | `200 text/html`, 46,919 bytes, no external assets |
| `WS /api/v1/games/{id}/stream?after_seq=0` | `stream.ready` → `events` frames carrying `game.created`, `mission.started`, `action.succeeded` |
| `POST /api/v1/games` | `game_id` + `session_token` issued |
| `training_prep`, 7 actions | `200` each, report grade **S**, 0 mistakes |
| `analog_water_leak` → `repair_pipe` **before** `close_valve` | **409 `precondition_failed`** with `details.unmet`, water stays at 88 L |
| `POST /api/v1/earth/compare` (nile_delta · lst · 2001-2005 → 2021-2025 · `live=true`) | **`provenance: live`** — NASA POWER `T2M`, mean delta **+0.8858 °C (+4.06%)**, verdict `degrading`, 5 hotspots |
| same request repeated | `provenance: cache` (900 s TTL) |
| `GET /api/v1/nasa/apod` | **`provenance: live`** — NASA APOD |

The live probes need outbound network access. When it is missing (or `DEMO_KEY`
is throttled) the same endpoints answer with `provenance: bundled` plus a
`live_fallback_reason`, which is the designed degradation — the campaign above
runs fully offline.
