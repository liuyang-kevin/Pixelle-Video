# Code Review Notes

## Workflows Usage

`workflows/` is split by execution backend:

- `workflows/runninghub/*.json`: RunningHub wrapper files, usually containing `source` and `workflow_id`.
- `workflows/selfhost/*.json`: local ComfyUI API workflow files.
- `data/workflows/` can override built-in workflow files through resource lookup helpers.

Core workflow discovery is implemented in:

- `pixelle_video/services/comfy_base_service.py`
- `pixelle_video/utils/os_util.py`

The normal service flow is:

1. Scan `workflows/{source}/*.json` and `data/workflows/{source}/*.json`.
2. Filter by filename prefix.
3. Build a workflow key such as `runninghub/image_flux.json`.
4. If the JSON contains `source=runninghub` and `workflow_id`, execute by workflow ID.
5. Otherwise execute by local workflow file path.

## Prefix-Based Modules

Known filename prefixes:

- `tts_`: TTS workflows, used by `TTSService`.
- `image_`: image generation workflows, used by `MediaService`.
- `video_`: video generation workflows, used by `MediaService`.
- `analyse_`: image analysis workflows.
- `analyse_video`: video analysis workflows.
- `af_`: action transfer workflows, handled separately in Streamlit UI.
- `i2v_`: image-to-video workflows, handled separately in Streamlit UI.
- `digital_`: digital human workflows, mostly hardcoded in Streamlit UI.

## Special Cases

Some feature modules do not use the shared `ComfyBaseService` discovery path.

- Action transfer: `web/pipelines/action_transfer.py`
  - Scans `workflows/runninghub` and `workflows/selfhost` directly.
  - Only accepts files named `af_*.json`.
  - Current repo only has `workflows/runninghub/af_scail.json`, so the UI only shows RunningHub.

- Image to video: `web/pipelines/i2v.py`
  - Scans directories directly.
  - Only accepts files named `i2v_*.json`.
  - Current repo only has `workflows/runninghub/i2v_LTX2.json`.

- Digital human: `web/pipelines/digital_human.py`
  - Uses hardcoded workflow paths.
  - RunningHub files exist.
  - Selfhost paths are referenced but corresponding files are currently absent.

## Potential Issues

- Workflow discovery is inconsistent between core services and Streamlit-only feature modules.
- Some comments/docs imply a standard `{source}/{service}.json` convention, but actual media files use model-specific names such as `image_flux.json` and `video_wan2.1_fusionx.json`.
- `video_understanding.json` starts with `video_`, so it may be listed as a media/video generation workflow even though the name suggests analysis.
- RunningHub video analysis expects `runninghub/analyse_video.json`, but that file is not present in the current `workflows/runninghub/` directory.
- Selfhost action transfer, I2V, and digital human are structurally supported in code paths, but corresponding workflow files are missing.

## Follow-Up Notes

- Consider centralizing special workflow discovery for `af_`, `i2v_`, and `digital_`.
- Consider adding metadata to workflow JSON files to avoid relying only on filename prefixes.
- Consider validating workflow availability at startup or UI render time with clearer error messages.
- Consider documenting required workflow parameter names per module, e.g. `prompt`, `image`, `video`, `second`, `audio`.

