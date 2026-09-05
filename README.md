# MiniMax H3 20s Continuation Workflow for ComfyUI

This is a two-pass MiniMax H3 Image-to-Video workflow for ComfyUI that generates two consecutive video segments and combines them into one approximately 20-second continuous video.

```text
Input Image
→ Pass 1 (~10s)
→ Extract Last Frame
→ Pass 2 (~10s)
→ Merge
→ Upscale
→ Final ~20s Video
```

## Included Workflows

* `workflows/MiniMax_H3_20s_Continuation.json` - API-format workflow.
* `workflows/MiniMax_H3_20s_Continuation_UI.json` - UI-format workflow with canvas layout and Markdown notes.

## Features

* Two sequential MiniMax H3 Image-to-Video passes.
* Automatic extraction of the final frame of Pass 1.
* Final frame used as the starting frame for Pass 2.
* Separate prompts for both passes.
* Automatic frame concatenation.
* Audio concatenation from both passes.
* Final image upscale before video creation.
* Approximately 20-second final video.

## Prompt Structure

The workflow uses two prompt fields, one for each MiniMax H3 pass. A practical way to write them is:

**Common:**
Information that should remain consistent across both passes, such as character identity, clothing, scene details, camera behavior, lighting and style.

**Start:**
Action and events for the first part of the video.

**End:**
Continuation, action and events for the second part of the video.

For a strong visual continuation, it is useful to keep important identity anchors visible near the end of Pass 1. For example, if a character is used, keeping the face visible in the final frame gives Pass 2 a stronger visual reference.

## How It Works

The included JSON keeps the two-pass continuation architecture:

```text
LoadImage
→ ImageScaleToTotalPixels
→ GetImageSize
→ MiniMaxH3ImageToVideo, Pass 1
→ SamplerCustomAdvanced
→ VAEDecode
```

Then the workflow extracts the final frame of Pass 1:

```text
VAEDecode, Pass 1
→ ComfyMathExpression, last frame index
→ ImageFromBatch
```

The extracted frame is used as the first frame for Pass 2:

```text
ImageFromBatch
→ MiniMaxH3ImageToVideo, Pass 2
→ SamplerCustomAdvanced
→ VAEDecode
```

Finally, both decoded frame batches are combined and rendered:

```text
VAEDecode, Pass 1 + VAEDecode, Pass 2
→ BatchImagesNode
→ UnloadAllModels
→ ImageUpscaleWithModel
→ CreateVideo
→ SaveVideo
```

Audio from both passes is decoded separately and concatenated:

```text
VAEDecodeAudio, Pass 1 + VAEDecodeAudio, Pass 2
→ AudioConcat
→ CreateVideo
```

## Installation

Install the required models and custom nodes in your own ComfyUI environment. This repository does not include models.

Required model files found in the workflow:

* `minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors`  
  https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/resolve/main/diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors
* `minimax_h3_video_vae_int8_convrot.safetensors`  
  https://huggingface.co/Kijai/MiniMax-H3-experimental/resolve/main/minimax_h3_video_vae_int8_convrot.safetensors
* `minimax_h3_audio_vae_fp32.safetensors`  
  https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors
* `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`  
  https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
* `2x_OpenProteus_Compact_i2_70K.pth`  
  https://huggingface.co/hfmaster/models-moved/resolve/main/upscalers/2x_OpenProteus_Compact_i2_70K.pth

The workflow also includes an unconnected `PrimitiveStringMultiline` node named `Model Links / Download Notes` with the same model list.

LoRA:

* No LoRA files are enabled in the public workflow.

Custom node classes used by this workflow:

* `MiniMaxH3ImageToVideo` - Repository link needs to be added.
* `H3SLAAttention` - Repository link needs to be added.
* `LTX_lora_loader` - Repository link needs to be added.
* `AudioConcat` - Repository link needs to be added.
* `BatchImagesNode` - Repository link needs to be added.
* `ImageScaleToTotalPixels` - Repository link needs to be added.
* `GetImageSize` - Repository link needs to be added.
* `ComfyMathExpression` - Repository link needs to be added.
* `ImageUpscaleWithModel` - Repository link needs to be added.
* `CreateVideo` - Repository link needs to be added.
* `SaveVideo` - Repository link needs to be added.
* `UnloadAllModels` - Repository link needs to be added.

## Usage

1. Install the required models and custom nodes.
2. Open or import `workflows/MiniMax_H3_20s_Continuation_UI.json` in ComfyUI for the visual canvas version, or use `workflows/MiniMax_H3_20s_Continuation.json` for API/script usage.
3. Load an input image in the `LoadImage` node.
4. Set the desired prompt for Pass 1 and Pass 2.
5. Queue the workflow.
6. The workflow generates Pass 1, extracts its final frame, generates Pass 2 and combines both sections into the final video.

## Demo Prompt

**Common:**

Cinematic realistic video, smooth natural motion, seamless continuation between both parts, consistent character identity, consistent red coat, elegant old hotel atmosphere, detailed brass elevator interior, soft atmospheric lighting, gentle camera movement, clean composition, realistic pacing. Keep the same main character, clothing, facial appearance, environment details and visual style throughout the sequence.

**Start:**

A young woman in a stylish red coat walks through the grand lobby of an elegant old hotel at night. Warm golden lights reflect on the polished floor, and the atmosphere feels quiet, luxurious, and slightly mysterious. The camera follows her smoothly from a medium distance as she approaches a vintage brass elevator with ornate doors. The elevator opens and she steps inside. The camera moves into the elevator with her. Inside, she turns to face the camera naturally. Her face remains clearly visible and recognizable while the warm elevator lights illuminate her features and red coat. She briefly looks toward the elevator doors and then back toward the camera as the elevator begins to move.

**End:**

Continue seamlessly from the same moment inside the same elevator. The young woman remains clearly visible in front of the camera, with the same recognizable face and unchanged red coat. The lighting inside the elevator gradually becomes brighter and more ethereal. She turns toward the elevator doors as they slowly open. Beyond the doors is no longer the hotel, but a breathtaking fantasy landscape high above the clouds: a vast sunlit valley, floating rock formations, long waterfalls, soft mist, and a distant shining city. She reacts with quiet surprise and a faint smile, then walks out of the elevator. The camera follows her forward and slightly upward, revealing the scale of the landscape as golden morning light fills the frame.

## Notes

Continuation is based on the last frame of Pass 1, so:

* Objects that fully disappear from the last frame may be less stable in Pass 2.
* If character identity matters, keep the face visible near the end of Pass 1.
* Very abrupt scene changes between passes may reduce continuity.

## Tested Configuration

Values below are taken from the included workflow JSON:

* MiniMax model: `minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors`
* Video VAE: `minimax_h3_video_vae_int8_convrot.safetensors`
* Audio VAE: `minimax_h3_audio_vae_fp32.safetensors`
* CLIP/text encoder: `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`
* Sampler: `res_multistep`
* Scheduler: `simple`
* Steps: `4`
* Denoise: `1`
* FPS: `24`
* Duration: `10` seconds per pass, approximately `20` seconds final video
* Input scaling: `0.4` megapixels, `32` resolution steps
* Upscale model: `2x_OpenProteus_Compact_i2_70K.pth`
* Attention node: `H3SLAAttention`
* Attention enabled: `true`
* Attention sparsity ratio: `0.85`
* Attention block size: `32`
* Attention minimum sequence length: `8192`
* Attention dense last steps: `1`
* Attention backend: `comfy_kitchen`
* Audio protection: `true`

## License

This repository uses the MIT License for the workflow and documentation. Third-party models, custom nodes and ComfyUI itself are not included and remain under their own licenses.
