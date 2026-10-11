# Kokoro German (Martin) — Home Assistant App

German TTS for Home Assistant Assist using the [Kokoro-82M ONNX Martin voice](https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin).

> ## ⚠️ What this repository is — and is not
>
> This is **packaging only**. It wraps the existing, independently developed
> [Kokoro-82M ONNX German "Martin"](https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin)
> project (by **Godelaune**) into a Home Assistant app so it can be installed
> on HAOS in one click.
>
> The model, the voice, the ONNX conversion, the synthesis service, the German
> text normalization and the Wyoming bridge are **all upstream work — not the
> work of the maintainer of this repo**. See [NOTICE](../NOTICE.txt) for the full
> attribution and the original authors to contact.
>
> _This repository was developed with an AI coding agent (see the
> [AI Disclosure](../README.md#ai-disclosure) section in the repository README)._

→ See [DOCS.md](./DOCS.md) for installation, options and troubleshooting.

```
kokoro-german-martin/
├── config.yaml      # app manifest (options, ports, discovery)
├── Dockerfile       # image: TTS service + Wyoming bridge + model
├── DOCS.md / README.md / CHANGELOG.md
├── icon.png / logo.png
└── rootfs/etc/s6-overlay/s6-rc.d/
    ├── kokoro/     # longrun  — uvicorn on 127.0.0.1:8881
    ├── wyoming/    # longrun  — wyoming_openai on 0.0.0.0:10203 (depends on kokoro)
    └── discovery/  # oneshot  — sends Wyoming discovery to Home Assistant
```

This repository deliberately contains **no copies of the upstream service
code**. The Dockerfile fetches it from
[Godelaune/Kokoro-82M-ONNX-German-Martin](https://github.com/Godelaune/Kokoro-82M-ONNX-German-Martin)
at build time, pinned to an exact commit and verified by SHA-256 checksum.
The model files come from Hugging Face; the ONNX runtime and Wyoming bridge
come from PyPI. Everything "cool" stays upstream.

No `apparmor.txt`: like the official Piper app, this relies on the
Supervisor's default AppArmor profile.

## Upstream

- [Godelaune/Kokoro-82M-ONNX-German-Martin](https://github.com/Godelaune/Kokoro-82M-ONNX-German-Martin) (Apache-2.0)
- [roryeckel/wyoming_openai](https://github.com/roryeckel/wyoming_openai)
- [thewh1teagle/kokoro-onnx](https://github.com/thewh1teagle/kokoro-onnx)
