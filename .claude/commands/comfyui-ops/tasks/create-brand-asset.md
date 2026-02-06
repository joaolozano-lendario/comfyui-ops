# Create Brand Asset - Asset Visual com Identidade de Marca

Gera assets visuais profissionais com identidade de marca integrada usando brand-config.yaml.

---

## PRE-FLIGHT CHECKS

### 1. Knowledge Base Status
Confirm access to:
- `knowledge/KB-capabilities.md` - IFGeminiNode features & limitations
- `knowledge/KB-prompting-heuristics.md` - Narrative prompt techniques
- `knowledge/KB-workflow-patterns.md` - JSON workflow structures
- `knowledge/KB-pipeline-reference.md` - Full generation pipeline (Stages 1-4)
- `knowledge/KB-system-inventory.md` - Available models & presets

### 2. MCP Connection
```bash
# Verify ComfyUI is running
curl -s http://127.0.0.1:8188/system_stats
```

Test MCP availability:
```
mcp__comfyui__get_status
mcp__comfyui__get_capabilities
```

### 3. Brand System Load
Read brand configuration:
```
brand-config.yaml (if it exists at repo root)
```

If `brand-config.yaml` exists, load brand colors from it.
If not, ask the user for their brand colors or proceed with generic defaults.

---

## WORKFLOW

### 1. Parse Briefing (2-5 minutes)

Gather requirements:

| Question | Purpose |
|----------|---------|
| What asset type? | Determines dimensions & composition |
| For which product tier? | Selects product color palette |
| What message/emotion? | Guides narrative prompt direction |
| Where will it be used? | Defines aspect ratio & resolution |
| Any existing reference? | Influences style & layout |

### 2. Asset Type Decision Matrix

| Asset Type | Dimensions | Aspect Ratio | Primary Use |
|------------|-----------|--------------|-------------|
| **Hero Image** | 1920×1080 | 16:9 | Website headers, YouTube thumbnails |
| **Social Media** | 1080×1080 | 1:1 | Instagram, Facebook posts |
| **Instagram Story** | 1080×1920 | 9:16 | Vertical stories, Reels |
| **Email Header** | 600×200 | 3:1 | Email campaigns |
| **Course Thumbnail** | 1280×720 | 16:9 | Course platforms, dashboards |
| **Presentation Slide** | 1920×1080 | 16:9 | Slides, webinar backgrounds |
| **Banner/Ad** | 728×90 or 300×250 | Custom | Display advertising |

### 3. Brand Color Integration Strategy

#### Option A: Subtle Brand Presence (Recommended for Photos)
Embed brand colors in **lighting & atmosphere**:

```text
Example narrative prompt:
"Professional entrepreneur in modern office, warm golden hour sunlight (#FFD44A tones) streaming through windows, deep charcoal walls (#0A0A0A), subtle cyan accent light (#22d3ee) highlighting laptop screen, sophisticated and luxurious atmosphere"
```

#### Option B: Dominant Brand Colors (Recommended for Abstract/Graphic)
Make brand colors the **primary visual elements**:

```text
Example narrative prompt:
"Abstract geometric composition, bold golden yellow (#FFD44A) hexagons floating over deep black void (#0A0A0A), electric cyan (#22d3ee) light trails connecting shapes, modern tech aesthetic, high contrast"
```

#### Option C: SVG Base Layer + img2img (Recommended for Logo/Text Assets)
1. Create SVG with brand elements (logo, text, shapes)
2. Render SVG to PNG using MCP
3. Use img2img with low denoise to enhance while preserving structure

### 4. SVG Strategy for Text/Logo Assets

When asset needs **text or precise logo placement**:

**Step 4.1: Design SVG Structure**
```xml
<svg width="1920" height="1080" xmlns="http://www.w3.org/2000/svg">
  <rect width="1920" height="1080" fill="#0A0A0A"/>
  <text x="960" y="540" font-family="Space Grotesk" font-size="96"
        fill="#FFD44A" text-anchor="middle">
    Your Brand Name
  </text>
  <circle cx="960" cy="300" r="120" fill="none"
          stroke="#22d3ee" stroke-width="4"/>
</svg>
```

**Step 4.2: Download Required Fonts (if needed)**
```
mcp__comfyui__download_font(
  font_family: "Space Grotesk",
  weights: ["400", "700"]
)
```

**Step 4.3: Render SVG to PNG**
```
mcp__comfyui__render_svg(
  svg: [svg_content_string],
  width: 1920,
  height: 1080,
  background: "#0A0A0A",
  fonts: ["Space Grotesk"],
  filename: "brand_base_layer.png"
)
```

**Step 4.4: Enhance with img2img**
Build workflow with:
- LoadImage (SVG render output)
- IFGeminiNode (denoise: 0.4-0.6, prompt adds atmosphere/effects)
- SaveImage

**Prompt for enhancement:**
```text
"Professional tech branding with golden accents and cyan glow effects,
sophisticated lighting, maintain text legibility, add subtle depth and texture"
```

### 5. Generation Strategy: txt2img vs img2img

| Strategy | When to Use | Denoise | Brand Control |
|----------|-------------|---------|---------------|
| **txt2img pure** | Illustrations, photos, abstract | N/A | Prompt-embedded colors |
| **img2img low denoise** | Enhance SVG/reference | 0.3-0.5 | High (preserves structure) |
| **img2img high denoise** | Reinterpret reference | 0.6-0.8 | Medium (uses as inspiration) |

### 6. Construct JSON Workflow

**Standard Brand Asset Workflow (txt2img):**
```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "[narrative_prompt_with_brand_colors]",
        "negative_prompt": "",
        "preset": "photographic",
        "aspect_ratio": "16:9",
        "batch_count": 1,
        "temperature": 0.8,
        "seed": -1
      },
      "outputs": {
        "image": {"node": 2, "input": "images"}
      }
    },
    {
      "id": 2,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "brand_asset_",
        "images": {"node": 1, "output": "image"}
      }
    }
  ]
}
```

**With Upscale & Face Restore (if asset has people):**
```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "[narrative_prompt]",
        "aspect_ratio": "16:9"
      },
      "outputs": {"image": {"node": 2, "input": "image"}}
    },
    {
      "id": 2,
      "type": "CodeFormer",
      "inputs": {
        "fidelity": 0.7,
        "image": {"node": 1, "output": "image"}
      },
      "outputs": {"image": {"node": 3, "input": "image"}}
    },
    {
      "id": 3,
      "type": "UpscaleModelLoader",
      "inputs": {"model_name": "4x-UltraSharp.pth"},
      "outputs": {"model": {"node": 4, "input": "upscale_model"}}
    },
    {
      "id": 4,
      "type": "ImageUpscaleWithModel",
      "inputs": {
        "upscale_model": {"node": 3, "output": "model"},
        "image": {"node": 2, "output": "image"}
      },
      "outputs": {"image": {"node": 5, "input": "images"}}
    },
    {
      "id": 5,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "brand_hero_final_",
        "images": {"node": 4, "output": "image"}
      }
    }
  ]
}
```

### 7. Execute Generation

```
mcp__comfyui__run_workflow(
  workflow: [json_workflow],
  sync: true,
  name: "brand_asset_[product]_hero_v1",
  outputMode: "full",
  imageFormat: "png",
  imageQuality: 100
)
```

**Parameters Explanation:**
- `sync: true` - Wait for completion (recommended for single assets)
- `name` - Use descriptive name: `[product]_[type]_[version]`
- `outputMode: "full"` - Get complete output data
- `imageFormat: "png"` - Use PNG for quality, JPEG for web optimization
- `imageQuality: 100` - Max quality for brand assets

### 8. Retrieve & Review

```
mcp__comfyui__get_image(
  filename: [output_filename],
  subfolder: "",
  folder_type: "output"
)
```

**Brand Validation Checklist:**
- [ ] Brand colors visible and accurate?
- [ ] Mood/emotion matches product tier?
- [ ] Text legible (if present)?
- [ ] Composition suitable for intended use?
- [ ] Quality sufficient for target resolution?

### 9. Save as Reusable Template (If Approved)

```
mcp__comfyui__save_template(
  name: "brand_asset_[product]_hero",
  workflow: [successful_workflow_json],
  description: "Hero image for [product] - [color] accent, [theme]",
  tags: ["brand", "[product]", "hero", "16:9"]
)
```

**Template Naming Convention:**
`brand_[product]_[type]_[variant]`

Examples:
- `brand_starter_hero_growth`
- `brand_pro_social_tech`
- `brand_enterprise_slide_results`

### 10. Track Asset in MCP Notes

```
mcp__comfyui__save_note(
  title: "Brand Asset Generation Log",
  content: "
    Date: 2026-02-06
    Asset: Product Hero Image
    Product: [product_name] ([product_color])
    Dimensions: 1920x1080 (16:9)
    Preset: photographic
    Prompt: [full_prompt_text]
    Template: brand_[product]_hero_[variant]
    Output: ComfyUI/output/brand_hero_final_00001_.png
    Status: Approved
  "
)
```

---

## PROMPT ENGINEERING FOR BRAND ASSETS

### Color Psychology for Brand Assets

Use brand colors from `brand-config.yaml` and map them to mood keywords:

| Color Tone | Mood Keywords | Prompt Themes |
|-----------|---------------|---------------|
| **Greens** | Growth, community, fresh start | "lush greenery", "growing plants", "vibrant tones", "organic growth" |
| **Blues** | Technical, building, creation | "electric blue circuitry", "blueprint aesthetics", "technical precision" |
| **Golds** | Premium, success, results | "luxurious golden accents", "champagne tones", "executive sophistication" |
| **Reds** | Energy, urgency, passion | "warm intensity", "bold presence", "dynamic energy" |

### Core Brand Prompt Elements (Use in Every Asset)

**Lighting Direction:**
- "warm golden sunlight" (references #FFD44A)
- "subtle cyan accent lights" (references #22d3ee)
- "dramatic contrast against deep black" (references #0A0A0A)

**Atmosphere Keywords:**
- "sophisticated", "modern", "luxurious"
- "high-tech", "professional", "premium"
- "cinematic lighting", "high contrast"

**Quality Anchors (Always Include):**
- "professional photography"
- "high resolution"
- "sharp focus"
- "studio lighting"

### Example Prompts by Asset Type

**Hero Image (Website Header):**
```text
"Confident entrepreneur in modern tech office, warm golden hour
sunlight streaming through floor-to-ceiling windows creating #FFD44A
tones, deep charcoal walls #0A0A0A, laptop screen casting subtle
cyan glow #22d3ee on desk, sophisticated professional atmosphere,
cinematic composition, professional photography, 16:9 aspect ratio"
```

**Social Media Post (1:1):**
```text
"Abstract geometric pattern of interconnected golden hexagons #FFD44A
floating over deep black void #0A0A0A, electric cyan data streams
#22d3ee connecting nodes, modern tech aesthetic, high contrast,
minimalist composition, perfect square format"
```

**Course Thumbnail:**
```text
"Sleek laptop displaying code editor with cyan syntax highlighting
#22d3ee against dark IDE theme #0A0A0A, warm golden desk lamp
#FFD44A illuminating keyboard, professional tech workspace, shallow
depth of field, cinematic 16:9 composition"
```

---

## PRODUCT COLOR USAGE GUIDELINES

### When to Use Different Brand Colors

| Scenario | Color Strategy |
|----------|---------------|
| **Generic brand asset** | Use core palette from brand-config.yaml |
| **Product-specific page** | Use product-specific color as primary accent |
| **Mixed product showcase** | Use multiple brand colors together |
| **Campaign for one product** | Feature that product's color prominently |

### Multi-Color Harmony

When showcasing multiple brand elements in one asset, use a balanced composition with distinct sections for each color, unified by the dark/background color.

---

## PRESET SELECTION FOR BRAND ASSETS

| Asset Purpose | Recommended Preset | Rationale |
|---------------|-------------------|-----------|
| **Professional photos** | `photographic` | Realistic, studio-quality lighting |
| **Abstract/Tech** | `cinematic` | High contrast, dramatic mood |
| **Illustrations** | `3d_model` or `anime` | Stylized, controlled aesthetic |
| **Product mockups** | `photographic` | Sharp detail, professional |

**Note:** IFGeminiNode has 38 presets available. Check full list:
```
mcp__comfyui__get_capabilities()
```

---

## TROUBLESHOOTING

### Issue 1: Colors Don't Match Brand
**Symptom:** Generated colors are off (e.g., orange instead of gold, gray instead of black)

**Diagnosis:**
- IFGeminiNode interprets colors descriptively, not via hex codes
- "Gold" can mean yellow-gold, orange-gold, or brass

**Solution:**
- Use specific color descriptions: "bright sunny yellow gold like #FFD44A, NOT orange or brass"
- Include color HEX in prompt as anchor: "(#FFD44A golden yellow tones)"
- Add negative examples: "avoid orange, avoid brown"

### Issue 2: Brand Mood Feels Wrong
**Symptom:** Asset looks cheap/generic instead of premium/sophisticated

**Diagnosis:**
- Missing quality anchors in prompt
- Preset doesn't match desired mood

**Solution:**
- Always include: "professional photography", "studio lighting", "high quality"
- For premium feel: add "luxurious", "sophisticated", "cinematic"
- Switch preset from `photographic` to `cinematic` for more drama

### Issue 3: Text in SVG Method Looks Wrong
**Symptom:** Text is distorted, blurry, or wrong after img2img

**Diagnosis:**
- Denoise too high (>0.6) causes AI to reinterpret text
- Font not available during SVG render

**Solution:**
- Keep denoise 0.3-0.5 for text preservation
- Download fonts first: `mcp__comfyui__download_font`
- Verify font render: check SVG output before img2img

### Issue 4: Logo Placement Inconsistent
**Symptom:** Logo appears in different positions across variations

**Diagnosis:**
- txt2img has no spatial control
- Prompt doesn't specify position clearly

**Solution:**
- Use SVG method for precise logo placement
- In prompt, specify position: "logo in top left corner", "centered at bottom"
- Consider generating without logo, add in post-production

### Issue 5: Similar Brand Colors Conflict
**Symptom:** Two similar brand colors clash in the same composition

**Diagnosis:**
- Multiple similar hues in same composition create visual confusion

**Solution:**
- Use one as primary, omit the other
- Or use one as main subject color, the other for accents only
- Keep similar colors in separate sections of the composition

### Issue 6: Output Resolution Too Low
**Symptom:** Image looks good in ComfyUI but pixelated when scaled up

**Diagnosis:**
- IFGeminiNode generates ~1024px, not sufficient for large prints/displays

**Solution:**
- Always upscale brand assets: add UpscaleModelLoader + ImageUpscaleWithModel nodes
- Use 4x-UltraSharp.pth for maximum detail preservation
- Final resolution: 1024px × 4 = 4096px (print quality)

### Issue 7: Generation Keeps Failing
**Symptom:** Workflow times out or returns errors

**Diagnosis:**
- Gemini API rate limit or quota exceeded
- ComfyUI server overloaded

**Solution:**
- Check status: `mcp__comfyui__get_status()`
- Reduce batch_count to 1
- Wait 30 seconds and retry
- Verify Gemini API key in ComfyUI settings

### Issue 8: Inconsistent Style Across Assets
**Symptom:** Each asset looks different despite similar prompts

**Diagnosis:**
- Temperature too high (>1.0) increases randomness
- No seed locking for variations

**Solution:**
- Set temperature to 0.7-0.8 for consistency
- Lock seed for variations: use same seed, modify only specific prompt elements
- Save approved style as template for reuse

---

## CROSS-REFERENCES

**Knowledge Base:**
- KB-capabilities.md → IFGeminiNode full feature list & limitations
- KB-prompting-heuristics.md → Narrative prompt construction techniques
- KB-workflow-patterns.md → JSON workflow structure examples
- KB-pipeline-reference.md → Stage 1 (Generation) + Stage 2 (Upscale) details

**Related Tasks:**
- `generate-image.md` → Basic generation workflow
- `svg-to-image.md` → SVG rendering workflow
- `upscale-image.md` → Post-generation upscaling
- `batch-variations.md` → Generate multiple brand asset variations

**Brand Config:**
- `brand-config.yaml` → Brand colors and guidelines (user-configurable)

---

## SUCCESS CRITERIA

Brand asset is complete when:
- [ ] Colors accurately reflect brand palette (validated against hex codes)
- [ ] Mood/emotion matches product tier personality
- [ ] Quality sufficient for intended use (resolution, sharpness)
- [ ] Composition works at target aspect ratio
- [ ] Asset saved with descriptive filename following naming convention
- [ ] Successful workflow saved as reusable template
- [ ] Generation logged in MCP notes for future reference

---

*ComfyUI Ops Task - Brand Asset Creation*
*Version: 1.0 | Updated: 2026-02-06*
*KB: capabilities, prompting-heuristics, workflow-patterns, pipeline-reference*
