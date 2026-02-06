# Inpaint Edit - Edição Prompt-Based de Regiões

Edita regiões específicas de imagens usando chat-based iterative editing do Gemini.

---

## IMPORTANT: LIMITATIONS

**Gemini API does NOT support:**
- ❌ Mask-based inpainting (no MASK input)
- ❌ Pixel-precise region selection
- ❌ ControlNet conditioning
- ❌ Negative prompts

**Gemini API DOES support:**
- ✅ Chat-based iterative editing (15-20 turns)
- ✅ Prompt-driven regional changes
- ✅ Natural language region description
- ✅ Multi-image reference for style/context

**Future:** When Stable Diffusion checkpoint is installed, mask-based inpainting becomes available.

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

---

## WORKFLOW

### STEP 1: Understand User Intent

Elicit from user:
1. **Base image:** Path or URL
2. **Region to edit:** "background", "shirt color", "remove person on left", etc.
3. **Desired result:** What should the region become?

### STEP 2: Setup Strategy

**Choose approach based on edit type:**

| Edit Type | Approach | Example Prompt |
|-----------|----------|----------------|
| **Background change** | Describe new background | "Replace the concrete background with lush green grass and wildflowers" |
| **Color change** | Specify new color + preserve rest | "Change the shirt to deep blue, keep everything else identical" |
| **Object removal** | Describe what fills the space | "Remove the person on the left, extend the wall naturally" |
| **Object addition** | Describe placement + context | "Add a coffee cup on the table to the right of the laptop" |
| **Style change** | Multi-ref with style image | Upload style ref + "Apply this aesthetic while keeping subject" |

### STEP 3: First Generation (Baseline)

Build workflow with IFGeminiNode:

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "LoadImage",
      "inputs": {
        "image": "path/to/image.jpg"
      }
    },
    {
      "id": 2,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "REGION-AWARE EDIT PROMPT HERE",
        "images": ["@1"],
        "operation_mode": "generate_images",
        "model_name": "gemini-2.5-flash",
        "temperature": 1.0,
        "aspect_ratio": "1:1",
        "batch_count": 4,
        "chat_mode": true,
        "clear_history": false,
        "max_images": 6,
        "seed": 0,
        "sequential_generation": false
      }
    },
    {
      "id": 3,
      "type": "PreviewImage",
      "inputs": {
        "images": ["@2"]
      }
    }
  ]
}
```

**Key parameters:**
- `chat_mode: true` → Enables iterative editing
- `batch_count: 4` → Generate 4 options
- `max_images: 6` → Accept up to 6 reference images
- `temperature: 1.0` → Google recommendation for images

### STEP 4: Craft Region-Aware Prompt

**Template:**
```
A photorealistic [SHOT TYPE] showing [ORIGINAL SUBJECT], but with
[REGION] now [DESIRED CHANGE]. The [REGION] should [INTEGRATION INSTRUCTIONS].
Keep the lighting and perspective identical to the original.
[ASPECT RATIO] format.
```

**Examples:**

**Background replacement:**
```
A photorealistic portrait of a woman in a red dress, but with the
concrete wall background now replaced by lush green grass with wildflowers.
The new background should have soft natural bokeh consistent with
the original 85mm f/1.8 shallow depth of field. Golden hour lighting
preserved. 3:4 portrait format.
```

**Color change:**
```
A photorealistic product photo of wireless earbuds, but with the
charging case color now changed from white to matte black. Keep the
studio lighting, reflections, and composition identical. The texture
should match the premium plastic finish of the original. 1:1 square format.
```

**Object removal:**
```
A photorealistic street scene, but with the car on the left side now
removed. The empty space should show natural continuation of the
cobblestone pavement with subtle wear patterns. Preserve the overcast
lighting and architectural details. 16:9 landscape format.
```

### STEP 5: Validate and Execute

```
mcp__comfyui__validate_workflow (workflow: JSON)
```

If valid:
```
mcp__comfyui__run_workflow (
  workflow: JSON,
  name: "edit_{descriptive_name}",
  sync: true,
  imageFormat: "jpeg",
  imageQuality: 95
)
```

### STEP 6: Review Results

User selects best of 4 options. If close but not perfect → STEP 7 (iterate).

### STEP 7: Iterative Refinement (Chat Mode)

**RULE:** ONE change per turn, maximum 15-20 turns.

**Refinement template:**
```
"Keep everything identical except [SINGLE SPECIFIC CHANGE]"
```

**Examples:**

```
Turn 2: "Keep everything identical except make the grass slightly darker green"
Turn 3: "Keep everything identical except add subtle shadows under wildflowers"
Turn 4: "Keep everything identical except soften the transition at the horizon line"
Turn 5: "Keep everything identical except increase background blur slightly"
```

**Each iteration:**
1. Keep `chat_mode: true`
2. Keep `clear_history: false`
3. Set `batch_count: 1` (faster for refinement)
4. User approves or requests another change

### STEP 8: Multi-Reference Style Transfer (Advanced)

For complex edits requiring style consistency:

**Workflow:**
1. Load base image (IMAGE_1)
2. Load style reference (IMAGE_2)
3. Optionally load background reference (IMAGE_3)

**Prompt:**
```
Apply the artistic style and color palette from the second reference image
to the subject from the first image, while placing them in the environment
shown in the third image. Maintain photorealistic quality and natural
lighting integration. [ASPECT RATIO] format.
```

**Parameters:**
- `max_images: 16` → Support multi-ref
- `batch_count: 4` → Explore variations

---

## EXAMPLE WORKFLOWS

### Example 1: Background Replacement

**User request:** "Change the office background to a beach"

**Prompt:**
```
A photorealistic headshot of a business professional, but with the
office background now replaced by a tropical beach at golden hour.
The beach background should have soft bokeh with visible palm trees
and ocean in the distance. Maintain the same 85mm f/1.8 shallow depth
of field and warm lighting on the subject's face. 3:4 portrait format.
```

**Iterations:**
- Turn 2: "Keep everything identical except make the ocean more teal-blue"
- Turn 3: "Keep everything identical except add subtle rim light on hair from beach sunset"

### Example 2: Product Color Variant

**User request:** "Show this shoe in red instead of black"

**Prompt:**
```
A photorealistic product photo of a running shoe, but with the shoe
color now changed from black to vibrant red (Pantone 186 C equivalent).
Keep the white sole, studio lighting with three-point setup, and all
reflections on the display surface identical. Preserve the texture of
the mesh fabric and synthetic overlays. 1:1 square format.
```

**Iterations:**
- Turn 2: "Keep everything identical except make the red slightly deeper"
- Turn 3: "Keep everything identical except preserve the original shadow under the shoe"

---

## MCP EXECUTION PATTERN

### Full Editing Session

```javascript
// 1. Validate workflow
const validation = await mcp__comfyui__validate_workflow({
  workflow: workflowJSON
});

if (!validation.valid) {
  console.error("Validation errors:", validation.errors);
  return;
}

// 2. Initial generation (batch_count: 4)
const task1 = await mcp__comfyui__run_workflow({
  workflow: workflowJSON,
  name: "edit_background_beach_v1",
  sync: true,
  imageFormat: "jpeg",
  imageQuality: 95
});

// 3. User selects best result

// 4. Iterative refinement (batch_count: 1)
const refinementWorkflow = {
  ...workflowJSON,
  nodes: workflowJSON.nodes.map(node => {
    if (node.type === "IFGeminiNode") {
      return {
        ...node,
        inputs: {
          ...node.inputs,
          prompt: "Keep everything identical except make ocean more teal-blue",
          batch_count: 1,
          chat_mode: true,
          clear_history: false
        }
      };
    }
    return node;
  })
};

const task2 = await mcp__comfyui__run_workflow({
  workflow: refinementWorkflow,
  name: "edit_background_beach_v2",
  sync: true
});

// Repeat turns 3-15 as needed
```

---

## TROUBLESHOOTING

| Problem | Cause | Solution |
|---------|-------|----------|
| **Edit affects wrong region** | Prompt too vague | Be more specific: "the wall behind the person" not "the background" |
| **Result too different** | Prompt too transformative | Use "Keep everything identical except..." |
| **No visible change** | Region description unclear | Add spatial anchors: "the shirt on the left person" |
| **Lighting inconsistent** | Forgot to mention preservation | Add: "Maintain the original lighting and shadows" |
| **Iterative drift** | Too many turns | Limit to 15 turns, then restart with best result |
| **Grey placeholder output** | Generation failed | Check API key, quota, or safety filters |
| **Blending artifacts** | Natural limitation | Use semantic negative: "Ensure seamless integration with natural transition" |

---

## ADVANCED: MULTI-IMAGE COMPOSITION

For complex edits requiring multiple references:

**Workflow inputs:**
1. Base subject (IMAGE_1)
2. New outfit reference (IMAGE_2)
3. Background scene (IMAGE_3)
4. Lighting reference (IMAGE_4)

**Prompt:**
```
Create a photorealistic composite: place the person from IMAGE_1 wearing
the outfit from IMAGE_2, in the environment from IMAGE_3, with lighting
matching IMAGE_4. Ensure natural integration with proper perspective,
shadows, and color harmony. 16:9 format.
```

**Note:** This is CONTEXTUAL guidance, not structural control (no ControlNet).

---

## SEMANTIC NEGATIVE PROMPTING

Since Gemini doesn't support negative prompts, reframe negatives as positives:

| Avoid | Instead Write |
|-------|---------------|
| "no blurry edges" | "tack-sharp edges with clean transitions" |
| "no unnatural colors" | "natural color palette consistent with original" |
| "no artifacts" | "seamless photorealistic integration" |
| "no visible seams" | "perfectly blended with natural transition" |

---

## COMPARISON: GEMINI vs STABLE DIFFUSION INPAINTING

| Feature | Gemini (Now) | SD Inpainting (Future) |
|---------|-------------|------------------------|
| **Mask input** | ❌ No | ✅ Yes (pixel-precise) |
| **Iterative editing** | ✅ Yes (15-20 turns) | ❌ Limited |
| **Natural language** | ✅ Excellent | ⚠️ Hit or miss |
| **Multi-ref** | ✅ Up to 16 images | ⚠️ Limited |
| **Speed** | ⚠️ ~30s per gen | ✅ ~5-10s |
| **Control precision** | ⚠️ Contextual | ✅ Pixel-level |
| **Best for** | Natural edits, style changes | Precise object removal, exact masks |

---

## PARAMETERS REFERENCE

| Parameter | Value | Why |
|-----------|-------|-----|
| `operation_mode` | `generate_images` | Always for editing |
| `model_name` | `gemini-2.5-flash` | Fast iteration |
| `temperature` | `1.0` | Google recommendation |
| `chat_mode` | `true` | Enable iterative editing |
| `clear_history` | `false` | Maintain context |
| `batch_count` | `4` initial, `1` refinement | Explore then refine |
| `max_images` | `6` standard, `16` advanced | Support multi-ref |
| `aspect_ratio` | Match original | Preserve composition |
| `seed` | `0` (random) | Explore variations |
| `sequential_generation` | `false` | Batch independent options |

---

## SUCCESS METRICS

| Metric | Target | How to Measure |
|--------|--------|----------------|
| **Region accuracy** | 90%+ | Region description matches result |
| **Preservation quality** | 95%+ | Untouched areas identical to original |
| **Iteration efficiency** | < 5 turns | Achieve desired result |
| **Blending quality** | Seamless | No visible artifacts at transitions |

---

## WHEN TO USE THIS vs OTHER TASKS

| Goal | Use This | Use Instead |
|------|----------|-------------|
| Change background | ✅ Yes | — |
| Change colors | ✅ Yes | — |
| Remove objects | ✅ Yes (describe fill) | — |
| Add objects | ✅ Yes | `create-brand-asset` |
| Style transfer | ✅ Yes | `generate-from-reference` |
| Exact mask removal | ❌ No | Wait for SD checkpoint |
| Face swap | ⚠️ Limited | Use Task Preset #4 |

---

*ComfyUI Ops Squad*
*Task: inpaint-edit.md*
*Updated: 2026-02-06*
*Status: Production Ready*
