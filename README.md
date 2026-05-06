# Apollo

Apollo is a self-hosted desktop stream host for [Artemis(Moonlight Noir)](https://github.com/ClassicOldSong/moonlight-android). Offering low latency, native client resolution, cloud gaming server capabilities with support for AMD, Intel, and Nvidia GPUs for hardware encoding. Software encoding is also available. A web UI is provided to allow configuration and client pairing from your favorite web browser. Pair from the local server or any mobile device.

## Usage

Refer to LizardByte's documentation hosted on [Read the Docs](https://docs.lizardbyte.dev/projects/sunshine) for now.

Currently Virtual Display support is Windows only, Linux support is planned and will be implemented in the future.

## PSP Compatibility (H.264 CAVLC)

PSP playback is significantly more reliable when the host emits H.264 with CAVLC-compatible settings. In vanilla Sunshine/Apollo paths, AMF H.264 can still end up producing CABAC-style output in scenarios where users expect CAVLC, and the bitstream entropy mode is not explicitly verified at runtime.

### Why this matters

- PSP hardware decoding is strict compared to modern clients.
- CABAC/high-profile oriented output can fail or behave inconsistently on PSP-focused playback pipelines.
- A config value alone is not enough if the final encoded bitstream does not match the requested entropy mode.

### What changed in this fork

- AMF H.264 `coder` handling is now wired with profile selection:
  - `cavlc` -> constrained baseline profile
  - `cabac` -> high profile
  - `auto` -> encoder default behavior
- Runtime bitstream verification was added for H.264 entropy mode (CABAC vs CAVLC), with mismatch logging.
- AMF active probe/validation is skipped for known hang scenarios and capability-only checks are used there.
- AMF option parsing for some toggles was normalized to integer values expected by the encoder options path.

### Why it is needed for PSP

The goal is deterministic PSP-safe output: when CAVLC is requested, the encoder path now aligns profile + coder configuration and verifies what is actually produced. This reduces silent fallbacks to incompatible entropy coding and makes troubleshooting visible in logs.

If you are targeting PSP, prefer H.264 with `amd_coder = cavlc` and avoid relying on automatic entropy selection.

## License

GPLv3
