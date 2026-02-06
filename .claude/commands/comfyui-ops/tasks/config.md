# Config — First-Time Setup Wizard
## Detects hardware, verifies dependencies, configures the system, and runs a smoke test

---
type: TASK
squad: comfyui-ops
priority: HIGHEST
created: 2026-02-06
version: 1.0.0
requires: [KB-system-inventory.md]
agents: [forge]
---

## OVERVIEW

Interactive 9-phase wizard that takes a fresh install from zero to first generation.
Run this ONCE after cloning the repo. Re-run anytime to verify system health.

---

## PHASE 1: SYSTEM DIAGNOSTICS

### Detect Hardware

```
DETECT:
1. OS: Check platform (win32, linux, darwin)
2. GPU:
   - NVIDIA: Run `nvidia-smi --query-gpu=name,memory.total --format=csv,noheader`
   - AMD/Intel (Windows): Run `wmic path win32_VideoController get Name,AdapterRAM`
   - Fallback: Ask user
3. RAM: Check total system RAM
4. Disk: Check free space on drive where ComfyUI lives
```

### Assign Hardware Tier

```
TIERS:
- cpu-only: No GPU detected or <2GB VRAM → Gemini API only, no local upscale
- low: 2-6GB VRAM → Gemini API + lightweight upscale (RealESRGAN)
- medium: 6-8GB VRAM → Full pipeline (Gemini + CodeFormer + upscale)
- high: 12+GB VRAM → Full pipeline + SDXL local generation possible
```

### Output

```
PRINT SUMMARY:
┌─────────────────────────────────────┐
│ SYSTEM DIAGNOSTICS                  │
├─────────────────────────────────────┤
│ OS:       Windows 11                │
│ GPU:      AMD Radeon RX 6700 XT     │
│ VRAM:     12 GB                     │
│ RAM:      32 GB                     │
│ Disk:     245 GB free               │
│ Tier:     HIGH                      │
└─────────────────────────────────────┘
```

---

## PHASE 2: COMFYUI CHECK

### Verify ComfyUI Running

```
ACTION:
1. Use mcp__comfyui__get_status to ping http://127.0.0.1:8188
2. If connected → print version, continue
3. If NOT connected → print install guide:

NOT CONNECTED:
  "ComfyUI is not running at http://127.0.0.1:8188"

  OPTIONS:
  1. "I have ComfyUI installed, I'll start it now" → wait and retry
  2. "I need to install ComfyUI" → run mcp__comfyui__get_install_guide
  3. "It's running on a different port" → ask for URL (not supported by MCP, note limitation)

IF user selects 1:
  Print startup command based on OS/GPU detected in Phase 1:
  - NVIDIA: cd D:\ComfyUI && .venv\Scripts\activate && python main.py
  - AMD/Intel: cd D:\ComfyUI && .venv\Scripts\activate && python main.py --directml
  - Linux: cd ~/ComfyUI && source venv/bin/activate && python main.py

  Wait for user to confirm, then retry mcp__comfyui__get_status
```

---

## PHASE 3: MCP SERVER CHECK

### Verify MCP Server

```
ACTION:
1. Check if comfyui-mcp/dist/index.js exists (Glob or Read)
2. If exists → "MCP server built and ready"
3. If NOT exists:
   a. Check if comfyui-mcp/ directory exists
   b. If directory exists but no dist/ → run: cd comfyui-mcp && npm install && npm run build
   c. If directory missing → "comfyui-mcp submodule not initialized. Run: git submodule update --init"

VERIFY:
- After build, confirm dist/index.js exists
- Check .mcp.json points to correct path
```

---

## PHASE 4: CUSTOM NODES CHECK

### Verify IFGeminiNode

```
ACTION:
1. Use mcp__comfyui__list_nodes with search "IFGemini"
2. If found → "IFGeminiNode detected"
3. If NOT found:
   "ComfyUI-IF_Gemini custom node not installed."
   "Install via ComfyUI Manager or manually:"
   "  cd D:\ComfyUI\custom_nodes"
   "  git clone https://github.com/if-ai/ComfyUI-IF_Gemini"
   "  Restart ComfyUI"
```

### Verify Gemini API Key

```
ACTION:
1. Attempt a minimal generation (prompt: "A red circle on white background", batch_count: 1)
   using txt2img-gemini-basic.json workflow
2. If succeeds → "Gemini API key is configured and working"
3. If fails with auth error:
   "Gemini API key not configured."
   "1. Get a key at https://aistudio.google.com/app/apikey"
   "2. Create .env file in ComfyUI-IF_Gemini directory:"
   "   GEMINI_API_KEY=your_key_here"
   "3. Restart ComfyUI"

DO NOT ask user to paste API key directly — security risk.
```

---

## PHASE 5: MODEL RECOMMENDATIONS

### Check Installed Models

```
ACTION:
1. Use mcp__comfyui__list_models for each type:
   - upscale_models
   - checkpoints
   - loras
2. Cross-reference with requirements per tier:

REQUIRED (all tiers except cpu-only):
  - RealESRGAN_x4plus.pth (upscale)
  - 4x-UltraSharp.pth (upscale)
  - codeformer.pth (face restore — check facerestore_models)

RECOMMENDED (high tier):
  - Juggernaut XL v9 checkpoint (~6.6GB) — enables local SDXL generation
  - Skin Realism SDXL LoRA — portrait enhancement
```

### Offer Downloads

```
FOR EACH MISSING REQUIRED MODEL:
1. Use mcp__comfyui__get_download_url with model name
2. Present download command
3. Ask user: "Download now? (Y/n)"

EXAMPLE OUTPUT:
  "Missing: RealESRGAN_x4plus.pth (upscale model)"
  "Download: wget -O models/upscale_models/RealESRGAN_x4plus.pth https://..."

FOR RECOMMENDED MODELS:
  Present as optional with size warning:
  "Optional: Juggernaut XL v9 (~6.6GB) — enables local SDXL generation"
  "Your tier (HIGH) supports this. Download? (Y/n/later)"
```

---

## PHASE 6: BRAND CONFIGURATION

### Ask About Brand

```
ASK USER (AskUserQuestion):
  "Do you want to configure brand colors for consistent asset generation?"

  OPTIONS:
  1. "Yes, I have brand colors" → collect brand info
  2. "Use defaults" → create brand-config.yaml with example colors
  3. "Skip for now" → no brand-config.yaml created

IF option 1:
  ASK:
  - Brand name
  - Primary color (hex)
  - Secondary color (hex)
  - Accent color 1 (hex, optional)
  - Accent color 2 (hex, optional)
  - Product tiers? (name + color for each, optional)

  CREATE brand-config.yaml:
  ```yaml
  brand:
    name: "User's Brand"
    colors:
      primary: "#XXXXXX"
      secondary: "#XXXXXX"
      accent_1: "#XXXXXX"
      accent_2: "#XXXXXX"
    products:
      tier_1:
        name: "Starter"
        color: "#XXXXXX"
  ```

IF option 2:
  CREATE brand-config.yaml with placeholder values:
  ```yaml
  brand:
    name: "My Brand"
    colors:
      primary: "#3B82F6"
      secondary: "#0A0A0A"
      accent_1: "#22d3ee"
      accent_2: "#22c55e"
  ```

IF option 3:
  Skip. System operates in generic mode.
```

---

## PHASE 7: USE CASE DETECTION

### Ask Primary Use Case

```
ASK USER (AskUserQuestion):
  "What will you primarily use ComfyUI-Ops for?"

  OPTIONS:
  1. "Portraits & headshots" → face pipeline defaults
  2. "Marketing assets (banners, thumbnails, ads)" → commercial pipeline defaults
  3. "Product photography" → product pipeline defaults
  4. "General image generation" → balanced defaults

SET DEFAULTS based on selection:

Option 1 (Portraits):
  default_workflow: txt2img-gemini-face.json
  default_aspect_ratio: "1:1"
  default_upscaler: "4x-UltraSharp.pth"
  face_restore: always
  default_temperature: 0.9

Option 2 (Marketing):
  default_workflow: txt2img-gemini-upscale.json
  default_aspect_ratio: "16:9"
  default_upscaler: "RealESRGAN_x4plus.pth"
  face_restore: when_detected
  default_temperature: 1.0

Option 3 (Product):
  default_workflow: txt2img-gemini-upscale.json
  default_aspect_ratio: "4:3"
  default_upscaler: "4x-UltraSharp.pth"
  face_restore: never
  default_temperature: 0.8

Option 4 (General):
  default_workflow: txt2img-gemini-basic.json
  default_aspect_ratio: "16:9"
  default_upscaler: "RealESRGAN_x4plus.pth"
  face_restore: when_detected
  default_temperature: 1.0

SAVE to MCP notes:
  mcp__comfyui__save_note(
    topic: "user-config",
    content: YAML string of defaults
  )
```

---

## PHASE 8: SMOKE TEST

### Run Test Generation

```
ACTION:
1. Load txt2img-gemini-basic.json from workflows/
2. Replace prompt with: "A single red apple on a clean white surface, soft studio lighting, product photography style"
3. Run via mcp__comfyui__run_workflow with sync: true
4. Check result:

IF SUCCESS:
  "Smoke test PASSED — image generated successfully"
  Show the generated image to user

  OPTIONAL (if face restore + upscale available):
  "Run full pipeline test? (Y/n)"
  If yes: Run txt2img-gemini-face.json with portrait prompt

IF FAILURE:
  Parse error message:
  - "API key" → "Gemini API key issue — revisit Phase 4"
  - "node not found" → "IFGeminiNode not installed — revisit Phase 4"
  - "connection" → "ComfyUI not running — revisit Phase 2"
  - Other → Print full error, suggest checking ComfyUI console logs
```

---

## PHASE 9: SUMMARY

### Print Configuration Summary

```
PRINT:
┌─────────────────────────────────────────────────────┐
│ COMFYUI-OPS CONFIGURATION COMPLETE                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│ System                                              │
│   OS:           Windows 11                          │
│   GPU:          AMD Radeon RX 6700 XT (12GB)        │
│   Tier:         HIGH                                │
│                                                     │
│ ComfyUI                                             │
│   Status:       Connected (v0.12.3)                 │
│   MCP Server:   Built and linked                    │
│   IFGemini:     Installed                           │
│   Gemini API:   Configured                          │
│                                                     │
│ Models                                              │
│   Upscale:      RealESRGAN ✅  UltraSharp ✅         │
│   Face:         CodeFormer ✅                        │
│   Checkpoints:  None (Gemini API only)              │
│                                                     │
│ Configuration                                       │
│   Brand:        Configured / Defaults / Skipped     │
│   Use Case:     Marketing assets                    │
│   Default:      txt2img-gemini-upscale.json         │
│                                                     │
│ Smoke Test:     PASSED ✅                            │
│                                                     │
├─────────────────────────────────────────────────────┤
│ NEXT STEPS                                          │
│                                                     │
│ 1. Try: /comfyui-ops:agents:comfyui-chief           │
│ 2. Or:  /comfyui-ops:tasks:generate-image           │
│ 3. Or just describe what you want to generate       │
│                                                     │
│ For SDXL local generation (high tier only):         │
│   /comfyui-ops:tasks:setup-model                    │
└─────────────────────────────────────────────────────┘
```

### Save Configuration

```
ACTION:
1. Save full config to MCP notes:
   mcp__comfyui__save_note(
     topic: "system-config",
     content: full YAML summary of all detected values,
     tags: ["config", "setup", hardware_tier]
   )

2. Print: "Configuration saved. Run /comfyui-ops:tasks:config anytime to re-check."
```

---

## ERROR RECOVERY

| Phase | Common Error | Recovery |
|-------|-------------|----------|
| 1 | Can't detect GPU | Ask user manually, default to cpu-only |
| 2 | ComfyUI not found | Provide install guide link |
| 3 | npm not installed | "Install Node.js 18+ from nodejs.org" |
| 4 | IFGemini missing | Provide git clone command |
| 4 | API key invalid | Direct to aistudio.google.com |
| 5 | Download fails | Provide manual download URL |
| 8 | Smoke test fails | Diagnose based on error, loop back to relevant phase |

---

## NOTES

- This wizard is idempotent — safe to run multiple times
- Each phase can be run independently if needed
- Hardware tier affects model recommendations, not capabilities
- The wizard does NOT modify ComfyUI installation, only verifies it
- Brand config and user preferences are saved as files/notes for persistence
