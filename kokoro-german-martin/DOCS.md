# Kokoro German (Martin)

German text-to-speech for **Home Assistant Assist** (Wyoming protocol) using
the [Kokoro-82M ONNX "Martin" voice](https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin)
with local German text normalization.

> **This is a packaging app, not the original project.** It bundles the
> upstream [Godelaune/Kokoro-82M-ONNX-German-Martin](https://github.com/Godelaune/Kokoro-82M-ONNX-German-Martin)
> service (model, voice, normalization, Wyoming bridge — all upstream work by
> **Godelaune** and the model authors) into a single Home Assistant app for
> HAOS. See [NOTICE](../NOTICE.txt) for full attribution.

This is a Home Assistant **app** (formerly add-on) packaged from the
[Godelaune/Kokoro-82M-ONNX-German-Martin](https://github.com/Godelaune/Kokoro-82M-ONNX-German-Martin)
service setup: the Kokoro ONNX FastAPI server and the Wyoming OpenAI bridge
run together in one container, managed by s6-overlay.

## What it does

- Exposes the German "Martin" voice to Home Assistant Assist (and any
  Wyoming client).
- Normalizes German text before synthesis: dates, times, ordinals, Euro
  amounts, decimals with units (`2,5 kWh` → "zwei komma fünf Kilowattstunden"),
  and dotted abbreviations (`Min.`, `Stck.`, `ltr.`, `ggf.`, …).
- Protects German dotted abbreviations during streaming sentence
  segmentation, so `Prof.` or `Min.` do not split a sentence early.
- Process-isolated synthesis workers for parallel sentence generation.

## Installation

### From this repository (recommended)

1. In Home Assistant: **Settings → Apps → App store → ⋮ → Repositories →
   Add**
2. Paste: `https://github.com/HtoTheB/kokoro-german-martin`
3. Install and start **Kokoro German (Martin)**. The first install builds
   the image on your device (base image + packages + ~326 MB model), which
   takes a few minutes.
4. Select "Kokoro (Streaming) / martin" as TTS voice in **Settings → Assist**.

The Wyoming endpoint is registered automatically via discovery — no manual
Wyoming integration is needed.

### As a local app (manual / offline)

1. Copy this folder to `/local_apps/kokoro-german-martin` on your HAOS
   system (via [Samba](https://my.home-assistant.io/redirect/supervisor_app/?app=core_samba)
   or [SSH](https://my.home-assistant.io/redirect/supervisor_app/?app=core_ssh)).
2. In Home Assistant open **Settings → Apps → App store** and click
   **Check for updates**.
3. Install and start the app, then pick the voice as above.

## Options

| Option | Default | Description |
|---|---|---|
| `workers` | `1` | Parallel synthesis workers. Keep at 1 unless the system has ~2 GB+ free RAM; the multi-worker path is not crash-resistant. |
| `intra_op_threads` | `2` | ONNX intra-op threads per worker. |
| `inter_op_threads` | `1` | ONNX inter-op threads per worker. |
| `speed` | `1.125` | Speaking speed multiplier. |
| `pause_duration` | `0.25` | Silence between sentences (seconds). |
| `execution_mode` | `sequential` | ONNX execution mode. |
| `graph_opt` | `all` | ONNX graph optimization level. |
| `allow_spinning` | `false` | ONNX thread spinning. |
| `concurrent_requests` | `3` | Concurrent streaming TTS requests. |

Recommended profile for a small Intel NUC / 4-core system: `workers: 1`,
`intra_op_threads: 2` (the defaults). Raising `workers` above 1 loads a full
model instance per worker (~0.5–1 GB each) and the upstream process pool is
not crash-resistant (a dying worker permanently breaks TTS until restart), so
only do that on systems with plenty of free RAM.

## Model update / image size

The ~326 MB ONNX model is baked into the image, so first start is instant.
The model stays in the image; when a new model version is published, the app
version is bumped and the image is rebuilt (redownloading the model).

The upstream service code is **fetched from the upstream GitHub repository at
build time**, pinned to an exact commit (`KOKORO_UPSTREAM_REF` in the
Dockerfile) and verified by SHA-256 checksum. This repo therefore contains no
copies of it. To track a new upstream release: update the pin to the new
commit, regenerate the checksums, and bump the app version in `config.yaml`.

## Troubleshooting

- **App does not show up in the store:** `config.yaml` is invalid or has
  CRLF line endings. Check **Settings → System → Logs → Supervisor** for the
  validation error.
- **Slow first sentence:** the workers do a background warm-up after startup;
  give the app a minute after a start/restart.
- **Check the service is reachable** (inside the container or on the host
  network): `curl http://localhost:8881/v1/audio/voices` →
  `{"voices": ["martin"]}`.
- Logs: **Apps → Kokoro German (Martin) → Logs**.

## Credits

- Model: [kikiri-tts/kikiri-german-martin](https://huggingface.co/kikiri-tts/kikiri-german-martin),
  base [dida-80b/kokoro-german-hui-multispeaker-base](https://huggingface.co/dida-80b/kokoro-german-hui-multispeaker-base),
  architecture [hexgrad/Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M).
- ONNX conversion, service and German normalization:
  [Godelaune/Kokoro-82M-ONNX-German-Martin](https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin).
- Wyoming bridge: [roryeckel/wyoming_openai](https://github.com/roryeckel/wyoming_openai).
- All code is used under its Apache-2.0 license.
