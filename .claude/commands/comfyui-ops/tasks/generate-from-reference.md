# Generate from Reference - img2img Inteligente

Generate images using reference images as visual context via Gemini API.

---

## PRE-FLIGHT

### 1. Load Knowledge Base

Ensure you have read these files before proceeding:

```
knowledge/KB-capabilities.md       # Gemini API capabilities and limitations
knowledge/KB-prompting-heuristics.md  # Prompt engineering for image generation
knowledge/KB-workflow-patterns.md     # Common workflow patterns
knowledge/KB-pipeline-reference.md    # Complete node/pipeline reference
```

**CRITICAL:** Gemini API does NOT support:
- Negative prompts
- Denoise strength parameter (it's a txt2img model)
- ControlNet
- Traditional img2img with denoising

**Gemini API DOES support:**
- Image input as visual context (up to 16 images)
- Chat mode with keep_alive for iterative refinement
- Multi-reference composition

### 2. Verify ComfyUI Status

```
mcp__comfyui__get_status
```

Expected response:
- ComfyUI running at http://127.0.0.1:8188
- Status: "ready"
- Queue length: 0

If not ready, abort and notify user.

---

## WORKFLOW

### 1. Briefing

Elicit from user:
- **Reference image(s):** Path(s) or URL(s) (max 16)
- **Transformation objective:** What to change/generate
- **Reference protocol:** Subject, style, or background?
- **Output dimensions:** Default 1024x1024 or custom

### 2. Understand Reference Protocol

Gemini interprets multiple images in a specific way:

| Image Position | Gemini Interpretation |
|----------------|----------------------|
| **First image** | Primary subject/composition |
| **Last image** | Background/environment context |
| **Middle images** | Style, texture, mood references |

**Decision Matrix:**

| User Intent | Image Order | Prompt Strategy |
|-------------|-------------|-----------------|
| Keep composition, change style | Subject FIRST, style images MIDDLE/LAST | "In the style of [describe style]" |
| Keep subject, new background | Subject FIRST, background LAST | "Same subject, placed in [new background]" |
| Merge multiple subjects | All subjects as separate images | "Combining elements from all images" |
| Extract and transform element | Single image | "Extract [element] and transform it into [description]" |

### 3. Craft Transformation Prompt

**Key principle:** Describe the RESULT you want, not the transformation process.

Prompt structure:
```
[COMPOSITION DESCRIPTION], [STYLE/AESTHETIC], [TECHNICAL QUALITY], [MOOD/LIGHTING]
```

**Examples:**

| Intent | Prompt |
|--------|--------|
| Style transfer | "A portrait in cyberpunk aesthetic, neon lighting, futuristic city background, 8k photorealistic" |
| Background change | "Same person, standing in a lush tropical forest, natural lighting, golden hour" |
| Object transformation | "Transform the car into a futuristic hovering vehicle, sleek metallic surfaces, glowing accents" |
| Artistic interpretation | "Watercolor painting style, soft brush strokes, pastel color palette, impressionist aesthetic" |

**Prompt specificity controls transformation intensity:**
- **High specificity:** More transformation (detailed description)
- **Low specificity:** Closer to original (vague description like "same style")

### 4. Select Workflow Template

Two strategies available:

#### Strategy A: Basic img2img (Standard)

```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": {
      "image": "reference.png"
    }
  },
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "transformation prompt here",
      "image": ["1", 0],
      "model": "gemini-2.0-flash-exp",
      "aspect_ratio": "1:1",
      "keep_alive": false,
      "seed": 42
    }
  },
  "3": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["2", 0],
      "filename_prefix": "img2img_result"
    }
  }
}
```

#### Strategy B: img2img with Upscale (High Quality)

```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": {
      "image": "reference.png"
    }
  },
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "transformation prompt here",
      "image": ["1", 0],
      "model": "gemini-2.0-flash-exp",
      "aspect_ratio": "1:1",
      "keep_alive": false,
      "seed": 42
    }
  },
  "3": {
    "class_type": "UpscaleModelLoader",
    "inputs": {
      "model_name": "RealESRGAN_x4plus.pth"
    }
  },
  "4": {
    "class_type": "ImageUpscaleWithModel",
    "inputs": {
      "upscale_model": ["3", 0],
      "image": ["2", 0]
    }
  },
  "5": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["4", 0],
      "filename_prefix": "img2img_upscaled"
    }
  }
}
```

**When to use upscale:**
- Output will be printed or displayed large
- User requests "high quality" or "4K"
- Text or fine details are critical

### 5. Handle Multiple Reference Images

For 2+ images, use LoadImage nodes for each:

```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": { "image": "subject.png" }
  },
  "2": {
    "class_type": "LoadImage",
    "inputs": { "image": "style_ref.png" }
  },
  "3": {
    "class_type": "LoadImage",
    "inputs": { "image": "background.png" }
  },
  "4": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Combine the subject from first image with style of second image and background of third image",
      "image": ["1", 0],
      "image_2": ["2", 0],
      "image_3": ["3", 0],
      "model": "gemini-2.0-flash-exp",
      "aspect_ratio": "1:1"
    }
  }
}
```

### 6. Validate Workflow

```
mcp__comfyui__validate_workflow (workflow: <JSON_WORKFLOW>)
```

If validation fails, fix errors before proceeding.

### 7. Execute Generation

```
mcp__comfyui__run_workflow (
  workflow: <JSON_WORKFLOW>,
  sync: true,
  name: "img2img_descriptive_name",
  outputMode: "all",
  imageFormat: "jpeg",
  imageQuality: 95
)
```

### 8. Iterative Refinement (Chat Mode)

For iterative improvements, enable chat mode:

```json
{
  "class_type": "IFGeminiNode",
  "inputs": {
    "prompt": "Make the lighting warmer and more cinematic",
    "image": ["previous_output", 0],
    "keep_alive": true,
    "seed": 42
  }
}
```

**Chat mode benefits:**
- Preserves context from previous generation
- Faster iterations (reuses conversation state)
- More coherent refinements

**Chat mode workflow:**
1. Initial generation with keep_alive: true
2. User provides feedback
3. New prompt references previous output
4. Repeat until satisfied

### 9. Name and Store Generation

```
mcp__comfyui__name_generation (
  taskId: "task_id_from_run",
  name: "project_img2img_final"
)
```

Retrieve later:
```
mcp__comfyui__get_generation_by_name (name: "project_img2img_final")
```

---

## DECISION MATRIX: img2img vs txt2img

| Scenario | Use img2img | Use txt2img |
|----------|-------------|-------------|
| Have reference image(s) to transform | ✅ | ❌ |
| Want specific composition preserved | ✅ | ❌ |
| Style transfer | ✅ | ❌ |
| Background replacement | ✅ | ❌ |
| Merging multiple references | ✅ | ❌ |
| Generating from scratch | ❌ | ✅ |
| No visual reference available | ❌ | ✅ |

---

## PARAMETERS

### IFGeminiNode (img2img mode)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| prompt | string | required | Transformation description |
| image | image | required | Primary reference image |
| image_2 to image_16 | image | optional | Additional reference images |
| model | string | "gemini-2.0-flash-exp" | Gemini model (flash recommended) |
| aspect_ratio | string | "1:1" | Output aspect ratio |
| keep_alive | boolean | false | Enable chat mode for iterations |
| seed | integer | random | Reproducibility seed |

**Available aspect ratios:**
- Square: "1:1" (1024×1024)
- Landscape: "16:9" (1344×768), "3:2" (1216×832), "4:3" (1152×896)
- Portrait: "9:16" (768×1344), "2:3" (832×1216), "3:4" (896×1152)

### UpscaleModelLoader

| Model | Scale | Best For |
|-------|-------|----------|
| RealESRGAN_x4plus.pth | 4x | Photorealistic images |
| 4x-UltraSharp.pth | 4x | Sharp details, illustrations |

---

## TROUBLESHOOTING

### Issue 1: Original image completely ignored

**Symptom:** Output looks nothing like reference image.

**Causes:**
1. Prompt too strong/specific (overpowering visual context)
2. Reference image too different from prompt description

**Solutions:**
- Reduce prompt specificity: "In the same style" vs "Cyberpunk neon aesthetic..."
- Add explicit instruction: "Preserve the composition and subject from the image"
- Try chat mode with "Make subtle changes to [specific aspect]"

### Issue 2: Output too similar to original

**Symptom:** Transformation barely visible.

**Causes:**
1. Prompt too vague
2. Not enough transformation instructions

**Solutions:**
- Increase prompt specificity and detail
- Use stronger transformation language: "Transform into" vs "Similar to"
- Add technical quality descriptors: "8k, photorealistic, studio lighting"

### Issue 3: Multiple images not being used

**Symptom:** Only first image appears to influence output.

**Causes:**
1. Missing image_2, image_3 connections in workflow
2. Prompt doesn't reference multiple images

**Solutions:**
- Verify all LoadImage nodes connected to IFGeminiNode
- Prompt must explicitly reference: "Combine elements from all images"
- Check image order matches intent (subject first, style middle, background last)

### Issue 4: Wrong style applied

**Symptom:** Style doesn't match intended reference.

**Causes:**
1. Style reference image in wrong position
2. Prompt conflicts with visual style

**Solutions:**
- Place style references in MIDDLE positions (images 2-N)
- Prompt should describe style abstractly: "In the style shown in reference images"
- Use chat mode to refine: "Match the artistic style more closely"

### Issue 5: Background not changing

**Symptom:** Background remains from original image.

**Causes:**
1. Background reference not in LAST position
2. Prompt doesn't explicitly change background

**Solutions:**
- Move background image to LAST position
- Prompt: "Same subject, placed in [background description]"
- Consider using img2img twice: isolate subject → place in new background

### Issue 6: Upscaled image looks worse

**Symptom:** Upscaled version has artifacts or noise.

**Causes:**
1. Wrong upscale model for image type
2. Base image quality too low

**Solutions:**
- Photorealistic images: Use RealESRGAN_x4plus.pth
- Illustrations/sharp details: Use 4x-UltraSharp.pth
- Generate higher quality base before upscaling (use gemini-2.0-flash-exp)

### Issue 7: Chat mode not preserving context

**Symptom:** Second iteration ignores first generation.

**Causes:**
1. keep_alive: false in first generation
2. Different seed between iterations
3. Not using previous output as image input

**Solutions:**
- First generation MUST have keep_alive: true
- Use same seed for all iterations
- Feed previous output as image input to next IFGeminiNode
- Keep prompt conversational: "Now make it warmer" vs complete redescription

---

## EXAMPLES

### Example 1: Style Transfer (Keep Composition)

**Scenario:** User has portrait photo, wants anime style.

**Workflow:**
1. LoadImage: portrait.jpg
2. IFGeminiNode:
   - prompt: "Anime illustration style, cel shading, vibrant colors, same person and pose"
   - image: [1, 0]
   - aspect_ratio: "2:3" (portrait)

### Example 2: Background Replacement

**Scenario:** Product photo on white background → tropical beach background.

**Workflow:**
1. LoadImage: product.png (subject)
2. LoadImage: beach.jpg (background reference)
3. IFGeminiNode:
   - prompt: "Same product, placed on a tropical beach with palm trees and ocean, natural lighting"
   - image: [1, 0] (product first)
   - image_2: [2, 0] (beach last)

### Example 3: Iterative Refinement

**Scenario:** Generate → refine lighting → adjust color → final.

**Workflow:**
```json
// Generation 1
{
  "1": { "class_type": "LoadImage", "inputs": { "image": "base.png" } },
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Professional studio lighting, clean background",
      "image": ["1", 0],
      "keep_alive": true,
      "seed": 42
    }
  }
}

// Generation 2 (refine lighting)
{
  "3": { "class_type": "LoadImage", "inputs": { "image": "output_gen1.jpg" } },
  "4": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Make the lighting warmer and more cinematic",
      "image": ["3", 0],
      "keep_alive": true,
      "seed": 42
    }
  }
}
```

---

## CROSS-REFERENCES

**Related tasks:**
- `generate-image.md` - Basic txt2img generation
- `svg-to-image.md` - Layout-based generation with diffusion
- `create-banner.md` - Uses img2img for style integration
- `batch-variations.md` - Generate multiple img2img variations

**Knowledge bases:**
- `KB-capabilities.md` - Gemini API full capabilities reference
- `KB-prompting-heuristics.md` - Advanced prompt engineering techniques
- `KB-workflow-patterns.md` - Common workflow patterns and templates
