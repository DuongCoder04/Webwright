# Reproduce OSWorld-V2

CUAWright supports two Ubuntu VM workflows:

1. `desktop run`: an arbitrary task in the pinned OSWorld VM. Submission saves the
   verified outcome; no official evaluator runs and no score is produced.
2. `desktop reproduce osworld`: an official task with the pinned task class, assets,
   setup, evaluator, and benchmark score.

There is no host-desktop mode.

## Install and prepare

Use Python 3.12+, Docker, and preferably KVM:

```bash
pip install -e ".[desktop]"
```

Accept access to both gated datasets, then authenticate:

- <https://huggingface.co/datasets/xlangai/osworld_v2_tasks>
- <https://huggingface.co/datasets/xlangai/osworld_v2_assets_gated>

```bash
uvx --from huggingface_hub hf auth login
```

The default setup clones the pinned OSWorld source, installs its dependencies,
downloads the exact gated task/assets snapshots, verifies the task hash manifest,
and downloads the official 0624 VM artifact reused by the 0808 release:

```bash
cuawright setup osworld --root ~/.cache/cuawright/osworld-v2-2026.08.08
cuawright verify osworld \
  --setup ~/.cache/cuawright/osworld-v2-2026.08.08/setup.json
```

The VM archive is about 14.2 GB. To register already provisioned resources without
copying them, pass all existing paths:

```bash
cuawright setup osworld \
  --root /path/to/cuawright-osworld \
  --source /path/to/clean/OSWorld-V2 \
  --tasks /path/to/task-classes \
  --assets /path/to/assets \
  --task-manifest /path/to/task_hashes.json \
  --vm /path/to/Ubuntu.qcow2 \
  --skip-install
```

`setup.json` records the selected VM hash. A repaired or expanded local VM is valid
when registered explicitly, but it is not claimed to be byte-identical to the
compressed release artifact.

Put one Responses API key in a user-owned mode-`0600` file. Keep it outside the
source, tasks, assets, and result directories.

## Smoke and custom task

Run a paid end-to-end VM smoke:

```bash
cuawright smoke osworld \
  --setup /path/to/setup.json \
  --credentials /path/to/api-key \
  --results /new/path/results/smoke \
  --model YOUR_MODEL \
  --responses-url https://api.example/v1/responses
```

Run any other desktop task:

```bash
cuawright desktop run \
  --setup /path/to/setup.json \
  --credentials /path/to/api-key \
  --results /new/path/results/custom \
  --model YOUR_MODEL \
  --responses-url https://api.example/v1/responses \
  --instruction "Create, save, verify, and submit the requested artifact."
```

## Official scored reproduction

```bash
cuawright desktop reproduce osworld \
  --setup /path/to/setup.json \
  --credentials /path/to/api-key \
  --results /new/path/results/task-001 \
  --task-id 001 \
  --model YOUR_MODEL \
  --responses-url https://api.example/v1/responses \
  --reasoning high
```

The result directory must not exist before launch. `manifest.json` pins harness,
source, task, assets, VM, prompts, tools, and settings. `trace.jsonl` records model
requests, responses, commands, retries, and cleanup. `result.json` records explicit
failure stages or the completed outcome; official reproduction additionally records
the evaluator score.

## Browser sample

Browser execution remains an independent Webwright workflow:

```bash
cuawright web main \
  -c base.yaml -c model_openai.yaml -c local_browser.yaml \
  -c environment.browser_mode=local_launch \
  -c environment.headless=true \
  -c model.responses_url=https://api.example/v1/responses \
  -c model.model_name=YOUR_MODEL \
  -t "Open the page and report its title." \
  --start-url https://example.com \
  --task-id sample
```

The existing `cuawright-web` and `webwright` commands remain compatible.
