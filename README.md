# kokoro-german-martin

A **Home Assistant app repository** that packages the upstream [Kokoro-82M ONNX German "Martin" TTS](https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin) project into a single installable app for Home Assistant OS (HAOS).

## What this is

**Packaging only.** The model, voice, ONNX conversion, synthesis service, German text normalization and Wyoming bridge are all upstream work by their respective authors — not the work of this repository's maintainer. This repo only bundles them so they can be installed on HAOS in one click.

- App documentation: [kokoro-german-martin/DOCS.md](kokoro-german-martin/DOCS.md)
- Attribution & upstream licenses: [NOTICE](NOTICE.txt)
- License: [Apache-2.0](LICENSE.txt) (same as the upstream model)

## Install

1. In Home Assistant: **Settings → Apps → App store → ⋮ → Repositories → Add**
2. Paste: `https://github.com/HtoTheB/kokoro-german-martin`
3. Install and start **Kokoro German (Martin)**
4. Select the `kokoro` / `martin` voice in **Settings → Assist**

## AI Disclosure

AI was used in the creation of this repository.

This project (the Home Assistant app **packaging** — Dockerfile, s6 service definitions, app manifest, and documentation) was developed with the help of an AI-powered coding agent.
- **Tool:** OpenCode
- **Model:** Qwen 3.8 27B

#### What the AI did

- Researched the Home Assistant app (formerly add-on) documentation and the current Supervisor source code to determine the correct packaging layout.
- Drafted the packaging files: `Dockerfile`, `config.yaml`, the s6-overlay service scripts under `rootfs/`, translations, `repository.yaml`, and the documentation.
- Analyzed the upstream source code and helped fix two startup bugs found during on-device testing (a missing s6-rc v3 `up` file for the `discovery` oneshot service, and an unsafe default for the synthesis worker count).

#### What the AI didn't

- The TTS model, the German "Martin" voice, the ONNX conversion, the synthesis service, the German text normalization and the Wyoming bridge are **upstream work by their respective authors** (see [NOTICE](NOTICE.txt)). AI was not involved in creating them, and they are not part of this repository's contributions.

#### Human review

The code was tested and verified working on my Home Assistant OS installation before publication. I can't guarantee that it works anywhere else. If you find any issues, feel free to open an issue or a PR.

This disclosure does not change the licensing: the packaging files remain under [Apache-2.0](LICENSE.txt), and all upstream components retain their own licenses as documented in [NOTICE](NOTICE.txt).
