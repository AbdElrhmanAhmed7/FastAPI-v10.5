# scratch – payload capture harness

`capture.py` plays a short campaign in-process and dumps the **real** response
body of every endpoint whose schema still exposes a free-form object (telemetry,
legacy dictionaries, simulation scenes, Earth grids…). It was used to decide
which of those payloads deserved a proper pydantic model.

```bash
cd NOVA/backend
.venv/bin/python ../../reference/scratch/capture.py     # writes payloads.json here
```

The generated `payloads.json` / `payloads_baseline.json` dumps (~290 KB each) are
**not committed** – they are reproducible artefacts, and the diff between two
runs is only interesting while you are actively typing a schema. Regenerate them
when you need a before/after comparison; nothing in `app/`, `scripts/` or `tests/`
reads them.

The authoritative, always-fresh contract is
[`NOVA/backend/docs/openapi.json`](../../NOVA/backend/docs/openapi.json), and
`python scripts/audit_routes.py --check` is the gate that reports how many
operations still expose free-form fields.
