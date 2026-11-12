# NOVA – Beyond the Mission

**Explore. Experience. Execute.**

لعبة ويب ثلاثية الأبعاد عن رائد فضاء بيستكشف المريخ ويبني قاعدة أبحاث قابلة للاستمرار.
مش Dashboard ومش لعبة أسئلة: اللاعب **بيتحرّك بنفسه**، بيوصل للجهاز، بيتفاعل معاه،
والسيرفر هو اللي بيقرر هل الإجراء مسموح ولا لأ وبيغيّر الموارد فعليًا.

> **حالة المشروع دلوقتي:** الـ **Backend خلص بالكامل** ومتحقّق منه (15 مهمة، 67 عملية HTTP،
> 437 اختبار، تغطية 93%، والحملة كاملة بتتلعب من الـ API العام).
> الفرونت-إند (React Three Fiber) هو المرحلة الجاية، وبيبدأ من عقد الـ API الجاهز.
> التفاصيل الكاملة في [`PROJECT_STATUS.md`](PROJECT_STATUS.md).

| الطبقة | الحالة | المرجع |
| --- | --- | --- |
| Gameplay backend (FastAPI) | ✅ **مكتمل** – v4.0.0 | [`NOVA/backend/README.md`](NOVA/backend/README.md) |
| محتوى المهمات (عربي/إنجليزي) | ✅ 8 مراحل · 15 مهمة · 95 إجراء | [`NOVA/backend/content/`](NOVA/backend/content) |
| المحاكيات الفيزيائية | ✅ 5 محاكيات على السيرفر | `app/simulations/` |
| حفظ التقدّم + التقارير | ✅ SQLAlchemy 2 async + Alembic | `app/db/`, `migrations/` |
| بيانات NASA | ✅ POWER حيّ + fallback مكتوب المصدر | `app/integrations/`, `app/services/earth_service.py` |
| عقد الـ API للفرونت-إند | ✅ OpenAPI 3.1 مُصدَّر ومتحقّق منه في CI | [`docs/openapi.json`](NOVA/backend/docs/openapi.json) |
| كونسول تشغيل بدون أي مكتبات | ✅ Mission Control على `/` | `app/static/index.html` |
| 3D client (R3F + Rapier) | ⏳ المرحلة الجاية – مش متعمل | خطة التسليم في `PROJECT_STATUS.md` |

---

## ابدأ في 3 أوامر

```bash
cd NOVA/backend
python3 -m venv .venv && .venv/bin/pip install -r requirements-dev.txt
.venv/bin/uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

افتح `http://localhost:8000/` → **Mission Control**: كونسول بيلعب الـ API فعليًا (HUD،
مهمات، محاكيات، بث أحداث، تشخيص الخدمة) من غير أي build step.
`/docs` فيها Swagger UI بزر Authorize.

تلعب الحملة كلها من غير متصفح ومن غير سيرفر:

```bash
.venv/bin/python scripts/playthrough.py --in-process --difficulty standard --seed 7
```

تتأكد إن كل حاجة سليمة (نفس اللي CI بيعمله):

```bash
make check     # ruff · mypy · validate_content · audit_routes · openapi · pytest · playthrough
```

---

## What NOVA is

NOVA is a **third-person 3D web game** (with a `V` toggle to first person) about an
astronaut preparing for, travelling to and surviving a Mars research mission.
The design rule for every mission is **move → interact → consequence**:

1. the player walks to the device,
2. reaches a specific world object,
3. interacts and picks an action,
4. the character plays an animation / the environment reacts,
5. a real result lands: mission state and resources change.

The water leak is the canonical example. There is no quiz about how to fix a pipe:
the player scans the manifold, closes the isolation valve, then repairs the joint.
Repairing **before** closing the valve fails, wastes a repair kit and costs water —
the server enforces the order, not the client.

The split of responsibility is deliberate:

* **Client (planned):** rendering, movement, camera, animation, input, HUD.
* **Backend (done):** mission rules and preconditions, resources, inventory and tool
  wear, oxygen and EVA, physics simulations, scoring, continuity, persistence,
  events, and every number the HUD displays.

## Repository layout

```
repo/
├── README.md                 ← you are here (project overview)
├── PROJECT_STATUS.md         ← what is finished, verified, and what comes next
├── .github/workflows/backend.yml   CI: lint · types · content · audit · contract · tests · campaign
└── NOVA/
    └── backend/              the authoritative gameplay service (FastAPI)
        ├── app/
        │   ├── api/v1/       meta, content, games, missions, loadout,
        │   │                 simulations, earth, science, nasa, stream
        │   ├── api/legacy.py Prototype-01 URLs (deprecated, same engine)
        │   ├── core/         logging, errors, middleware, security, rate limit,
        │   │                 cache, locks, openapi, route introspection
        │   ├── db/           models, repositories, async session
        │   ├── domain/       enums, GameState document, catalogues
        │   ├── missions/     definitions, engine, registry, validation
        │   ├── simulations/  lunar_lander, microgravity_fire, crew_vitals,
        │   │                 rover_traverse, earth_trends
        │   ├── systems/      resources, inventory, oxygen, scoring,
        │   │                 continuity, achievements, event bus
        │   ├── services/     game / simulation / earth / science
        │   ├── schemas/      pydantic request & response models (136 schemas)
        │   └── static/       Mission Control dev console
        ├── content/          15 mission blueprints, items, resources,
        │                     achievements, science, NASA fallbacks
        ├── migrations/       Alembic environment + revisions
        ├── scripts/          playthrough, autopilot, validate_content,
        │                     audit_routes, export_openapi
        ├── tests/            437 tests, no network required
        └── docs/             openapi.json, routes.md, routes.json
```

## The campaign — 8 stages, 15 missions

Every mission is authored in `content/missions/*.json` with bilingual text
(`en` / `ar`), world objects, animations, preconditions, effects and scoring.
`requires_missions` builds the dependency graph; the engine refuses to start a
mission whose prerequisites are unmet.

| # | Stage | Mission | Environment | Objectives |
| --: | --- | --- | --- | --: |
| 1 | Training Facility | Mission Preparation · تجهيز المهمة | `training_facility` | 7 |
| 2 | Earth Analog | Habitat Systems Survey · فحص أنظمة القاعدة | `earth_analog` | 4 |
| 2 | Earth Analog | Water Leak Repair · إصلاح تسريب المياه | `earth_analog` | 4 |
| 2 | Earth Analog | Connect the Power Module · توصيل وحدة الطاقة | `earth_analog` | 6 |
| 2 | Earth Analog | Start the Water Recycler · تشغيل إعادة تدوير المياه | `earth_analog` | 6 |
| 2 | Earth Analog | Communications Blackout · انقطاع الاتصالات | `earth_analog` | 5 |
| 2 | Earth Analog | Faulty Science Device · عطل في جهاز علمي | `earth_analog` | 6 |
| 2 | Earth Analog | Emergency Drill · تدريب طوارئ | `earth_analog` | 8 |
| 3 | Mars Arrival | Mars Arrival & Exploration · الوصول للمريخ | `mars_surface` | 8 |
| 3 | Mars Arrival | Long-Range Rover Traverse · رحلة العربة الطويلة | `mars_surface` | 5 |
| 4 | Crew Health | Health Monitoring · متابعة صحة الطاقم | `medical_bay` | 8 |
| 5 | Freefall Fire | Flame in Freefall · حريق في انعدام الجاذبية | `orbital_station` | 7 |
| 6 | Martian Guide | Geological Sampling · أخذ عينات جيولوجية | `mars_lab` | 7 |
| 7 | Lunar Landing | Lunar Landing Navigator · محاكي الهبوط على القمر | `simulation_center` | 6 |
| 8 | Earth Trends | Observation Station · محطة رصد الأرض | `earth_observation` | 6 |

93 objectives + 2 optional side actions = **95 validated actions**, ~117 minutes of
authored play. The Moon landing lives in the **Simulation Center**: the game never
pretends the Moon and Mars are the same place.

## Controls → endpoints

The client's key bindings are served by `GET /api/v1/content/controls`, so the
frontend and the backend cannot drift.

| Key | Action | Endpoint |
| --- | --- | --- |
| `W A S D` / `Shift` | move / sprint | `POST /games/{id}/tick` (`activity=walk|run`) |
| `Mouse` / `Space` / `V` | look / jump / camera toggle | *client only* |
| `E` | interact | `POST /games/{id}/missions/{mission_id}/actions` |
| `F` | use tool | `POST /games/{id}/inventory/use` |
| `Tab` | inventory | `GET /games/{id}/inventory` |

`GET …/interactions` returns the nearby world objects with `available`, `unmet` and
`hint` for each interaction, so the HUD can grey out an impossible action instead
of letting the player fail blindly — while the server still enforces the rule.

## Mission Continuity System

`GET /api/v1/games/{id}/continuity` explains every carry-over modifier: preparation
quality, tool maintenance, resource discipline and the training grade feed a
readiness score and the starting conditions of later stages. Each modifier carries
a label, a source mission and a delta, so a report can show *why* the rover started
at 80% battery. The modifiers are **gameplay balance, not physics** — and the
payload says so.

## Scientific honesty

Every data payload labels its provenance: `live`, `cache`, `bundled` or `simulated`.

* **Earth Trends** can query **NASA POWER** monthly climate data (no API key) for the
  temperature and precipitation layers; NDVI, aerosol and night-lights layers use a
  clearly-labelled educational model.
* **APOD / EPIC / DONKI / NEO / Mars photos** come from `api.nasa.gov` with a bundled
  offline snapshot as fallback (`DEMO_KEY` is heavily rate limited).
* **Crew vitals, the medical procedure and microgravity fire** are educational
  simulations. The disclaimer travels inside the payload, not only in the docs.

## Quality gates

```bash
make check
```

runs ruff (lint + format), mypy, static mission-content validation, the route audit
(guards, typed responses, security blocks), the OpenAPI freshness check, the offline
pytest suite with coverage, and a full 15-mission campaign played through public HTTP.
CI (`.github/workflows/backend.yml`) runs the same gates in three jobs and uploads the
coverage and campaign reports.

## Frontend integration (next phase)

```bash
npx openapi-typescript NOVA/backend/docs/openapi.json -o src/api/nova.d.ts
```

1. `POST /api/v1/games` → keep `game_id` (and `session_token` when auth is on).
2. `GET /api/v1/games/{id}` → bootstrap the HUD from one snapshot.
3. `POST /api/v1/games/{id}/tick` about once per second (**not every frame**).
4. `E` → `POST …/missions/{mission_id}/actions` with an `Idempotency-Key` header.
5. `WS /api/v1/games/{id}/stream?after_seq=N` for live events — a reconnect resumes
   from its cursor and never loses an event.

Planned client stack: React + Vite + Three.js / React Three Fiber + Rapier + Tailwind,
with Chart.js for the science and Earth panels. Assets in Blender.

## Docs map

| Document | What |
| --- | --- |
| [`PROJECT_STATUS.md`](PROJECT_STATUS.md) | حالة كل طبقة، نتائج الفحص، والخطوة الجاية |
| [`NOVA/backend/README.md`](NOVA/backend/README.md) | backend manual: contract, config, deployment, security |
| [`NOVA/backend/docs/routes.md`](NOVA/backend/docs/routes.md) | generated route inventory (67 operations) |
| [`NOVA/backend/docs/openapi.json`](NOVA/backend/docs/openapi.json) | the published contract (OpenAPI 3.1, 64 paths, 136 schemas) |
| `NOVA/backend/.env.example` | every `NOVA_*` setting with its default |

## License

MIT.
