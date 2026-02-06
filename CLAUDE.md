# ComfyUI-Ops: AI Visual Asset Factory

Operate ComfyUI as a professional visual asset factory via Claude Code + MCP tools.

5 agents | 13 tasks | 8 knowledge base files | 6 workflow JSONs | 37+ MCP tools

---

## QUICK START

1. **First time?** Run `/comfyui-ops:tasks:config` — interactive wizard that checks hardware, dependencies, and runs a smoke test
2. Ensure ComfyUI is running: `http://127.0.0.1:8188`
3. Run `/comfyui-ops:agents:comfyui-chief` to activate the orchestrator
4. Describe what you want: "generate a professional portrait" or "create a YouTube thumbnail"

---

## AGENTS

| Agent | Command | Role |
|-------|---------|------|
| **Pixel** (ComfyUI Chief) | `/comfyui-ops:agents:comfyui-chief` | Orchestrator — classifies requests, routes to specialists or handles inline |
| **Lyra** (Prompt Architect) | `/comfyui-ops:agents:prompt-architect` | Crafts state-of-art narrative prompts (94% vs 61% coherence) |
| **Kael** (Visual Director) | `/comfyui-ops:agents:visual-director` | Art direction, composition, color theory, brand consistency |
| **Forge** (Workflow Engineer) | `/comfyui-ops:agents:workflow-engineer` | Builds/debugs ComfyUI workflows, multi-stage pipelines |
| **Iris** (Quality Inspector) | `/comfyui-ops:agents:quality-inspector` | QA visual, iteration, refinement, approval |

**When to use which:**
- Simple request ("generate a photo of X") → Pixel handles inline
- Complex art direction → Pixel delegates to Kael + Lyra
- Workflow issues → Pixel delegates to Forge
- Quality approval needed → Pixel delegates to Iris

---

## TASKS

| # | Task | Command | When to Use |
|---|------|---------|-------------|
| 0 | **Config wizard** | `/comfyui-ops:tasks:config` | **First-time setup, system health check** |
| 1 | Generate image | `/comfyui-ops:tasks:generate-image` | txt2img with smart selection |
| 2 | From reference | `/comfyui-ops:tasks:generate-from-reference` | img2img |
| 3 | Brand asset | `/comfyui-ops:tasks:create-brand-asset` | With brand design system |
| 4 | Banner | `/comfyui-ops:tasks:create-banner` | Platform-optimized banners |
| 5 | Thumbnail | `/comfyui-ops:tasks:create-thumbnail` | YouTube, course thumbnails |
| 6 | Batch variations | `/comfyui-ops:tasks:batch-variations` | Multiple options for selection |
| 7 | Upscale | `/comfyui-ops:tasks:upscale-image` | Increase resolution |
| 8 | Inpaint | `/comfyui-ops:tasks:inpaint-edit` | Edit specific regions |
| 9 | Style transfer | `/comfyui-ops:tasks:style-transfer` | Apply visual style |
| 10 | SVG to image | `/comfyui-ops:tasks:svg-to-image` | Layout → diffusion |
| 11 | Video | `/comfyui-ops:tasks:create-video` | Image → video frames |
| 12 | Setup model | `/comfyui-ops:tasks:setup-model` | Download/configure models |

---

## KNOWLEDGE BASE

**CRITICAL RULE:** Always load relevant KB files before any generation.

| KB File | Content | When to Load |
|---------|---------|-------------|
| `knowledge/KB-capabilities.md` | IF_Gemini features, 38 presets, params | Before ANY generation |
| `knowledge/KB-prompting-heuristics.md` | Narrative prompting, master template | Before ANY generation |
| `knowledge/KB-workflow-patterns.md` | 7 workflow architectures, JSON templates | When building workflows |
| `knowledge/KB-pipeline-reference.md` | 4-stage pipeline, face restore, upscale | For quality pipeline |
| `knowledge/KB-system-inventory.md` | Installed models, paths, API config | On activation |
| `knowledge/KB-prompt-patterns.md` | Prompt pattern library | For prompt crafting |
| `knowledge/KB-sdxl-models.md` | SDXL model reference | When using SDXL |
| `knowledge/KB-sdxl-workflows.md` | SDXL workflow patterns | When using SDXL |

**Loading priority:**
1. KB-system-inventory (what's installed)
2. KB-prompting-heuristics (how to prompt)
3. KB-capabilities (what's possible)
4. Others as needed

---

## MCP TOOLS

All ComfyUI operations use MCP tools prefixed with `mcp__comfyui__`.

**Core generation:**
- `mcp__comfyui__run_workflow` — Execute workflow JSON
- `mcp__comfyui__validate_workflow` — Validate before execution
- `mcp__comfyui__get_image` — Retrieve generated image

**Discovery:**
- `mcp__comfyui__list_models` — Available checkpoints, LoRAs, upscalers
- `mcp__comfyui__list_nodes` — Available nodes
- `mcp__comfyui__get_node_info` — Node details

**Templates:**
- `mcp__comfyui__save_template` / `get_template` / `search_templates`

**Utilities:**
- `mcp__comfyui__render_svg` — SVG to PNG
- `mcp__comfyui__download_font` — Google Fonts
- `mcp__comfyui__save_note` / `get_notes` / `search_notes`

**Status:**
- `mcp__comfyui__get_status` — Check connection
- `mcp__comfyui__get_queue` — Queue status
- `mcp__comfyui__get_history` — Generation history

---

## BRAND CONFIGURATION (Optional)

To use brand colors consistently, create a `brand-config.yaml` at the repo root:

```yaml
brand:
  name: "Your Brand Name"
  colors:
    primary: "#FFD44A"
    secondary: "#0A0A0A"
    accent_1: "#22d3ee"
    accent_2: "#22c55e"
  products:
    tier_1:
      name: "Starter"
      color: "#10B981"
    tier_2:
      name: "Pro"
      color: "#3B82F6"
    tier_3:
      name: "Enterprise"
      color: "#C9A227"
```

If no `brand-config.yaml` exists, the system operates in generic mode and asks for color preferences when needed.

---

## WORKFLOWS

Pre-built API-format JSONs in `workflows/` — ready to use with `mcp__comfyui__run_workflow`.

| Workflow | Pipeline | Use Case |
|----------|----------|----------|
| `txt2img-gemini-basic.json` | IFGeminiNode → SaveImage | Quick generation, ideation |
| `txt2img-gemini-upscale.json` | IFGeminiNode → RealESRGAN 4x → SaveImage | Social media, web assets |
| `txt2img-gemini-face.json` | IFGeminiNode → CodeFormer → UltraSharp 4x → SaveImage | Portraits, headshots |
| `img2img-gemini.json` | LoadImage → IFGeminiNode → SaveImage | Transform existing image |
| `svg-enhance.json` | LoadImage → IFGeminiNode → RealESRGAN 4x → SaveImage | SVG → diffusion-enhanced |
| `upscale-only.json` | LoadImage → UltraSharp 4x → SaveImage | Upscale without AI generation |

Replace `YOUR_PROMPT_HERE` and `YOUR_INPUT_IMAGE.png` placeholders before execution. See `workflows/README.md` for details.

---

## GOLDEN RULES

1. **NARRATIVE > KEYWORDS** — Write prompts as stories, not tag lists (94% vs 61% coherence)
2. **ALWAYS load KBs** — At minimum KB-prompting-heuristics + KB-system-inventory
3. **ALWAYS specify aspect_ratio** — Never leave it undefined
4. **MCP first, REST fallback** — Use `mcp__comfyui__run_workflow`, REST API only if MCP fails
5. **No negative prompts** — Gemini doesn't support them; use positive reframing
6. **1 change per iteration** — 94% consistency vs 41% with multiple changes
7. **CodeFormer for faces** — Always add face restore before upscale for portraits
8. **Batch for quality** — Generate batch:4, select best, apply full pipeline

---

## DIRECTORY STRUCTURE

```
comfyui-ops/
├── CLAUDE.md                    # This file
├── README.md                    # Setup guide
├── brand-config.yaml            # Your brand colors (create this)
├── .mcp.json                    # MCP server config
├── .claude/
│   ├── settings.local.json      # MCP permissions
│   └── commands/comfyui-ops/
│       ├── agents/              # 5 agent definitions
│       └── tasks/               # 12 task definitions
├── knowledge/                   # 8 KB files
├── workflows/                   # 6 executable workflow JSONs (API format)
├── data/                        # Reference data (presets, specs)
├── models/                      # User's custom models (empty by default)
└── outputs/                     # Generated outputs (gitignored)
```

---

## TROUBLESHOOTING

| Problem | Solution |
|---------|---------|
| ComfyUI not connected | Start ComfyUI: `cd D:\ComfyUI && .venv\Scripts\activate && python main.py --directml` |
| Gray/placeholder output | Check `operation_mode` is "generate_images", not text |
| MCP tools not working | Check `.mcp.json` path and run `npm run build` in comfyui-mcp |
| Faces distorted | Add CodeFormer (fidelity 0.5-0.7) before upscale |
| Colors don't match | Use hex codes in prompts: "gold (#FFD44A)" |
| Prompt not working | Load KB-prompting-heuristics.md, use narrative structure |
| Low resolution | Add upscale pipeline (4x-UltraSharp or RealESRGAN) |
