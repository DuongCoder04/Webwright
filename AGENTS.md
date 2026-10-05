# Working in CUAWright

Use Python 3.12 for desktop work. Install with:

```bash
pip install -e ".[desktop,test,build]"
playwright install chromium
```

Keep the browser and desktop runtimes separate:

- `src/cuawright/webwright/` owns browser messages, tools, compaction, and completion.
- `src/cuawright/desktop/` owns the OSWorld VM, guest bridge, explicit submit, and evaluation.
- `src/cuawright/core/` is only for small contracts that are genuinely identical.

Do not introduce a universal agent/model/environment abstraction or duplicate the
browser runtime under another package. Preserve `webwright` compatibility aliases.
Public code and artifacts must contain only standard Responses-compatible API
configuration; never persist credentials or endpoint identities.

Before changing desktop prompts, read `src/cuawright/desktop/config/prompts.py` in
full. Plain final text never completes a desktop task; the Actor must execute the
standalone submit control. Official reproduction must retain the pinned release,
task setup, evaluator, provenance, and score.

Validate focused changes first, then run:

```bash
pytest -q tests/unit release/osworld/tests
python -m build
python tests/packaging/check_wheels.py
```

Keep net changes within 900 production Python lines and 600 test lines. No newly
added production file should exceed 300 lines.
