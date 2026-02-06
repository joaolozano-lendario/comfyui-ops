# ComfyUI-Ops

AI-powered visual asset factory using ComfyUI + Claude Code.

5 specialized agents orchestrate image generation, upscaling, face restoration, and brand-consistent asset creation through 37+ MCP tools.

## Prerequisites

- **Node.js 18+** — [nodejs.org](https://nodejs.org)
- **ComfyUI** installed and running — [github.com/comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- **Claude Code CLI** — [docs.anthropic.com/claude-code](https://docs.anthropic.com/en/docs/claude-code)
- **ComfyUI-IF_Gemini** custom node installed in ComfyUI (for Gemini API generation)
- **Gemini API key** configured in ComfyUI-IF_Gemini `.env`

### Hardware Requirements

| Tier | VRAM | Capabilities |
|------|------|-------------|
| **cpu-only** | <2 GB | Gemini API generation only, no local upscale |
| **low** | 2-6 GB | Gemini API + lightweight upscale (RealESRGAN) |
| **medium** | 6-8 GB | Full pipeline (Gemini + CodeFormer + upscale) |
| **high** | 12+ GB | Full pipeline + SDXL local generation possible |

Run `/comfyui-ops:tasks:config` to auto-detect your tier and get tailored recommendations.

### Required ComfyUI Models

These should be in your ComfyUI `models/` directory:

| Type | Model | Path |
|------|-------|------|
| Upscale | `RealESRGAN_x4plus.pth` | `models/upscale_models/` |
| Upscale | `4x-UltraSharp.pth` | `models/upscale_models/` |
| Face Restore | `codeformer.pth` | `models/facelib/` |

## Setup

### 1. Clone this repo

```bash
git clone --recurse-submodules https://github.com/YOUR_USERNAME/comfyui-ops.git
cd comfyui-ops
```

### 1b. Run the setup wizard (recommended)

```
claude
/comfyui-ops:tasks:config
```

This auto-detects your hardware, verifies dependencies, and runs a smoke test.

### 2. Build the MCP server

```bash
cd comfyui-mcp
npm install
npm run build
cd ..
```

### 3. Start ComfyUI

```bash
# Windows with DirectML (AMD/Intel GPU)
cd D:\ComfyUI && .venv\Scripts\activate && python main.py --directml

# Windows with CUDA (NVIDIA GPU)
cd D:\ComfyUI && .venv\Scripts\activate && python main.py

# Linux/Mac
cd ~/ComfyUI && source venv/bin/activate && python main.py
```

Verify it's running at `http://127.0.0.1:8188`.

### 4. Open with Claude Code

```bash
claude --project D:\comfyui-ops
```

Or just `cd D:\comfyui-ops && claude`.

### 5. Test the connection

In Claude Code, the MCP server should auto-connect. Try:

```
/comfyui-ops:agents:comfyui-chief
```

This activates the orchestrator agent, which will check the ComfyUI connection and present a status summary.

## Usage

### Quick generation

Just describe what you want:

```
Generate a professional portrait photo with cinematic lighting
```

### Use specific tasks

```
/comfyui-ops:tasks:create-banner
/comfyui-ops:tasks:create-thumbnail
/comfyui-ops:tasks:generate-image
```

### Use specialist agents

```
/comfyui-ops:agents:prompt-architect    # For crafting optimal prompts
/comfyui-ops:agents:visual-director     # For art direction
/comfyui-ops:agents:workflow-engineer   # For custom workflows
/comfyui-ops:agents:quality-inspector   # For QA and approval
```

## Brand Configuration (Optional)

Create a `brand-config.yaml` at the repo root to use consistent brand colors:

```yaml
brand:
  name: "Your Brand"
  colors:
    primary: "#FFD44A"
    secondary: "#0A0A0A"
    accent_1: "#22d3ee"
    accent_2: "#22c55e"
```

Without this file, the system operates in generic mode.

## Directory Structure

```
comfyui-ops/
├── CLAUDE.md              # Claude Code instructions
├── README.md              # This file
├── .mcp.json              # MCP server config
├── .claude/               # Claude Code settings + agent/task definitions
├── knowledge/             # 8 knowledge base files
├── data/                  # Reference data (presets, platform specs)
├── workflows/             # 6 executable workflow JSONs
├── models/                # User's custom models (empty by default)
├── image-bank/            # Reference images
├── comfyui-mcp/           # MCP server (git submodule)
└── outputs/               # Generated outputs (gitignored)
```

## Models

The `models/` directory is empty by default. Use the setup wizard or the model setup task to download recommended models:

```
/comfyui-ops:tasks:config       # First-time setup — recommends models for your hardware tier
/comfyui-ops:tasks:setup-model  # Download/configure specific models anytime
```

## Workflows

Pre-built API-format workflow JSONs in `workflows/`:

| File | Pipeline |
|------|----------|
| `txt2img-gemini-basic.json` | IFGeminiNode → SaveImage |
| `txt2img-gemini-upscale.json` | IFGeminiNode → RealESRGAN 4x → SaveImage |
| `txt2img-gemini-face.json` | IFGeminiNode → CodeFormer → UltraSharp 4x → SaveImage |
| `img2img-gemini.json` | LoadImage → IFGeminiNode → SaveImage |
| `svg-enhance.json` | LoadImage → IFGeminiNode → RealESRGAN 4x → SaveImage |
| `upscale-only.json` | LoadImage → UltraSharp 4x → SaveImage |

See `workflows/README.md` for usage details and placeholder values.

## Troubleshooting

**ComfyUI not connecting:** Check that ComfyUI is running at `http://127.0.0.1:8188` and the `.mcp.json` path to `comfyui-mcp/dist/index.js` is correct.

**MCP tools not available:** Run `cd comfyui-mcp && npm run build` to compile the MCP server.

**Gray/placeholder images:** Make sure `operation_mode` is set to `"generate_images"` in the workflow, and the prompt describes an image (not a question).

**Faces look distorted:** The system should auto-apply CodeFormer for face restoration. If not, use `/comfyui-ops:tasks:upscale-image` with face restore enabled.

## License

MCP server (`comfyui-mcp/`) is MIT licensed — see [shawnrushefsky/comfyui-mcp](https://github.com/shawnrushefsky/comfyui-mcp).
