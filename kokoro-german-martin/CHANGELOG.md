# Changelog

This app tracks the versioning of the upstream
[Godelaune/Kokoro-82M-ONNX-German-Martin](https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin)
publication (v1.0, v1.1, v1.2). The app version is bumped whenever the
upstream version is — and on the first packaging fix releases below.

## 1.2

- Packaging fix: default `workers` is 1. With 2 workers one of the parallel
  model loads died on a 2 GB RAM test system (BrokenProcessPool), which
  permanently breaks the process pool for all later requests. 1 worker
  (sequential sentence synthesis) is the safe default; raise it only on
  systems with enough free RAM (~2 GB+).
- Packaging fix: container startup failure on current HAOS base images —
  the `discovery` oneshot service required its s6-rc v3 `up` file. The
  container now boots and all three services start.

## 1.1

- Upstream v1.1: German text normalization (decimal numbers with units,
  singular/plural units, abbreviations, Euro amounts, ordinals).

## 1.0

- Initial app version: Kokoro ONNX German "Martin" voice as a Home Assistant
  app (Wyoming TTS for Assist), bundling upstream v1.0.
- Combines the Kokoro ONNX FastAPI service and the Wyoming OpenAI bridge
  with the German sentence-separator patch in a single container.
- Model files are baked into the image.
