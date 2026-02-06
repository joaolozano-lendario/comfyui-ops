# Create Thumbnail - Thumbnail YouTube/Curso

Create high-CTR thumbnails for YouTube videos and course modules.

---

## PRE-FLIGHT

### 1. Load Knowledge Base

Ensure you have read these files before proceeding:

```
knowledge/KB-capabilities.md       # Gemini API capabilities
knowledge/KB-prompting-heuristics.md  # Visual composition and emotion
knowledge/KB-workflow-patterns.md     # Thumbnail workflow patterns
brand-config.yaml                                # Brand colors (if configured)
```

### 2. Verify ComfyUI Status

```
mcp__comfyui__get_status
```

Expected: "ready" at http://127.0.0.1:8188

---

## WORKFLOW

### 1. Briefing

Elicit from user:
- **Video/course topic:** What is the content about?
- **Target emotion:** Curiosity, urgency, surprise, excitement, authority, shock?
- **Text overlay:** Headline (max 5 words, ideally 3)
- **Visual style:** Photorealistic, illustration, hybrid, abstract?
- **Include person:** Yes (face/expression) or No (product/concept)?
- **Brand:** Use brand colors from brand-config.yaml, or generic?

### 2. YouTube Thumbnail Specifications

| Spec | Value | Why |
|------|-------|-----|
| **Dimensions** | 1280×720 | YouTube standard, 16:9 aspect ratio |
| **Max file size** | 2MB | YouTube upload limit |
| **Format** | JPG or PNG | JPG for photos, PNG for graphics |
| **Aspect ratio** | 16:9 | Displays correctly on all devices |
| **Safe zone** | Center 1120×630 | Avoid text/faces near edges (mobile crop) |

**Gemini output:** Use `aspect_ratio: "16:9"` → generates 1344×768, crop to 1280×720 in post.

### 3. High-CTR Thumbnail Principles

**The 6 Elements of Clickable Thumbnails:**

| Element | Rule | Why |
|---------|------|-----|
| **Face/Subject** | LARGE, fills 60%+ of frame | Human faces attract attention |
| **Expression** | EXAGGERATED emotion (surprise, excitement, shock) | Emotion = curiosity |
| **Contrast** | High contrast colors, avoid muddy tones | Stands out in feed |
| **Text** | MAX 5 words, bold, legible at small size | Quick comprehension |
| **Color** | Bright, saturated (yellow, red, cyan) | Visibility in thumbnails |
| **Composition** | Asymmetric, dynamic (not centered) | More engaging than static |

**Color psychology for CTR:**

| Color | Emotion | Use For |
|-------|---------|---------|
| Yellow/Gold (#FFD44A) | Urgency, optimism | Attention-grabbing |
| Red | Excitement, urgency | Breaking news, alerts |
| Cyan/Blue (#22d3ee) | Trust, tech | AI/tech content |
| Green (#22c55e) | Growth, success | Results, progress |
| Orange | Energy, enthusiasm | How-to, tutorials |
| Purple | Luxury, mystery | Premium content |

### 4. Choose Thumbnail Strategy

Two primary approaches:

| Strategy | Best For | Workflow |
|----------|----------|----------|
| **A: Pure Gemini** | Photorealistic thumbnails with in-scene text | txt2img only |
| **B: SVG Text + Gemini Background** | Bold typography with precise positioning | SVG → render → img2img/composite |

**Decision matrix:**

| User Need | Strategy | Why |
|-----------|----------|-----|
| Face with emotion + minimal text | Pure Gemini | Natural photo generation |
| Bold headline as primary element | SVG + Gemini | Text readability critical |
| Product showcase + tagline | Pure Gemini | Integrated composition |
| Typography-first design | SVG + Gemini | Font control |
| Split-screen comparison | SVG + Gemini | Precise layout |

### 5. Strategy A: Pure Gemini Generation

**Use when:** Thumbnail is primarily photographic or text is minimal/integrated.

**Prompt structure for high-CTR thumbnails:**
```
A YouTube thumbnail showing [SUBJECT], [EXPRESSION/EMOTION],
[COMPOSITION], bold vibrant colors, high contrast,
[COLOR SCHEME], photorealistic/illustration style,
dramatic lighting, 16:9 format
```

**Expression vocabulary:**
- Surprise: "wide eyes, mouth open in shock"
- Excitement: "enthusiastic smile, energetic pose"
- Curiosity: "intrigued expression, leaning forward"
- Authority: "confident gaze, professional demeanor"
- Urgency: "intense focus, urgent expression"

**Examples by content type:**

| Content Type | Prompt |
|--------------|--------|
| Tutorial | "A person with excited expression showing laptop screen, bright modern background, vibrant colors with cyan accents (#22d3ee), high contrast, photorealistic, YouTube thumbnail 16:9" |
| Product review | "Smartphone held up prominently in hand, dramatic spotlight, gold (#FFD44A) and black (#0A0A0A) color scheme, commercial photography, sharp focus, 16:9 thumbnail" |
| Listicle | "Energetic presenter with surprised expression, three floating icons beside face, bright bold colors, high contrast, illustrated style, 16:9 YouTube thumbnail" |
| Case study | "Before-and-after split screen, dramatic transformation, contrasting sides, bold visual impact, professional photography, 16:9" |

**Workflow:**

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "high-CTR thumbnail prompt here",
      "model": "gemini-2.0-flash-exp",
      "aspect_ratio": "16:9",
      "seed": 42,
      "keep_alive": false
    }
  },
  "2": {
    "class_type": "UpscaleModelLoader",
    "inputs": {
      "model_name": "4x-UltraSharp.pth"
    }
  },
  "3": {
    "class_type": "ImageUpscaleWithModel",
    "inputs": {
      "upscale_model": ["2", 0],
      "image": ["1", 0]
    }
  },
  "4": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["3", 0],
      "filename_prefix": "thumbnail_topic_v1"
    }
  }
}
```

**Why upscale:** Ensures text/faces are sharp even when cropped to 1280×720.

### 6. Strategy B: SVG Text Overlay + Gemini Background

**Use when:** Text must be perfectly legible, typography is primary design element.

**Step 1: Download Bold Fonts**

```
mcp__comfyui__download_font (
  source: {
    type: "google",
    family: "Montserrat",
    weight: 900
  }
)
```

Recommended fonts for thumbnails:

| Font | Weight | Use For |
|------|--------|---------|
| Montserrat Black | 900 | Bold headlines |
| Inter Black | 900 | Modern tech content |
| Oswald Bold | 700 | Impact text |
| Bebas Neue | 400 (naturally bold) | All-caps headlines |
| Roboto Black | 900 | Clean readability |

**Step 2: Generate Background (Pure Gemini)**

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Energetic presenter with excited expression, bright modern studio background, space for text overlay on left side, vibrant colors, high contrast, photorealistic, 16:9 YouTube thumbnail",
      "model": "gemini-2.0-flash-exp",
      "aspect_ratio": "16:9",
      "seed": 42
    }
  },
  "2": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "thumbnail_bg"
    }
  }
}
```

**Step 3: Create SVG Text Overlay**

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1280 720">
  <!-- Semi-transparent background for text readability -->
  <rect x="50" y="250" width="600" height="220"
        rx="20" fill="#0A0A0A" opacity="0.8"/>

  <!-- Main headline (max 5 words) -->
  <text x="350" y="340" text-anchor="middle"
        font-family="Montserrat" font-size="64"
        font-weight="900" fill="#FFD44A"
        stroke="#000000" stroke-width="3">
    HEADLINE TEXT
  </text>

  <!-- Subheadline (optional, small) -->
  <text x="350" y="410" text-anchor="middle"
        font-family="Montserrat" font-size="32"
        font-weight="700" fill="#FFFFFF"
        stroke="#000000" stroke-width="2">
    Supporting Text
  </text>
</svg>
```

**Typography rules for thumbnails:**
- **Font size:** 64-80px for headlines (legible at small size)
- **Stroke:** Add black stroke (2-4px) for readability on any background
- **Contrast:** White/yellow/cyan text on dark background or vice versa
- **Alignment:** Left-aligned or centered, avoid right-align
- **Safe zone:** Keep text in center 1120×630 area

**Step 4: Render SVG**

```
mcp__comfyui__render_svg (
  svg: "<svg>...</svg>",
  width: 1280,
  height: 720,
  background: "transparent",
  fonts: [
    { name: "Montserrat", family: "Montserrat" }
  ]
)
```

Returns: `text_overlay.png` with transparency.

**Step 5: Composite Text Over Background**

**Option 1: Direct composite (outside ComfyUI)**
Use external tool to composite transparent PNG over background.

**Option 2: img2img integration (ComfyUI)**

```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": { "image": "thumbnail_bg.png" }
  },
  "2": {
    "class_type": "LoadImage",
    "inputs": { "image": "text_overlay.png" }
  },
  "3": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Overlay the text elements from second image onto the background, maintain text clarity and contrast, professional YouTube thumbnail aesthetic",
      "image": ["1", 0],
      "image_2": ["2", 0],
      "model": "gemini-2.0-flash-exp",
      "keep_alive": false
    }
  },
  "4": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["3", 0],
      "filename_prefix": "thumbnail_final"
    }
  }
}
```

**Note:** img2img may transform text. For pixel-perfect text, use external compositing.

### 7. Brand Integration (Optional)

If `brand-config.yaml` exists, load brand colors and apply them:

**Branded thumbnails should include:**
- Primary brand color for headlines or highlights
- Dark brand color for backgrounds or text strokes
- Product-specific colors if applicable

**Prompt template for branded thumbnails:**
```
A YouTube thumbnail for [TOPIC], featuring [SUBJECT/PERSON],
[BRAND] aesthetic, [PRIMARY COLOR] accents,
[DARK COLOR] elements, modern professional style,
high contrast vibrant colors, photorealistic, 16:9
```

If no brand config exists, use high-contrast generic colors (yellow, cyan, white on dark).

### 8. Batch Strategy (4 Variations for Testing)

Generate 4 variations to A/B test:

**Variation matrix:**

| Version | Variable Changed |
|---------|------------------|
| A | Original expression/angle |
| B | Different facial expression (more extreme) |
| C | Different color scheme |
| D | Different composition/angle |

**Batch workflow:**

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "base prompt, surprised expression",
      "aspect_ratio": "16:9",
      "seed": 100
    }
  },
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "base prompt, excited expression",
      "aspect_ratio": "16:9",
      "seed": 200
    }
  },
  "3": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "base prompt, warm color scheme",
      "aspect_ratio": "16:9",
      "seed": 300
    }
  },
  "4": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "base prompt, close-up composition",
      "aspect_ratio": "16:9",
      "seed": 400
    }
  }
}
```

Run with `sync: false` for parallel generation.

**Testing strategy:**
1. Upload all 4 versions to YouTube (private)
2. Test on different devices/screen sizes
3. Check legibility at small thumbnail size
4. Select highest CTR after 24-48 hours

### 9. Validate and Execute

```
mcp__comfyui__validate_workflow (workflow: <JSON>)
```

```
mcp__comfyui__run_workflow (
  workflow: <JSON>,
  sync: true,
  name: "thumbnail_video_topic",
  outputMode: "all",
  imageFormat: "jpeg",
  imageQuality: 95
)
```

---

## TROUBLESHOOTING

### Issue 1: Text too small/illegible

**Symptom:** Text can't be read when thumbnail is viewed at small size.

**Causes:**
1. Font size too small (<60px)
2. Low contrast with background
3. No stroke/outline on text

**Solutions:**
- Font size: 64-80px minimum for headlines
- Add black stroke: `stroke="#000000" stroke-width="3"`
- Use bold fonts (weight 900)
- High contrast: white/yellow on dark, or dark on light
- Test at 120px wide (actual display size)

### Issue 2: Face not prominent enough

**Symptom:** Subject's face is too small or not attention-grabbing.

**Causes:**
1. Composition too wide
2. Face not filling frame
3. Expression not strong enough

**Solutions:**
- Prompt: "close-up portrait, face fills 60% of frame"
- Exaggerate expression: "wide-eyed surprise", "huge enthusiastic smile"
- Use "dramatic lighting on face"
- Crop tighter in post-processing

### Issue 3: Colors look muddy/not vibrant

**Symptom:** Thumbnail doesn't pop in feed, looks dull.

**Causes:**
1. Prompt lacks color direction
2. No "high contrast" specified
3. Complex color palette

**Solutions:**
- Add to prompt: "vibrant saturated colors, high contrast"
- Specify exact colors: "bright cyan (#22d3ee) and gold (#FFD44A)"
- Avoid: "realistic colors" (too muted)
- Use: "bold bright colors, commercial advertising style"

### Issue 4: Thumbnail not clickable/lacks curiosity

**Symptom:** Thumbnail is well-designed but doesn't generate clicks.

**Causes:**
1. Too much information (overwhelm)
2. No emotional hook
3. Static composition

**Solutions:**
- Simplify: Max 5 words text, one clear subject
- Add curiosity gap: "What happens when..." vs "Tutorial on..."
- Dynamic composition: Asymmetric, diagonal lines, movement
- Exaggerated expression: Surprise, shock, excitement
- Test different emotions with batch generation

### Issue 5: SVG text overlay ruins background

**Symptom:** img2img transforms background when compositing text.

**Causes:**
1. Gemini interpreting as transformation rather than overlay
2. Prompt too complex

**Solutions:**
- Use external compositing tool (Photoshop, GIMP, Canva)
- Or minimal img2img prompt: "Overlay text, no other changes"
- Or skip img2img, composite PNG with transparency
- Render SVG with semi-transparent background box for readability

### Issue 6: Batch variations too similar

**Symptom:** All 4 thumbnails look nearly identical.

**Causes:**
1. Seeds too close
2. Prompt variations too subtle

**Solutions:**
- Use distinct seeds: 100, 200, 300, 400
- Make variations significant:
  - Version A: Surprised expression
  - Version B: Excited expression
  - Version C: Warm color scheme
  - Version D: Close-up composition
- Change subject angle/pose between versions

### Issue 7: Brand colors not matching

**Symptom:** Generated colors don't match brand palette.

**Causes:**
1. Gemini interpreting color name differently
2. Lighting affecting color perception

**Solutions:**
- Always use hex codes from brand-config.yaml in the prompt
- For precise colors, use SVG strategy
- Validate output colors before delivery
- If using Pure Gemini: "exact color palette: [COLOR] #HEXCODE"

### Issue 8: File size over 2MB limit

**Symptom:** YouTube rejects upload due to file size.

**Causes:**
1. High quality JPEG or PNG with transparency
2. Upscaled to 4K+ resolution

**Solutions:**
- Export as JPEG with quality 90-95 (not 100)
- Crop to exactly 1280×720 (no larger)
- Use compression tool (TinyPNG, Squoosh)
- Avoid PNG unless transparency needed

---

## EXAMPLES

### Example 1: Tutorial Thumbnail (Pure Gemini)

**Scenario:** "How to Build AI Agents" tutorial video.

**Workflow:**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "A YouTube thumbnail showing excited developer at laptop with holographic AI visualization floating above screen, surprised expression, blue (#3B82F6) and gold (#FFD44A) color scheme, modern tech aesthetic, dramatic lighting, high contrast vibrant colors, photorealistic, 16:9",
      "model": "gemini-2.0-flash-exp",
      "aspect_ratio": "16:9",
      "seed": 42
    }
  },
  "2": {
    "class_type": "UpscaleModelLoader",
    "inputs": { "model_name": "4x-UltraSharp.pth" }
  },
  "3": {
    "class_type": "ImageUpscaleWithModel",
    "inputs": {
      "upscale_model": ["2", 0],
      "image": ["1", 0]
    }
  },
  "4": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["3", 0],
      "filename_prefix": "thumbnail_ai_agents"
    }
  }
}
```

### Example 2: Course Module Thumbnail (SVG + Gemini)

**Scenario:** "Module 3: Sales" for a premium tier course.

**Step 1: Generate background**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Professional business setting with charts and growth visuals, gold (#C9A227) and black (#0A0A0A) color scheme, space for text on left side, corporate aesthetic, high contrast, 16:9",
      "aspect_ratio": "16:9",
      "seed": 42
    }
  }
}
```

**Step 2: Create SVG text**
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1280 720">
  <rect x="50" y="200" width="700" height="320"
        rx="30" fill="#0A0A0A" opacity="0.85"/>
  <text x="400" y="320" text-anchor="middle"
        font-family="Montserrat" font-size="72"
        font-weight="900" fill="#C9A227"
        stroke="#000000" stroke-width="4">
    MÓDULO 3
  </text>
  <text x="400" y="410" text-anchor="middle"
        font-family="Montserrat" font-size="48"
        font-weight="700" fill="#FFFFFF"
        stroke="#000000" stroke-width="3">
    Vendas para Técnicos
  </text>
</svg>
```

**Step 3: Composite externally or via img2img**

### Example 3: Batch Variations (4 Expressions)

**Scenario:** Testing different emotions for product launch video.

**Workflow:**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Person with surprised wide-eyed expression holding smartphone, bright colors, 16:9 thumbnail",
      "aspect_ratio": "16:9",
      "seed": 100
    }
  },
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Person with excited enthusiastic smile holding smartphone, bright colors, 16:9 thumbnail",
      "aspect_ratio": "16:9",
      "seed": 200
    }
  },
  "3": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Person with intrigued curious expression holding smartphone, bright colors, 16:9 thumbnail",
      "aspect_ratio": "16:9",
      "seed": 300
    }
  },
  "4": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Person with confident authoritative expression holding smartphone, bright colors, 16:9 thumbnail",
      "aspect_ratio": "16:9",
      "seed": 400
    }
  }
}
```

Run with `sync: false`, then test all 4 on YouTube.

---

## CROSS-REFERENCES

**Related tasks:**
- `create-banner.md` - Similar principles for social media
- `svg-to-image.md` - SVG overlay technique details
- `generate-from-reference.md` - img2img for background variations
- `batch-variations.md` - Batch generation for A/B testing

**Knowledge bases:**
- `KB-capabilities.md` - Gemini API aspect ratios and limitations
- `KB-prompting-heuristics.md` - Emotion and composition prompting
- `KB-workflow-patterns.md` - Thumbnail workflow patterns
- `brand-config.yaml` - Brand colors (user-configurable)

**External resources:**
- YouTube Creator Academy: Thumbnail best practices
- VidIQ: Thumbnail A/B testing guide
- Canva: Thumbnail design templates
