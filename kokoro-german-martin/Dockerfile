# https://developers.home-assistant.io/docs/apps/configuration#app-dockerfile
#
# Note: the tutorial's alpine base (ghcr.io/home-assistant/base:latest) cannot
# be used here: onnxruntime only ships glibc (manylinux) wheels, and
# wyoming-openai requires Python >= 3.12. The official Debian base image
# ships Python 3.13, s6-overlay v3 and bashio, and is multi-arch, so a single
# FROM line covers both amd64 and aarch64.
FROM ghcr.io/home-assistant/base-debian:trixie

ARG BUILD_VERSION

# --- System packages -------------------------------------------------------
RUN \
    apt-get update \
    && apt-get install -y --no-install-recommends \
        python3 \
        python3-pip \
        curl \
        git \
        espeak-ng \
        libsndfile1 \
    && rm -f /usr/lib/python3*/EXTERNALLY-MANAGED \
    && rm -rf /var/lib/apt/lists/*

# --- Python packages -------------------------------------------------------
# kokoro-onnx 0.5.0    -> ONNX Kokoro runtime (needs espeakng-loader/phonemizer)
# wyoming-openai 0.7.0 -> Wyoming bridge, requires Python >= 3.12
# watchfiles <1.3      -> 1.3.0 ships no cp313 manylinux wheel (uvicorn extra dep)
RUN \
    pip3 install --no-cache-dir \
        "uv" \
    && uv pip install --system --no-cache \
        "fastapi" \
        "uvicorn[standard]" \
        "watchfiles<1.3" \
        "numpy" \
        "kokoro-onnx==0.5.0" \
        "wyoming-openai==0.7.0" \
    && rm -rf /root/.cache

# --- Model files (baked into the image) -------------------------------------
# ~326 MB from Hugging Face. Kept in the image on purpose so first start is
# instant; the app version is bumped when the model changes.
RUN \
    mkdir -p /app \
    && curl -L --fail -o /app/kokoro-martin.onnx \
        "https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin/resolve/main/kokoro-martin.onnx" \
    && curl -L --fail -o /app/voices-martin.npz \
        "https://huggingface.co/Godelaune/Kokoro-82M-ONNX-German-Martin/resolve/main/voices-martin.npz"

# --- TTS service code (fetched from upstream at build time) -----------------
# This repository deliberately contains NO copies of the upstream service
# code. It is fetched here from the upstream GitHub repository, pinned to an
# exact commit. The checksums make the build deterministic: if the files
# ever change, the build fails instead of silently drifting.
#
# Bumping the upstream version (e.g. v1.3): update KOKORO_UPSTREAM_REF to the
# new commit, regenerate the four checksums (sha256sum of the files at that
# commit), and bump the app version in config.yaml.
ARG KOKORO_UPSTREAM_REF=d96573c2bf296a5576014ee57b8baa059d0746d9
RUN \
    git clone --quiet https://github.com/Godelaune/Kokoro-82M-ONNX-German-Martin \
            /tmp/kokoro-upstream \
    && cd /tmp/kokoro-upstream \
    && git checkout --quiet "${KOKORO_UPSTREAM_REF}" \
    && printf '%s\n' \
         "3145392160d8845d30dd8b25946d787c2ce5e50904c66b0a34ec6f66c7844b4f  onnx-docker/main.py" \
         "d13f742a0386db7dcb12434d3a74d1efcb746dd972fb9ce843b7378b48022182  onnx-docker/tts_normalizer.py" \
         "ea5b942290c2dc612bef0177c1b3d1755d3b6b5452e728d15537b55a24ddaacd  german_text_rules.py" \
         "12728aa2ae5cf6c87f441c7ea4cf9591221befa4b70271ff22677f9ca2df1f7d  wyoming_openai_german_separator/apply_german_separator_patch.py" \
        | sha256sum -c - \
    && cp onnx-docker/main.py /app/main.py \
    && cp onnx-docker/tts_normalizer.py /app/tts_normalizer.py \
    && cp german_text_rules.py /app/german_text_rules.py \
    && cp wyoming_openai_german_separator/apply_german_separator_patch.py \
            /tmp/apply_german_separator_patch.py \
    && cp german_text_rules.py \
            "$(python3 -c 'import site; print(site.getsitepackages()[0])')/german_text_rules.py" \
    && cd / \
    && rm -rf /tmp/kokoro-upstream

# main.py (upstream) has VOICES_PATH hardcoded to /app/voices-martin.npz
# (the model files are copied into /app above).

# The Wyoming bridge (importing wyoming_openai.handler) also imports
# german_text_rules (patched in below), so it is placed in site-packages as
# well (done in the step above).

# --- German sentence-separator patch for the Wyoming bridge -----------------
# Protects dotted German abbreviations/dates/ordinals during pySBD
# segmentation. Applied at build time; the script is idempotent.
RUN python3 /tmp/apply_german_separator_patch.py \
    && rm /tmp/apply_german_separator_patch.py

# --- s6 service definitions --------------------------------------------------
COPY rootfs /

ENV ORT_DISABLE_TELEMETRY=1
WORKDIR /app

LABEL \
    io.hass.version="${BUILD_VERSION}" \
    io.hass.type="app" \
    io.hass.arch="aarch64|amd64" \
    org.opencontainers.image.title="Home Assistant App: Kokoro German (Martin)" \
    org.opencontainers.image.description="German Kokoro-82M ONNX TTS with Wyoming bridge for Home Assistant Assist." \
    org.opencontainers.image.source="https://github.com/Godelaune/Kokoro-82M-ONNX-German-Martin" \
    org.opencontainers.image.licenses="Apache-2.0"
