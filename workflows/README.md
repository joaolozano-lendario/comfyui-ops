# ComfyUI-Ops Workflow JSONs

Pre-built API-format workflows ready to use with `mcp__comfyui__run_workflow`.

## Workflows

| File | Pipeline | Use Case |
|------|----------|----------|
| `txt2img-gemini-basic.json` | IFGeminiNode → SaveImage | Quick generation, ideation, drafts |
| `txt2img-gemini-upscale.json` | IFGeminiNode → RealESRGAN 4x → SaveImage | Social media posts, web assets |
| `txt2img-gemini-face.json` | IFGeminiNode → CodeFormer → UltraSharp 4x → SaveImage | Portraits, headshots, any image with faces |
| `img2img-gemini.json` | LoadImage → IFGeminiNode → SaveImage | Transform existing image, style transfer |
| `svg-enhance.json` | LoadImage → IFGeminiNode → RealESRGAN 4x → SaveImage | SVG layout → diffusion-enhanced image |
| `upscale-only.json` | LoadImage → UltraSharp 4x → SaveImage | Upscale any existing image (no AI generation) |

## How to Use

### 1. Via MCP (recommended)

Load a workflow JSON file, replace placeholder values, and run:

```
# Read the workflow file
# Replace YOUR_PROMPT_HERE with your actual prompt
# Replace YOUR_INPUT_IMAGE.png with your actual filename (for img2img workflows)
# Pass to mcp__comfyui__run_workflow
```

### 2. Placeholder Values

All workflows use these placeholders — replace before execution:

| Placeholder | Replace With |
|-------------|-------------|
| `YOUR_PROMPT_HERE` | Your narrative prompt (see knowledge/KB-prompting-heuristics.md) |
| `YOUR_INPUT_IMAGE.png` | Filename of image in ComfyUI's input directory |
| `YOUR_SVG_RENDER.png` | SVG rendered to PNG via `mcp__comfyui__render_svg` |

### 3. Customization

**Change aspect ratio:** Edit `aspect_ratio` in the IFGeminiNode inputs. Options: `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `4:5`, `5:4`

**Change upscale model:** Replace `RealESRGAN_x4plus.pth` with `4x-UltraSharp.pth` (or vice versa). UltraSharp is better for faces/skin; RealESRGAN is better for general scenes.

**Change face restore fidelity:** Adjust `fidelity` in FaceRestoreCFCodeFormer (0.5 = more cosmetic, 1.0 = more faithful to original).

**Change seed:** Set `seed` to a specific number for reproducibility, or `0` for random.

## Requirements

- ComfyUI running at `http://127.0.0.1:8188`
- ComfyUI-IF_Gemini custom node installed with Gemini API key
- Upscale models: `RealESRGAN_x4plus.pth` and/or `4x-UltraSharp.pth`
- Face restore: `codeformer.pth` (only for `txt2img-gemini-face.json`)

Run `/comfyui-ops:tasks:config` to verify all prerequisites are met.
