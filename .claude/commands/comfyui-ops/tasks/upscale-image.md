# Upscale Image - Aumentar Resolucao com Pipeline Profissional

Aumenta resolução de imagens com pipeline profissional incluindo restauração de rostos e controle de qualidade.

---

## PRE-FLIGHT CHECKS

### 1. Knowledge Base Status
Confirm access to:
- `knowledge/KB-capabilities.md` - IFGeminiNode output resolution limits
- `knowledge/KB-pipeline-reference.md` - Stage 2 (Upscale) + Stage 3 (Face Restore)
- `knowledge/KB-system-inventory.md` - Section 3 (Available upscale models & face restoration)
- `knowledge/KB-workflow-patterns.md` - Upscale workflow patterns

### 2. MCP Connection
```bash
# Verify ComfyUI is running
curl -s http://127.0.0.1:8188/system_stats
```

Test MCP tools:
```
mcp__comfyui__get_status
mcp__comfyui__list_models(type: "upscale_models")
mcp__comfyui__list_models(type: "codeformer")
```

### 3. Verify Available Models

**CRITICAL CHECK:** Confirm models are installed before proceeding.

```python
# Check upscale models
upscale_models = mcp__comfyui__list_models(type: "upscale_models")
# Expected: ["RealESRGAN_x4plus.pth", "4x-UltraSharp.pth"]

# Check face restoration models
face_models = mcp__comfyui__list_models(type: "codeformer")
# Expected: ["codeformer.pth"]
```

**If models missing:** See "Troubleshooting Issue 1" for download instructions.

---

## CRITICAL RULE: FACE RESTORATION SEQUENCE

```
┌──────────────────────────────────────────────────────────┐
│  NEVER UPSCALE FACES WITHOUT RESTORATION FIRST           │
│                                                           │
│  ❌ WRONG: Generate → Upscale → CodeFormer              │
│  ✅ RIGHT: Generate → CodeFormer → Upscale              │
│                                                           │
│  Upscaling amplifies facial artifacts. Always restore    │
│  faces BEFORE upscaling to avoid magnifying distortions. │
└──────────────────────────────────────────────────────────┘
```

---

## WORKFLOW

### 1. Analyze Input Image (2-3 minutes)

Before selecting pipeline, evaluate image characteristics:

| Question | Determines |
|----------|------------|
| **Does image have faces?** | CodeFormer required? |
| **What's the subject?** | Model selection (RealESRGAN vs UltraSharp) |
| **Current resolution?** | Target resolution (2x vs 4x) |
| **Final use case?** | Output format (PNG vs JPEG) |
| **Quality issues?** | Face restoration fidelity level |

### 2. Pipeline Decision Tree

```
START: Image needs upscaling
│
├─ Has faces (people/portraits)?
│  ├─ YES
│  │  └─ Pipeline: CodeFormer → Upscale
│  │     • Face restore FIRST (fidelity 0.5-1.0)
│  │     • Then upscale (4x-UltraSharp recommended)
│  │
│  └─ NO
│     └─ Pipeline: Direct Upscale
│        • Choose model by subject type
│        • RealESRGAN: landscapes, products, general
│        • 4x-UltraSharp: detail-heavy, textures, architecture
│
└─ From generation pipeline?
   └─ Build integrated workflow (Stage 1 → 2 → 3)
```

### 3. Upscale Model Selection

**Available Models:**

| Model | Best For | Characteristics | When to Use |
|-------|----------|-----------------|-------------|
| **RealESRGAN_x4plus.pth** | General purpose | Slight smoothing, good for landscapes/products | Default choice, safe for most images |
| **4x-UltraSharp.pth** | Detail preservation | Maximum sharpness, excellent for portraits/textures | When detail is critical, professional work |

**Selection Criteria:**

| Image Type | Recommended Model | Reasoning |
|------------|------------------|-----------|
| **Portraits (with faces)** | 4x-UltraSharp | Preserves skin texture after CodeFormer |
| **Landscapes** | RealESRGAN_x4plus | Natural smoothing prevents over-sharpening |
| **Products/Objects** | RealESRGAN_x4plus | Balanced detail without artifacts |
| **Architecture** | 4x-UltraSharp | Preserves fine lines and textures |
| **Text/Documents** | 4x-UltraSharp | Maximum clarity for readability |
| **Abstract/Graphic** | 4x-UltraSharp | Maintains crisp edges |
| **Screenshots** | 4x-UltraSharp | Preserves UI elements |

### 4. CodeFormer Fidelity Guide

**When faces are present, CodeFormer processes BEFORE upscaling.**

| Fidelity | Effect | Trade-off | Use When |
|----------|--------|-----------|----------|
| **0.5** | More cosmetic/beautiful | Less faithful to original face | Marketing/glamour shots, want attractive result |
| **0.6** | Moderate enhancement | Slight beautification | Social media, general portraits |
| **0.7** | **Balanced (RECOMMENDED)** | Good preservation + cleanup | Default for most use cases |
| **0.8** | High fidelity | Minimal alteration | Professional photography |
| **0.9** | Very faithful | Preserves all features | Archive/documentary work |
| **1.0** | Maximum fidelity | No beautification, only cleanup | Scientific/forensic applications |

**Rule of thumb:**
- Marketing/sales: **0.5-0.6** (make people look good)
- General purpose: **0.7** (balanced)
- Professional: **0.8-1.0** (preserve identity)

### 5. Pipeline A: Simple Upscale (No Faces)

**Use when:** Image has no people, or faces are too small to matter.

#### Step 5.1: Construct Workflow

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "LoadImage",
      "inputs": {
        "image": "input_image.png"
      },
      "outputs": {
        "image": {"node": 2, "input": "image"}
      }
    },
    {
      "id": 2,
      "type": "UpscaleModelLoader",
      "inputs": {
        "model_name": "4x-UltraSharp.pth"
      },
      "outputs": {
        "model": {"node": 3, "input": "upscale_model"}
      }
    },
    {
      "id": 3,
      "type": "ImageUpscaleWithModel",
      "inputs": {
        "upscale_model": {"node": 2, "output": "model"},
        "image": {"node": 1, "output": "image"}
      },
      "outputs": {
        "image": {"node": 4, "input": "images"}
      }
    },
    {
      "id": 4,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "upscaled_4x_",
        "images": {"node": 3, "output": "image"}
      }
    }
  ]
}
```

#### Step 5.2: Execute

```python
mcp__comfyui__run_workflow(
  workflow: [workflow_json],
  sync: true,  # Recommended for upscale (fast operation)
  name: "upscale_simple",
  outputMode: "full",
  imageFormat: "png",
  imageQuality: 100
)
```

### 6. Pipeline B: Face Restore + Upscale (Portraits)

**Use when:** Image has people/faces that need enhancement.

**CRITICAL:** CodeFormer runs BEFORE upscale to avoid amplifying facial artifacts.

#### Step 6.1: Construct Workflow

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "LoadImage",
      "inputs": {
        "image": "input_portrait.png"
      },
      "outputs": {
        "image": {"node": 2, "input": "image"}
      }
    },
    {
      "id": 2,
      "type": "CodeFormer",
      "inputs": {
        "fidelity": 0.7,
        "image": {"node": 1, "output": "image"}
      },
      "outputs": {
        "image": {"node": 3, "input": "image"}
      }
    },
    {
      "id": 3,
      "type": "UpscaleModelLoader",
      "inputs": {
        "model_name": "4x-UltraSharp.pth"
      },
      "outputs": {
        "model": {"node": 4, "input": "upscale_model"}
      }
    },
    {
      "id": 4,
      "type": "ImageUpscaleWithModel",
      "inputs": {
        "upscale_model": {"node": 3, "output": "model"},
        "image": {"node": 2, "output": "image"}
      },
      "outputs": {
        "image": {"node": 5, "input": "images"}
      }
    },
    {
      "id": 5,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "portrait_final_",
        "images": {"node": 4, "output": "image"}
      }
    }
  ]
}
```

#### Step 6.2: Execute

```python
mcp__comfyui__run_workflow(
  workflow: [workflow_json],
  sync: true,
  name: "upscale_with_face_restore",
  outputMode: "full",
  imageFormat: "png",
  imageQuality: 100
)
```

### 7. Pipeline C: Full Generation + Upscale (Integrated)

**Use when:** Starting from text/image generation and want to apply upscaling in same workflow.

#### Step 7.1: Integrated Workflow (txt2img → CodeFormer → Upscale)

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "Professional portrait of entrepreneur",
        "preset": "photographic",
        "aspect_ratio": "16:9",
        "temperature": 0.8,
        "seed": 42
      },
      "outputs": {
        "image": {"node": 2, "input": "image"}
      }
    },
    {
      "id": 2,
      "type": "CodeFormer",
      "inputs": {
        "fidelity": 0.7,
        "image": {"node": 1, "output": "image"}
      },
      "outputs": {
        "image": {"node": 3, "input": "image"}
      }
    },
    {
      "id": 3,
      "type": "UpscaleModelLoader",
      "inputs": {
        "model_name": "4x-UltraSharp.pth"
      },
      "outputs": {
        "model": {"node": 4, "input": "upscale_model"}
      }
    },
    {
      "id": 4,
      "type": "ImageUpscaleWithModel",
      "inputs": {
        "upscale_model": {"node": 3, "output": "model"},
        "image": {"node": 2, "output": "image"}
      },
      "outputs": {
        "image": {"node": 5, "input": "images"}
      }
    },
    {
      "id": 5,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "generated_final_",
        "images": {"node": 4, "output": "image"}
      }
    }
  ]
}
```

**Note:** This is the recommended workflow for production assets - generates at base resolution (~1024px), applies face restoration, then upscales to 4096px.

### 8. Output Resolution & File Size Management

**Resolution Math:**

| Input Resolution | Upscale Factor | Output Resolution |
|------------------|----------------|-------------------|
| 512×512 | 4x | 2048×2048 |
| 1024×1024 | 4x | 4096×4096 |
| 1920×1080 | 4x | 7680×4320 (8K) |

**IFGeminiNode generates ~1024px:**
- 1024px base × 4x upscale = **4096px output (print quality)**

**File Size Considerations:**

| Format | Resolution | Approx File Size | Use Case |
|--------|-----------|------------------|----------|
| PNG | 4096×4096 | 20-50MB | Archival, editing, maximum quality |
| JPEG (quality 90) | 4096×4096 | 2-5MB | Web, email, general distribution |
| JPEG (quality 80) | 4096×4096 | 1-3MB | Social media, fast loading |

**Optimization Strategy:**

```python
# For web use: compress after upscale
mcp__comfyui__run_workflow(
  workflow: [upscale_workflow],
  imageFormat: "jpeg",
  imageQuality: 90  # Good balance of quality/size
)
```

### 9. Retrieve & Validate Output

#### Step 9.1: Get Upscaled Image

```python
result = mcp__comfyui__run_workflow(...)

# If sync=true, get image directly from result
image_data = mcp__comfyui__get_image(
  filename: result.output.filename,
  subfolder: "",
  folder_type: "output"
)
```

#### Step 9.2: Quality Validation Checklist

**Before/After Comparison:**
- [ ] Resolution increased correctly? (check dimensions)
- [ ] No new artifacts introduced? (zoom to 100%)
- [ ] Faces look natural (if CodeFormer used)?
- [ ] Details preserved or enhanced?
- [ ] Colors unchanged?
- [ ] No blocky/pixelated areas?

**Zoom Test:** View at 100% zoom. Good upscale should look sharp, not blurry or over-sharpened.

### 10. Document Settings for Reuse

```python
mcp__comfyui__save_note(
  title: "Upscale Settings - {project_name}",
  content: f"""
    Date: {date}
    Input: {input_filename}
    Input Resolution: {input_width}x{input_height}

    Pipeline:
    {pipeline_type}

    Models Used:
    - Upscale: {upscale_model}
    - Face Restore: {face_restore} (fidelity: {fidelity})

    Output:
    - Filename: {output_filename}
    - Resolution: {output_width}x{output_height}
    - Format: {format}
    - File Size: {file_size}

    Quality Notes:
    {quality_observations}
  """
)
```

---

## UPSCALE FACTOR DECISION

**When to use 2x vs 4x:**

| Scenario | Factor | Reasoning |
|----------|--------|-----------|
| Already 2048px+ | 2x | Avoid excessive file size |
| Low resolution (<1024px) | 4x | Maximize quality gain |
| Web use only | 2x | Sufficient for screens |
| Print/large display | 4x | Ensure quality at scale |
| Quick preview | 2x | Faster processing |

**Note:** ComfyUI models are typically 4x. To achieve 2x, upscale 4x then downscale 50% (or use 2x model if available).

---

## TROUBLESHOOTING

### Issue 1: Missing Upscale Models
**Symptom:** `list_models(type: "upscale_models")` returns empty or missing expected models.

**Diagnosis:**
- Models not downloaded
- Models in wrong directory
- ComfyUI not seeing models folder

**Solution:**

**Download RealESRGAN_x4plus.pth:**
```bash
cd D:\ComfyUI\models\upscale_models
curl -L -O https://github.com/xinntao/Real-ESRGAN/releases/download/v0.1.0/RealESRGAN_x4plus.pth
```

**Download 4x-UltraSharp.pth:**
```bash
cd D:\ComfyUI\models\upscale_models
# Visit https://openmodeldb.info/ → search "4x-UltraSharp"
# Download and place in D:\ComfyUI\models\upscale_models\
```

**Verify directory:**
```bash
ls D:\ComfyUI\models\upscale_models
# Should show: RealESRGAN_x4plus.pth, 4x-UltraSharp.pth
```

**Restart ComfyUI** for models to be recognized.

### Issue 2: Face Looks Distorted After Upscale
**Symptom:** Upscaled portrait has weird facial features, artifacts around eyes/mouth.

**Diagnosis:**
- Upscaled BEFORE face restoration (wrong order)
- CodeFormer fidelity too low or too high
- Upscale model not suited for faces

**Solution:**
- **CRITICAL FIX:** Ensure CodeFormer runs BEFORE upscale in workflow
- Adjust fidelity:
  - If face looks too "AI-generated": increase fidelity to 0.8-1.0
  - If artifacts remain: decrease fidelity to 0.5-0.6
- Use 4x-UltraSharp model (better for portraits than RealESRGAN)

**Correct sequence:**
```
Generate/Load → CodeFormer (0.7) → Upscale (4x-UltraSharp) → Save
```

### Issue 3: Over-Sharpening / Halos Around Objects
**Symptom:** Upscaled image looks artificially sharp, visible halos around edges.

**Diagnosis:**
- 4x-UltraSharp too aggressive for this content type
- Image already sharp before upscaling

**Solution:**
- Switch to RealESRGAN_x4plus (gentler)
- If using img2img workflow, reduce denoise strength
- Consider 2x upscale instead of 4x

### Issue 4: Artifacts Amplified During Upscale
**Symptom:** Small defects in original image become obvious in upscaled version.

**Diagnosis:**
- Upscaling magnifies existing problems
- Original generation had artifacts

**Solution:**
- **Prevention:** Generate at higher quality first
  - Use better preset (photographic instead of anime for realism)
  - Lower temperature for more coherence
- **Fix:** Apply CodeFormer even if no faces (works on general artifacts)
- **Alternative:** Regenerate base image with improved prompt

### Issue 5: File Size Too Large
**Symptom:** 4K PNG is 40-50MB, too large for web/email.

**Diagnosis:**
- PNG format is lossless = large files
- 4K resolution is overkill for web

**Solution:**
- **Option A:** Change output format to JPEG (90 quality):
  ```python
  imageFormat: "jpeg",
  imageQuality: 90  # Good balance
  ```
- **Option B:** Downscale after upscaling (4K → 2K for web)
- **Option C:** Use 2x upscale instead of 4x

**Compression comparison:**
- 4K PNG: 40MB
- 4K JPEG (q=90): 3MB
- 2K JPEG (q=90): 800KB

### Issue 6: Upscale Takes Too Long
**Symptom:** Workflow times out or takes 5+ minutes.

**Diagnosis:**
- Processing 4x upscale on large image
- ComfyUI server overloaded
- GPU memory exhausted

**Solution:**
- Check server status: `mcp__comfyui__get_status()`
- Restart ComfyUI if memory issues
- For very large images (>4K input), consider 2x upscale
- Close other GPU-intensive applications

**Expected timing:**
- 1024px → 4096px: 30-60 seconds
- 2048px → 8192px: 2-3 minutes

### Issue 7: Colors Look Different After Upscale
**Symptom:** Upscaled image has slightly different color tone than original.

**Diagnosis:**
- Model applies color correction
- Format conversion (PNG→JPEG) changed colors

**Solution:**
- Keep output format same as input (PNG→PNG)
- If JPEG needed, use high quality (95+) to minimize color shift
- Check color profile consistency

### Issue 8: CodeFormer Not Working
**Symptom:** Faces still look wrong after CodeFormer node.

**Diagnosis:**
- codeformer.pth model not installed
- Face too small in image (CodeFormer needs minimum size)
- Fidelity setting inappropriate

**Solution:**
- Verify model: `mcp__comfyui__list_models(type: "codeformer")`
- Download if missing:
  ```bash
  cd D:\ComfyUI\models\codeformer
  # Download codeformer.pth from official repo
  ```
- Ensure faces are >128px before restoration
- Try different fidelity values (0.5, 0.7, 0.9)

### Issue 9: Upscaled Image Still Looks Blurry
**Symptom:** After 4x upscale, image doesn't look much sharper.

**Diagnosis:**
- Original image was already low quality
- Wrong model selected
- Image has motion blur/focus issues

**Solution:**
- Check original resolution - if <512px, upscaling has limits
- Switch to 4x-UltraSharp for maximum sharpness
- If original has blur, upscaling can't fix it (consider regenerating)
- Ensure model actually loaded: check workflow node connections

### Issue 10: Can't Find Upscaled Image
**Symptom:** Workflow completes but can't locate output file.

**Diagnosis:**
- Filename mismatch (ComfyUI adds counter)
- Wrong folder (temp vs output)

**Solution:**
- Check workflow result for actual filename:
  ```python
  result = mcp__comfyui__run_workflow(...)
  print(result.output.filename)  # Actual saved filename
  ```
- List output directory:
  ```bash
  ls D:\ComfyUI\output | grep upscaled
  ```
- Use `get_task_result` to retrieve correct path

---

## CROSS-REFERENCES

**Knowledge Base:**
- KB-capabilities.md → IFGeminiNode output resolution limitations
- KB-pipeline-reference.md → Stage 2 (Upscale) + Stage 3 (Face Restore) detailed specs
- KB-system-inventory.md → Section 3 (Models: RealESRGAN, UltraSharp, CodeFormer)
- KB-workflow-patterns.md → Complete upscale workflow examples

**Related Tasks:**
- `generate-image.md` → Base generation before upscaling
- `generate-from-reference.md` → img2img workflows that may need upscaling
- `create-brand-asset.md` → Upscaling brand assets for print
- `batch-variations.md` → Upscaling selected variations post-selection

**MCP Documentation:**
- `run_workflow` → Execute upscale pipeline
- `list_models` → Verify model availability
- `get_image` → Retrieve upscaled output

---

## BEST PRACTICES

### 1. Always Preview Before Full Pipeline
Generate base image → review → if approved → apply upscale. Don't upscale everything.

### 2. Save Original Before Upscaling
Keep original generation in case upscale introduces issues.

### 3. Use Descriptive Filenames
`portrait_entrepreneur_codeformer_07_4xultrasharp.png` tells the story.

### 4. Document Successful Settings
Save workflows that work well for reuse (via `save_template`).

### 5. Batch Upscaling Strategy
If upscaling many images:
1. Generate all base images first
2. Review and select winners
3. Apply expensive upscale only to selected images

### 6. Web Optimization
For web use: upscale to 4K, then export JPEG at 90 quality for distribution.

---

## PRODUCTION WORKFLOW RECOMMENDATION

**For professional brand assets:**

```
1. Generate with IFGeminiNode
   ↓
2. Review output at base resolution
   ↓
3. If approved AND has faces:
   → Apply CodeFormer (fidelity 0.7)
   ↓
4. Upscale with 4x-UltraSharp
   ↓
5. Save as PNG (archive) + JPEG q=90 (distribution)
   ↓
6. Document settings in MCP notes
   ↓
7. Save winning workflow as template
```

**Time per image:** 2-3 minutes
**Cost per image:** ~R$0.01 (Gemini) + R$0 (upscale is local)

---

## SUCCESS CRITERIA

Upscale task is complete when:
- [ ] Input image analyzed (faces detected/not detected)
- [ ] Correct pipeline selected (with or without CodeFormer)
- [ ] Appropriate model chosen (RealESRGAN vs UltraSharp)
- [ ] Workflow executed without errors
- [ ] Output resolution verified (4x increase confirmed)
- [ ] Quality validated (no artifacts, faces natural if present)
- [ ] File size appropriate for use case
- [ ] Before/after comparison documented
- [ ] Settings logged in MCP notes for reuse

---

## QUICK REFERENCE: PIPELINE SELECTION

```
Has faces? → YES → CodeFormer (0.7) → Upscale (4x-UltraSharp)
Has faces? → NO → Direct Upscale (model by subject)

Subject type:
├─ Portrait → 4x-UltraSharp
├─ Landscape → RealESRGAN_x4plus
├─ Product → RealESRGAN_x4plus
├─ Architecture → 4x-UltraSharp
├─ Text/Screenshot → 4x-UltraSharp
└─ Abstract → 4x-UltraSharp
```

---

*ComfyUI Ops Task - Upscale Image*
*Version: 1.0 | Updated: 2026-02-06*
*KB: capabilities, pipeline-reference, system-inventory, workflow-patterns*
