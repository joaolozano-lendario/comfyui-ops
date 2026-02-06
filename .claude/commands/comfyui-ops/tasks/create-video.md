# Create Video - Generate High-Quality Frames for Video

Generates production-ready keyframes for video using Gemini, then guides integration with external video tools.

---

## IMPORTANT: LIMITATIONS

**Gemini API CANNOT generate video:**
- ❌ No native video generation (not supported by Gemini API)
- ❌ AnimateDiff/SVD require Stable Diffusion checkpoints (not installed)
- ❌ No temporal coherence between frames (yet)

**What Gemini CAN do:**
- ✅ Generate high-quality individual frames
- ✅ Sequential generation with visual coherence
- ✅ Batch keyframes for video timelines
- ✅ Upscale and enhance frames for video export
- ✅ Multi-panel storyboards (Task Preset #18)

**Future:** When AnimateDiff or SVD checkpoints are installed, native video generation becomes possible.

---

## PRE-FLIGHT

### 1. Load Knowledge
```
Read KB-capabilities.md
Read KB-prompting-heuristics.md
```

### 2. Check ComfyUI
```
mcp__comfyui__get_status
```

Verify:
- ✅ ComfyUI running
- ✅ Gemini API key configured
- ✅ IFGeminiNode available

### 3. Check Video Capabilities
```
mcp__comfyui__list_models (type: checkpoints)
```

**If you see:** SVD, SVD-XT, AnimateDiff, Hunyuan Video, Mochi, LTX Video
→ Native video generation is possible (NOT covered in this task)

**If you see:** Only Gemini or no video models
→ Use this task (frame generation approach)

---

## WORKFLOW OVERVIEW

```
1. Define video concept and keyframes
2. Generate individual frames with Gemini
3. Optional: Upscale and enhance frames
4. Export frames in sequence
5. Integrate with external video tool
```

---

## APPROACH 1: SEQUENTIAL KEYFRAME GENERATION

For videos requiring visual coherence across frames (e.g., before/after, transformation sequence).

### STEP 1: Define Keyframes

Elicit from user:
1. **Video concept:** "Product reveal", "Before/After", "Tutorial steps", etc.
2. **Number of keyframes:** 4-8 for typical video (30-60s at 1-2s per frame)
3. **Transitions:** How each frame differs from previous

**Example:**
```
Video: Product unboxing sequence
Keyframes:
1. Closed box on table (hero shot)
2. Hand lifting box lid (action)
3. Product revealed inside (moment)
4. Product being held up (result)
```

### STEP 2: Build Sequential Workflow

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "KEYFRAME SEQUENCE PROMPT HERE",
        "operation_mode": "generate_images",
        "model_name": "gemini-2.5-flash",
        "temperature": 1.0,
        "aspect_ratio": "16:9",
        "batch_count": 4,
        "sequential_generation": true,
        "seed": 12345,
        "chat_mode": false
      }
    },
    {
      "id": 2,
      "type": "SaveImage",
      "inputs": {
        "images": ["@1"],
        "filename_prefix": "keyframe_sequence"
      }
    }
  ]
}
```

**Key parameters:**
- `sequential_generation: true` → Maintains visual coherence
- `batch_count: 4-8` → Number of keyframes
- `aspect_ratio: "16:9"` → Standard video format
- `seed: fixed` → Reproducibility

### STEP 3: Craft Sequential Prompt

**Template:**
```
Generate a sequence of [N] photorealistic keyframes showing [CONCEPT].

Frame 1: [INITIAL STATE]
Frame 2: [FIRST TRANSITION]
Frame 3: [SECOND TRANSITION]
Frame 4: [FINAL STATE]

Each frame should maintain consistent lighting ([LIGHTING]), camera angle
([CAMERA]), and environment ([ENVIRONMENT]). Captured with [LENS/CAMERA].
16:9 widescreen format.
```

**Example:**
```
Generate a sequence of 4 photorealistic keyframes showing a wireless
earbud unboxing experience.

Frame 1: Pristine white product box on a minimalist desk, closed. Soft
window light from the left creates subtle shadows.

Frame 2: Hand entering frame from the right, fingers gripping the box
lid, slightly lifted (5-10mm gap visible).

Frame 3: Box fully open, revealing the black earbuds nestled in white
foam padding. Hand moved to side of frame.

Frame 4: Hand holding both earbuds up against clean white background,
showing front and side profile.

Each frame should maintain consistent soft diffused window lighting from
the left, same desk surface texture, and same camera position (slight
overhead angle, 30 degrees). Shot with 50mm lens at f/2.8 for shallow
depth of field. Commercial photography quality. 16:9 widescreen format.
```

### STEP 4: Execute and Export

```
mcp__comfyui__validate_workflow (workflow: JSON)

mcp__comfyui__run_workflow (
  workflow: JSON,
  name: "keyframes_{project_name}",
  sync: true,
  imageFormat: "png",
  imageQuality: 100
)
```

Frames are saved as: `keyframe_sequence_00001.png`, `keyframe_sequence_00002.png`, etc.

---

## APPROACH 2: BATCH KEYFRAME GENERATION

For videos where frames are independent (e.g., different angles, A/B variations, storyboard exploration).

### Build Batch Workflow

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "BATCH KEYFRAME PROMPT HERE",
        "operation_mode": "generate_images",
        "model_name": "gemini-2.5-flash",
        "temperature": 1.0,
        "aspect_ratio": "16:9",
        "batch_count": 8,
        "sequential_generation": false,
        "use_random_seed": true,
        "seed": 0
      }
    },
    {
      "id": 2,
      "type": "SaveImage",
      "inputs": {
        "images": ["@1"],
        "filename_prefix": "batch_keyframes"
      }
    }
  ]
}
```

**Key differences:**
- `sequential_generation: false` → Independent frames
- `use_random_seed: true` → Maximum variation
- `batch_count: 8+` → Generate many options

**When to use:**
- Exploring different angles for same shot
- A/B testing compositions
- Generating options for client selection

---

## APPROACH 3: MULTI-PANEL STORYBOARD (Task Preset #18)

For visualizing entire video concept in single image.

### Use IFTaskPromptManager

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFTaskPromptManager",
      "inputs": {
        "task": "Storyboard_And_BRoll",
        "custom_instruction": "Product unboxing sequence: 1) Closed box 2) Opening 3) Reveal 4) Product showcase",
        "append_mode": "after"
      }
    },
    {
      "id": 2,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": ["@1"],
        "operation_mode": "generate_images",
        "model_name": "gemini-2.5-flash",
        "temperature": 1.0,
        "aspect_ratio": "16:9",
        "batch_count": 2
      }
    },
    {
      "id": 3,
      "type": "SaveImage",
      "inputs": {
        "images": ["@2"],
        "filename_prefix": "storyboard"
      }
    }
  ]
}
```

**Result:** Single image with 4-8 panels showing the sequence.

**Use case:** Concept approval, client presentations, planning.

---

## APPROACH 4: HIGH-RES FRAME PIPELINE

For production-quality frames requiring maximum detail.

### Enhanced Pipeline

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "YOUR_PROMPT_HERE",
        "operation_mode": "generate_images",
        "model_name": "gemini-2.5-flash",
        "temperature": 1.0,
        "aspect_ratio": "16:9",
        "batch_count": 1
      }
    },
    {
      "id": 2,
      "type": "ImageUpscaleWithModel",
      "inputs": {
        "image": ["@1"],
        "upscale_model": "RealESRGAN_x4plus"
      }
    },
    {
      "id": 3,
      "type": "FaceRestore",
      "inputs": {
        "image": ["@2"],
        "model": "CodeFormer"
      }
    },
    {
      "id": 4,
      "type": "SaveImage",
      "inputs": {
        "images": ["@3"],
        "filename_prefix": "highres_frame",
        "format": "png",
        "quality": 100
      }
    }
  ]
}
```

**Pipeline stages:**
1. Generate frame at native resolution (1408x768 for 16:9)
2. Upscale 4x with RealESRGAN (5632x3072)
3. Restore faces with CodeFormer (if people present)
4. Export lossless PNG

---

## EXTERNAL INTEGRATION

### Export Specifications

| Target Tool | Format | Resolution | FPS | Notes |
|-------------|--------|-----------|-----|-------|
| **CapCut** | PNG sequence | 1920x1080 | 24/30 | Import as "Image Sequence" |
| **Premiere Pro** | PNG/TIFF | 1920x1080+ | 24/30/60 | File → Import → Image Sequence |
| **DaVinci Resolve** | PNG/EXR | 3840x2160 | 24/30/60 | Media Pool → Import |
| **After Effects** | PNG sequence | Any | Any | Composition → Add to Render Queue |
| **Final Cut Pro** | PNG/ProRes | 1920x1080+ | 24/30/60 | File → Import → Files |

### Frame Numbering

Ensure sequential numbering for import:
```
keyframe_00001.png
keyframe_00002.png
keyframe_00003.png
keyframe_00004.png
...
```

### Duration Per Frame

| Frame Duration | FPS | Use Case |
|---------------|-----|----------|
| 1 second | 24/30 | Standard keyframe hold |
| 2 seconds | 24/30 | Emphasis, storytelling |
| 0.5 seconds | 24/30 | Fast transitions |
| 0.25 seconds | 24/30 | Quick cuts, montage |

---

## SVG ANIMATION OVERLAY

For animated titles, logos, or graphics over frames.

### Use render_svg for Motion Graphics

```
mcp__comfyui__render_svg (
  svg_content: "<svg>...</svg>",
  width: 1920,
  height: 1080,
  output_path: "overlay_title.png"
)
```

**Use cases:**
- Animated title cards
- Logo reveals
- Lower thirds
- Call-to-action overlays

**Workflow:**
1. Generate video frames with Gemini
2. Create SVG animations separately
3. Composite in video editor

---

## EXAMPLE WORKFLOWS

### Example 1: Product Reveal (4 Keyframes)

**Concept:** Wireless earbud reveal

**Frames:**
1. Hero shot (closed box)
2. Unboxing moment (lid lifting)
3. Product reveal (inside box)
4. In-hand showcase

**Prompt (sequential):**
```
Generate a sequence of 4 photorealistic keyframes for a premium wireless
earbud product reveal video.

Frame 1: Pristine matte black product box centered on a marble surface,
closed. Soft studio lighting with three-point setup creates subtle
highlights on box edges. Clean white background fading to light gray.

Frame 2: Same composition, but now elegant fingers grip the box lid,
lifted 2cm to reveal a thin gap of golden interior lighting spilling out.
Anticipation moment.

Frame 3: Box fully open, revealing sleek black earbuds in precision-cut
foam padding. Soft glow from inside box illuminates the product. Hand
repositioned to side of frame.

Frame 4: Close-up of single earbud held delicately between fingers against
pure white background. Studio lighting highlights the matte finish and
metallic accents. Logo visible.

Maintain consistent lighting direction (key light 45° left), marble
surface texture, and camera position (slight overhead, 25° angle).
Shot with 85mm lens at f/2.8 for commercial depth of field. 16:9 format.
```

**Settings:**
- `sequential_generation: true`
- `batch_count: 4`
- `seed: 42000`

### Example 2: Tutorial Steps (6 Keyframes)

**Concept:** Software interface walkthrough

**Prompt (sequential):**
```
Generate a sequence of 6 screen captures showing a clean software
onboarding flow.

Frame 1: Login screen with email field highlighted, cursor hovering.
Frame 2: Dashboard view, welcome tooltip visible in top-right.
Frame 3: Settings panel open, dark mode toggle being clicked.
Frame 4: File upload interface with drag-drop zone emphasized.
Frame 5: Processing animation (spinning loader, 60% progress).
Frame 6: Success screen with green checkmark and "Complete!" message.

Maintain consistent UI design system (dark theme, teal accent color),
same viewport dimensions, and smooth visual progression. Modern SaaS
application aesthetic. 16:9 screen format.
```

**Settings:**
- `sequential_generation: true`
- `batch_count: 6`
- `aspect_ratio: "16:9"`

---

## ADVANCED: VARIATION EXPLORATION

Generate multiple versions of same keyframe for selection.

### Workflow

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "A cinematic establishing shot of a modern office lobby at golden hour",
        "operation_mode": "generate_images",
        "model_name": "gemini-2.5-flash",
        "temperature": 1.0,
        "aspect_ratio": "16:9",
        "batch_count": 8,
        "use_random_seed": true
      }
    }
  ]
}
```

**Result:** 8 variations of same concept, pick best for each keyframe position.

---

## TROUBLESHOOTING

| Problem | Cause | Solution |
|---------|-------|----------|
| **Frames not consistent** | Not using sequential mode | Set `sequential_generation: true` |
| **Low resolution** | Native Gemini output | Use upscale pipeline (Approach 4) |
| **Can't import to editor** | Wrong format/numbering | Ensure PNG sequence with zero-padded numbers |
| **Grey placeholder output** | Generation failed | Check API key, quota, safety filters |
| **Frames too different** | Random seeds | Fix seed value for reproducibility |
| **People look different** | Not sequential + no face consistency | Use sequential mode, reference image in prompt |

---

## COMPARISON: FRAME GENERATION vs NATIVE VIDEO

| Feature | Frame Gen (Now) | Native Video (Future) |
|---------|----------------|----------------------|
| **Temporal coherence** | ⚠️ Limited (sequential helps) | ✅ Full (AnimateDiff/SVD) |
| **Resolution** | ✅ Up to 4K (with upscale) | ⚠️ Usually 1024x576 |
| **Control** | ✅ Full per-frame control | ⚠️ Limited (motion params) |
| **Speed** | ✅ ~30s per frame | ⚠️ 2-10min for video |
| **Editing flexibility** | ✅ Maximum (individual frames) | ❌ Generated as-is |
| **Best for** | Keyframes, storyboards, tutorials | Motion, animation, transitions |

---

## PARAMETERS REFERENCE

| Parameter | Value | Why |
|-----------|-------|-----|
| `operation_mode` | `generate_images` | Always for frames |
| `model_name` | `gemini-2.5-flash` | Fast, cost-effective |
| `temperature` | `1.0` | Google recommendation |
| `aspect_ratio` | `16:9` | Standard video format |
| `batch_count` | `4-8` keyframes | Typical video length |
| `sequential_generation` | `true` for coherence | Visual consistency |
| `use_random_seed` | `false` sequential, `true` batch | Control variation |
| `seed` | Fixed for reproducibility | Consistent results |

---

## FRAME SPECIFICATIONS BY USE CASE

### Social Media

| Platform | Aspect Ratio | Resolution | Duration |
|----------|-------------|-----------|----------|
| Instagram Reel | `9:16` | 1080x1920 | 0.5-1s per frame |
| YouTube Short | `9:16` | 1080x1920 | 1-2s per frame |
| TikTok | `9:16` | 1080x1920 | 0.5-1s per frame |
| Instagram Feed | `1:1` or `4:5` | 1080x1080 / 1080x1350 | 2s per frame |
| YouTube Video | `16:9` | 1920x1080 | 2-3s per frame |

### Professional

| Use Case | Aspect Ratio | Resolution | FPS |
|----------|-------------|-----------|-----|
| Commercial | `16:9` | 3840x2160 (4K) | 24/30 |
| Presentation | `16:9` | 1920x1080 | 30 |
| Tutorial | `16:9` | 1920x1080 | 30 |
| Cinematic | `2.39:1` | 4096x1714 | 24 |

---

## MCP EXECUTION PATTERN

```javascript
// 1. Load KBs
const capabilities = await Read("knowledge/KB-capabilities.md");
const prompting = await Read("knowledge/KB-prompting-heuristics.md");

// 2. Check status
const status = await mcp__comfyui__get_status();
console.log("ComfyUI ready:", status.ready);

// 3. Build workflow
const workflow = {
  nodes: [
    {
      id: 1,
      type: "IFGeminiNode",
      inputs: {
        prompt: sequentialPrompt,
        operation_mode: "generate_images",
        model_name: "gemini-2.5-flash",
        temperature: 1.0,
        aspect_ratio: "16:9",
        batch_count: 6,
        sequential_generation: true,
        seed: 42000
      }
    },
    {
      id: 2,
      type: "SaveImage",
      inputs: {
        images: ["@1"],
        filename_prefix: "keyframes_product_reveal"
      }
    }
  ]
};

// 4. Validate
const validation = await mcp__comfyui__validate_workflow({ workflow });

// 5. Execute
if (validation.valid) {
  const result = await mcp__comfyui__run_workflow({
    workflow,
    name: "product_reveal_keyframes",
    sync: true,
    imageFormat: "png",
    imageQuality: 100
  });

  console.log("Frames generated:", result.outputs);
}
```

---

## SUCCESS METRICS

| Metric | Target | How to Measure |
|--------|--------|----------------|
| **Visual coherence** | 90%+ consistency | Sequential frames feel connected |
| **Frame quality** | 4K-ready | After upscale, suitable for 1080p+ delivery |
| **Production time** | < 5min for 8 frames | Gemini generation + export |
| **Usability** | Seamless import | External tool accepts sequence without issues |

---

## WHEN TO USE THIS vs OTHER APPROACHES

| Goal | Use This | Use Instead |
|------|----------|-------------|
| Keyframes for video | ✅ Yes | — |
| Storyboard visualization | ✅ Yes (Task #18) | — |
| Tutorial screenshots | ✅ Yes | — |
| Product reveal sequence | ✅ Yes | — |
| Smooth motion video | ❌ No | Wait for AnimateDiff/SVD |
| Character animation | ❌ No | Wait for SVD/Mochi |
| Camera movement | ❌ No | Wait for native video models |

---

## FUTURE: NATIVE VIDEO GENERATION

When Stable Diffusion video models are installed:

### Models to Look For

| Model | Type | Frames | Quality |
|-------|------|--------|---------|
| **SVD** | img2vid | 14 (~0.5s) | Good |
| **SVD-XT** | img2vid | 25 (~1s) | Good |
| **AnimateDiff** | txt2vid | Variable | Excellent |
| **Hunyuan Video** | txt2vid/img2vid | Variable | Excellent |
| **Mochi** | txt2vid | Variable | Good |
| **LTX Video** | txt2vid | Variable | Good |

### Check Availability

```
mcp__comfyui__list_models (type: checkpoints)
```

If video models present, ask user if they want native video generation guidance.

---

*ComfyUI Ops Squad*
*Task: create-video.md*
*Updated: 2026-02-06*
*Status: Production Ready*
