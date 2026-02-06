# Create Banner - Banners para Ads e Social Media
## Cria banners otimizados por plataforma para advertising e redes sociais

---
type: TASK
squad: comfyui-ops
priority: HIGH
created: 2026-02-06
version: 1.0.0
requires: [KB-prompting-heuristics.md, KB-capabilities.md, KB-workflow-patterns.md]
agents: [lyra, kael, forge]
---

## PRE-FLIGHT

### 1. Load Knowledge Base

```
ANTES DE QUALQUER GERAÇÃO:
1. knowledge/KB-prompting-heuristics.md  # Narrative prompting (CRITICAL)
2. knowledge/KB-capabilities.md          # Gemini API params, aspect ratios
3. knowledge/KB-workflow-patterns.md     # Banner pipeline patterns
4. data/platform-specs.yaml             # Platform dimensions reference
5. brand-config.yaml                    # Brand colors (if configured)
6. mcp__comfyui__get_status             # Verify ComfyUI running
```

### 2. Load Brand Identity

```yaml
# Load from brand-config.yaml if it exists
# If not, ask user or use generic high-contrast defaults
brand:
  primary: "#FFD44A"     # Primary accent
  secondary: "#0A0A0A"   # Primary dark
  accent_1: "#22d3ee"    # Accent 1
  accent_2: "#22c55e"    # Accent 2
```

If no `brand-config.yaml` exists, system operates in generic mode — ask user for color preferences or use high-contrast defaults.

---

## WORKFLOW

### 1. Briefing

Elicit from user:

- **Purpose:** What is this banner for? (ad, social post, hero banner, email header, web banner)
- **Platform:** Where will it appear? (Instagram, Facebook, YouTube, LinkedIn, Google Ads, web, email)
- **Message:** What headline/copy goes on it? (max 7 words for headline)
- **Visual style:** Photorealistic, illustration, abstract, minimal, bold graphic?
- **Subject:** Person, product, concept, text-only?
- **Brand:** Use brand colors from brand-config.yaml, or custom/generic?
- **CTA:** Call-to-action text if any (button text)
- **Tone:** Professional, energetic, elegant, tech, warm, bold?

### 2. Platform Specifications

Reference `data/platform-specs.yaml` for exact dimensions. Quick reference:

#### Instagram

| Format | Dimensions | Aspect Ratio | Gemini Ratio | Use Case |
|--------|-----------|--------------|-------------|----------|
| Feed Square | 1080×1080 | 1:1 | `1:1` | Standard post, carousel |
| Feed Portrait | 1080×1350 | 4:5 | `4:5` | Maximum feed real estate |
| Story/Reels | 1080×1920 | 9:16 | `9:16` | Full-screen vertical |
| Feed Landscape | 1080×566 | 1.91:1 | `16:9` (crop) | Wide photos |

#### Facebook

| Format | Dimensions | Aspect Ratio | Gemini Ratio | Use Case |
|--------|-----------|--------------|-------------|----------|
| Feed Post | 1200×630 | 1.91:1 | `16:9` (crop) | News feed |
| Ad (Square) | 1080×1080 | 1:1 | `1:1` | Single image ad |
| Ad (Vertical) | 1080×1350 | 4:5 | `4:5` | Mobile feed ad |
| Cover Photo | 820×312 | 2.63:1 | `16:9` (crop) | Page/group cover |
| Story | 1080×1920 | 9:16 | `9:16` | Facebook stories |

#### Google Ads

| Format | Dimensions | Aspect Ratio | Gemini Ratio | Use Case |
|--------|-----------|--------------|-------------|----------|
| Responsive Display | 1200×628 | 1.91:1 | `16:9` (crop) | Display network |
| Square | 1200×1200 | 1:1 | `1:1` | Display network |
| Landscape | 1200×628 | 1.91:1 | `16:9` (crop) | YouTube banner |

#### LinkedIn

| Format | Dimensions | Aspect Ratio | Gemini Ratio | Use Case |
|--------|-----------|--------------|-------------|----------|
| Feed Post | 1200×627 | 1.91:1 | `16:9` (crop) | Feed posts |
| Banner | 1584×396 | 4:1 | `16:9` (crop) | Profile/company banner |

#### YouTube

| Format | Dimensions | Aspect Ratio | Gemini Ratio | Use Case |
|--------|-----------|--------------|-------------|----------|
| Channel Banner | 2560×1440 | 16:9 | `16:9` (upscale) | Channel art |
| Video Thumbnail | 1280×720 | 16:9 | `16:9` | Use create-thumbnail task |

#### Web / Email

| Format | Dimensions | Aspect Ratio | Gemini Ratio | Use Case |
|--------|-----------|--------------|-------------|----------|
| Hero Banner | 1920×1080 | 16:9 | `16:9` (upscale) | Website hero |
| Email Header | 600×200 | 3:1 | `16:9` (crop) | Email campaigns |
| Web Ad (Leaderboard) | 728×90 | 8:1 | Custom (crop) | Display ads |
| Web Ad (Medium Rect) | 300×250 | 6:5 | `1:1` (crop) | Display ads |

### 3. Banner Design Principles

**The 5 Rules of Effective Banners:**

| Rule | Principle | Why |
|------|-----------|-----|
| **Hierarchy** | ONE dominant element (image or text, not both competing) | Instant comprehension |
| **Contrast** | High contrast between text and background | Readability across devices |
| **Simplicity** | Max 7 words headline + 1 CTA | Scanned in <2 seconds |
| **Color** | 2-3 colors max, one dominant | Visual coherence |
| **Space** | 30%+ breathing room around text | Avoid cramped feeling |

**Color psychology for ads:**

| Color | Emotion | Best For |
|-------|---------|----------|
| Yellow/Gold (#FFD44A) | Urgency, optimism, attention | Flash sales, announcements |
| Red (#EF4444) | Excitement, urgency, passion | Limited time, CTAs |
| Cyan/Blue (#22d3ee) | Trust, technology, calm | Tech, SaaS, B2B |
| Green (#22c55e) | Growth, success, money | Results, financial |
| Orange (#F97316) | Energy, enthusiasm, warmth | How-to, community |
| Purple (#8B5CF6) | Luxury, creativity, premium | Premium offers |
| Black (#0A0A0A) | Elegance, power, authority | Luxury, minimalist |
| White (#FFFFFF) | Clean, simple, modern | Minimalist, health |

### 4. Choose Banner Strategy

| Strategy | Best For | Workflow |
|----------|----------|----------|
| **A: Pure Gemini** | Photographic banners, scene-based ads | txt2img + upscale |
| **B: SVG Layout + Gemini Fill** | Typography-heavy banners, precise text | SVG → render → img2img |
| **C: SVG Only** | Clean graphic banners, no photo needed | SVG → render → upscale |

**Decision matrix:**

| User Need | Strategy | Why |
|-----------|----------|-----|
| Product photo + tagline | A: Pure Gemini | Natural scene composition |
| Bold headline + CTA button | B: SVG + Gemini | Text must be pixel-perfect |
| Minimal brand banner (text + color) | C: SVG Only | No AI generation needed |
| Lifestyle scene + overlay text | A or B | Depends on text importance |
| Abstract/gradient + text | C: SVG Only | Clean graphic design |
| Person + headline | A: Pure Gemini | Photorealistic face generation |

### 5. Strategy A: Pure Gemini Generation

**Use when:** Banner is primarily photographic, text is minimal or integrated into scene.

**Prompt structure for banners:**

```
A [PLATFORM] banner showing [SUBJECT/SCENE], [MOOD/TONE],
[COLOR SCHEME with hex codes], [COMPOSITION notes],
professional advertising quality, high contrast,
clean composition with space for text overlay,
[ASPECT RATIO] format
```

**Prompt modifiers by banner type:**

| Type | Key Modifiers |
|------|---------------|
| Product ad | "hero product center frame, dramatic spotlight, clean background" |
| Lifestyle ad | "natural lifestyle scene, warm authentic mood, brand colors" |
| Event promo | "energetic dynamic composition, bold vibrant colors, event atmosphere" |
| Service ad | "professional setting, trust-building imagery, corporate aesthetic" |
| Hero banner | "wide cinematic composition, dramatic lighting, plenty of negative space for text" |

**Workflow JSON:**

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "YOUR_BANNER_PROMPT_HERE",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 0,
      "batch_count": 1,
      "aspect_ratio": "16:9",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 1,
      "keep_alive": true
    }
  },
  "2": {
    "class_type": "UpscaleModelLoader",
    "inputs": {
      "model_name": "RealESRGAN_x4plus.pth"
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
      "filename_prefix": "banner"
    }
  }
}
```

**Aspect ratio mapping:**

| Platform | Gemini `aspect_ratio` | Post-crop |
|----------|-----------------------|-----------|
| Instagram Feed (1:1) | `1:1` | None needed |
| Instagram Portrait (4:5) | `4:5` | None needed |
| Instagram Story (9:16) | `9:16` | None needed |
| Facebook/LinkedIn Feed | `16:9` | Crop to 1200×630 |
| Google Display | `16:9` | Crop to 1200×628 |
| YouTube Banner | `16:9` | Upscale to 2560×1440 |
| Hero Banner | `16:9` | Upscale to 1920×1080 |
| Email Header | `16:9` | Crop to 600×200 |

### 6. Strategy B: SVG Layout + Gemini Background

**Use when:** Text must be pixel-perfect, typography is the primary design element.

**Step 1: Generate background image (Pure Gemini)**

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Abstract background with [COLOR SCHEME], gradient, soft bokeh, clean space for text overlay, professional advertising aesthetic, [ASPECT RATIO]",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 42,
      "batch_count": 1,
      "aspect_ratio": "16:9",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 1,
      "keep_alive": true
    }
  },
  "2": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "banner_bg"
    }
  }
}
```

**Step 2: Download bold fonts**

```
mcp__comfyui__download_font(
  source: { type: "google", family: "Montserrat", weight: 900 }
)
```

Recommended banner fonts:

| Font | Weight | Use For |
|------|--------|---------|
| Montserrat Black | 900 | Headlines, impact |
| Inter Black | 900 | Clean modern |
| Oswald Bold | 700 | Condensed headlines |
| Bebas Neue | 400 | All-caps impact |
| Poppins Bold | 700 | Friendly, rounded |
| Playfair Display | 700 | Elegant, premium |

**Step 3: Create SVG text overlay**

Template for Instagram Square (1080×1080):

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1080 1080">
  <!-- Semi-transparent background card -->
  <rect x="80" y="350" width="920" height="380"
        rx="24" fill="#0A0A0A" opacity="0.8"/>

  <!-- Headline -->
  <text x="540" y="480" text-anchor="middle"
        font-family="Montserrat" font-size="72"
        font-weight="900" fill="#FFD44A"
        stroke="#000000" stroke-width="3">
    HEADLINE HERE
  </text>

  <!-- Subheadline -->
  <text x="540" y="560" text-anchor="middle"
        font-family="Montserrat" font-size="36"
        font-weight="600" fill="#FFFFFF">
    Supporting text goes here
  </text>

  <!-- CTA Button -->
  <rect x="370" y="610" width="340" height="70"
        rx="35" fill="#FFD44A"/>
  <text x="540" y="658" text-anchor="middle"
        font-family="Montserrat" font-size="28"
        font-weight="700" fill="#0A0A0A">
    CALL TO ACTION
  </text>
</svg>
```

Template for Facebook/LinkedIn (1200×630):

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 630">
  <!-- Left-aligned text block -->
  <rect x="60" y="140" width="600" height="350"
        rx="20" fill="#0A0A0A" opacity="0.85"/>

  <!-- Headline -->
  <text x="360" y="260" text-anchor="middle"
        font-family="Montserrat" font-size="56"
        font-weight="900" fill="#FFD44A"
        stroke="#000000" stroke-width="2">
    HEADLINE
  </text>

  <!-- Body copy -->
  <text x="360" y="330" text-anchor="middle"
        font-family="Montserrat" font-size="28"
        font-weight="500" fill="#FFFFFF">
    Supporting copy line here
  </text>

  <!-- CTA Button -->
  <rect x="220" y="370" width="280" height="60"
        rx="30" fill="#22d3ee"/>
  <text x="360" y="410" text-anchor="middle"
        font-family="Montserrat" font-size="24"
        font-weight="700" fill="#0A0A0A">
    LEARN MORE
  </text>
</svg>
```

**Step 4: Render SVG**

```
mcp__comfyui__render_svg(
  svg: "<svg>...</svg>",
  width: 1080,   # Match platform dimensions
  height: 1080,
  background: "transparent",
  fonts: [
    { name: "Montserrat", family: "Montserrat" }
  ]
)
```

Returns: `text_overlay.png` with transparency.

**Step 5: Composite text over background**

**Option 1: img2img composite (ComfyUI)**

```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": { "image": "banner_bg.png" }
  },
  "2": {
    "class_type": "LoadImage",
    "inputs": { "image": "text_overlay.png" }
  },
  "3": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Overlay the text layout from the second image onto the background image. Maintain exact text clarity, positioning, and colors. Professional advertising banner.",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 0.7,
      "images": ["1", 0],
      "keep_alive": false
    }
  },
  "4": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["3", 0],
      "filename_prefix": "banner_final"
    }
  }
}
```

**Option 2: External compositing** (recommended for pixel-perfect text)
Use external tool (Photoshop, GIMP, Canva, ImageMagick) to composite transparent PNG over background.

**Note:** img2img may transform text. For pixel-perfect results, use external compositing.

### 7. Strategy C: SVG Only (No AI Generation)

**Use when:** Clean graphic design — solid colors, gradients, text, shapes. No photographic elements needed.

**Full SVG example — Instagram ad:**

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1080 1080">
  <!-- Background gradient -->
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#0A0A0A"/>
      <stop offset="100%" style="stop-color:#1a1a2e"/>
    </linearGradient>
  </defs>
  <rect width="1080" height="1080" fill="url(#bg)"/>

  <!-- Accent line -->
  <rect x="80" y="380" width="120" height="6" rx="3" fill="#FFD44A"/>

  <!-- Headline -->
  <text x="80" y="460" font-family="Montserrat" font-size="64"
        font-weight="900" fill="#FFFFFF">
    YOUR HEADLINE
  </text>
  <text x="80" y="540" font-family="Montserrat" font-size="64"
        font-weight="900" fill="#FFD44A">
    GOES HERE
  </text>

  <!-- Body -->
  <text x="80" y="620" font-family="Montserrat" font-size="28"
        font-weight="400" fill="#AAAAAA">
    Supporting text with details
  </text>

  <!-- CTA -->
  <rect x="80" y="700" width="300" height="64" rx="32" fill="#FFD44A"/>
  <text x="230" y="742" text-anchor="middle"
        font-family="Montserrat" font-size="24"
        font-weight="700" fill="#0A0A0A">
    GET STARTED
  </text>
</svg>
```

Render with `mcp__comfyui__render_svg`, then optionally upscale if higher resolution needed.

### 8. Brand Integration

If `brand-config.yaml` exists, load and apply brand colors:

```yaml
# Read from brand-config.yaml
brand:
  colors:
    primary: "#FFD44A"    # → Headlines, CTA backgrounds, accent lines
    secondary: "#0A0A0A"  # → Backgrounds, text color on light
    accent_1: "#22d3ee"   # → Secondary CTAs, highlights
    accent_2: "#22c55e"   # → Success elements, check marks
```

**Brand color application in prompts:**

```
A [PLATFORM] advertising banner with [BRAND NAME] aesthetic,
[PRIMARY COLOR] (#HEX) as accent highlight,
[SECONDARY COLOR] (#HEX) as base,
professional brand consistency, [SUBJECT], [COMPOSITION]
```

**Brand color application in SVG:**
- `fill="#PRIMARY"` for headlines and CTAs
- `fill="#SECONDARY"` for backgrounds
- `fill="#ACCENT_1"` for secondary elements
- Use product-specific color if banner is tier-specific

If no brand config exists, use high-contrast generic palette: gold (#FFD44A), black (#0A0A0A), white (#FFFFFF).

### 9. Batch Strategy (Multi-Platform)

When user needs banners for multiple platforms simultaneously:

**Step 1: Generate master asset (highest resolution)**

Generate at 16:9 (largest common ratio) and crop/adapt for each platform:

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Professional advertising banner, [SUBJECT], [COLOR SCHEME], clean composition with subject on right, text space on left, high contrast, commercial quality, 16:9",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 42,
      "batch_count": 4,
      "aspect_ratio": "16:9",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 4,
      "keep_alive": true
    }
  },
  "2": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "banner_master"
    }
  }
}
```

**Step 2: Select best, then generate platform variants**

For each required platform, re-generate with the correct aspect ratio:

| Platform | aspect_ratio | filename_prefix |
|----------|-------------|-----------------|
| Instagram Square | `1:1` | `banner_ig_square` |
| Instagram Portrait | `4:5` | `banner_ig_portrait` |
| Instagram Story | `9:16` | `banner_ig_story` |
| Facebook Feed | `16:9` | `banner_fb_feed` |
| LinkedIn | `16:9` | `banner_li` |

**Step 3: Add text overlay per platform**

Create SVG overlays with dimensions matching each platform. Adjust text size and position for each format.

### 10. Validate and Execute

**Pre-execution checklist:**
```
[ ] Platform and dimensions confirmed
[ ] Aspect ratio matches platform
[ ] Brand colors applied (if configured)
[ ] Prompt is narrative (not keywords)
[ ] Text content finalized (headline, CTA)
```

**Validate workflow:**
```
mcp__comfyui__validate_workflow(workflow: <JSON>)
```

**Execute:**
```
mcp__comfyui__run_workflow(
  workflow: <JSON>,
  sync: true,
  name: "banner_platform_purpose",
  outputMode: "auto",
  imageFormat: "jpeg",
  imageQuality: 95
)
```

**Naming convention:**
```
banner_{platform}_{purpose}_v{n}
```
Examples: `banner_ig_flash_sale_v1`, `banner_fb_product_launch_v2`, `banner_hero_homepage_v1`

---

## TROUBLESHOOTING

### Issue 1: Text illegible at small display sizes

**Symptom:** Banner text can't be read on mobile or in-feed display.

**Causes:**
1. Font too small for the platform
2. Low contrast with background
3. No text background/stroke

**Solutions:**
- Minimum font sizes by platform:
  - Instagram feed (displays ~300px wide): headline 48px+
  - Facebook feed: headline 40px+
  - Story (full-screen): headline 64px+
- Add semi-transparent background card behind text
- Add stroke: `stroke="#000000" stroke-width="2-4"`
- High contrast: light text on dark card, or vice versa
- Test by viewing at actual display size

### Issue 2: Composition doesn't leave space for text

**Symptom:** Generated image fills entire frame, no clean area for text overlay.

**Causes:**
1. Prompt didn't specify text space
2. Subject centered without composition direction

**Solutions:**
- Add to prompt: "clean space on [left/right/top/bottom] for text overlay"
- Use "rule of thirds composition, subject on right third"
- Add "negative space" or "breathing room"
- Generate with batch_count: 4, select the one with best text space
- For guaranteed text space: use Strategy B (SVG + background)

### Issue 3: Colors don't match brand palette

**Symptom:** Generated colors approximate but don't match brand hex codes.

**Causes:**
1. Gemini interprets color names loosely
2. Lighting affects perceived color
3. Color described by name instead of hex

**Solutions:**
- Always specify hex codes in prompt: "gold (#FFD44A)" not just "gold"
- For exact colors, use Strategy B or C (SVG provides pixel-perfect color)
- Check `brand-config.yaml` for correct hex values
- Add to prompt: "exact color palette:" followed by hex codes

### Issue 4: Banner looks generic / lacks impact

**Symptom:** Professional but forgettable, doesn't stop the scroll.

**Causes:**
1. Safe, conventional composition
2. Muted colors
3. No emotional hook

**Solutions:**
- Increase contrast: "bold vibrant colors, high contrast"
- Add dynamic composition: "diagonal lines, asymmetric layout"
- Stronger emotion: "dramatic lighting, intense spotlight"
- One dominant visual element (face, product, or headline)
- Use unexpected color combination
- Test with batch variations (different compositions)

### Issue 5: Multi-platform banners look inconsistent

**Symptom:** Same campaign looks different across platforms.

**Causes:**
1. Regenerated independently for each platform
2. Different seeds, prompts, or styles

**Solutions:**
- Generate master image first (16:9 high-res)
- Use same seed and similar prompt for all platform variants
- Keep brand colors consistent across all versions
- Use Strategy B (SVG overlay) for text — guarantees text consistency
- Create a "campaign brief" before generating (colors, mood, copy)

### Issue 6: CTA button looks painted/unreal

**Symptom:** Gemini generates a painted-looking button instead of a clean UI element.

**Causes:**
1. Gemini treats buttons as part of the scene

**Solutions:**
- Never include CTA buttons in Gemini prompt
- Always use Strategy B or C for CTA buttons (SVG renders clean UI elements)
- Add button as SVG overlay in post-processing

### Issue 7: Upscaled banner has artifacts

**Symptom:** Visible artifacts, noise, or blurriness after upscaling.

**Causes:**
1. Original image too small
2. Wrong upscaler for content type

**Solutions:**
- For photography: use `4x-UltraSharp.pth`
- For graphics/text: use `RealESRGAN_x4plus.pth`
- If still blurry: generate at highest native resolution first
- For banners with faces: add CodeFormer before upscale (use `txt2img-gemini-face.json` pattern)

### Issue 8: File size too large for platform

**Symptom:** Platform rejects upload due to file size.

**Solutions by platform:**
- Instagram: Max 30MB → usually not an issue
- Facebook ads: Max 30MB → usually not an issue
- Google Ads: Max 150KB → compress aggressively (JPEG quality 70-80)
- Email: Max 200KB → JPEG quality 60-70, reduce dimensions
- Use `imageFormat: "jpeg"` and `imageQuality: 85` in run_workflow
- For Google/email: crop to exact platform dimensions, compress with external tool

---

## EXAMPLES

### Example 1: Instagram Ad — Flash Sale (Pure Gemini)

**Scenario:** Flash sale banner for Instagram feed, product-focused.

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "An Instagram advertising banner for a flash sale, dramatic spotlight on a sleek modern product in center frame, bold gold (#FFD44A) and black (#0A0A0A) color scheme, clean negative space on left for text overlay, high contrast, commercial product photography, urgency feeling, 1:1 square format",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 42,
      "batch_count": 1,
      "aspect_ratio": "1:1",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 1,
      "keep_alive": true
    }
  },
  "2": {
    "class_type": "UpscaleModelLoader",
    "inputs": { "model_name": "RealESRGAN_x4plus.pth" }
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
      "filename_prefix": "banner_ig_flash_sale"
    }
  }
}
```

### Example 2: Facebook Ad — SVG Text + Gemini Background

**Scenario:** Facebook feed ad with bold headline and CTA button.

**Step 1: Generate background**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Abstract professional background with deep blue (#1e3a5f) to black (#0A0A0A) gradient, subtle tech particles, bokeh lights, clean and modern, space for text overlay on left half, advertising quality, 16:9",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 42,
      "batch_count": 1,
      "aspect_ratio": "16:9",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 1,
      "keep_alive": true
    }
  },
  "2": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "banner_fb_bg"
    }
  }
}
```

**Step 2: Create SVG overlay**
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 630">
  <text x="80" y="220" font-family="Montserrat" font-size="56"
        font-weight="900" fill="#FFFFFF">
    TRANSFORM YOUR
  </text>
  <text x="80" y="290" font-family="Montserrat" font-size="56"
        font-weight="900" fill="#22d3ee">
    BUSINESS WITH AI
  </text>
  <text x="80" y="360" font-family="Montserrat" font-size="24"
        font-weight="400" fill="#CCCCCC">
    Start building systems that work for you
  </text>
  <rect x="80" y="400" width="240" height="56" rx="28" fill="#22d3ee"/>
  <text x="200" y="436" text-anchor="middle"
        font-family="Montserrat" font-size="20"
        font-weight="700" fill="#0A0A0A">
    GET STARTED
  </text>
</svg>
```

**Step 3: Render SVG and composite**

### Example 3: LinkedIn Banner — SVG Only (No AI)

**Scenario:** Clean professional company page banner.

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1584 396">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:#0A0A0A"/>
      <stop offset="100%" style="stop-color:#1a1a2e"/>
    </linearGradient>
  </defs>
  <rect width="1584" height="396" fill="url(#bg)"/>

  <!-- Accent line -->
  <rect x="100" y="120" width="80" height="4" rx="2" fill="#FFD44A"/>

  <!-- Company name -->
  <text x="100" y="190" font-family="Montserrat" font-size="48"
        font-weight="900" fill="#FFFFFF">
    YOUR BRAND
  </text>

  <!-- Tagline -->
  <text x="100" y="240" font-family="Montserrat" font-size="24"
        font-weight="400" fill="#AAAAAA">
    Building the future with AI
  </text>

  <!-- Right side decorative element -->
  <circle cx="1400" cy="198" r="120" fill="#FFD44A" opacity="0.1"/>
  <circle cx="1400" cy="198" r="80" fill="#FFD44A" opacity="0.15"/>
</svg>
```

Render at 1584×396 with `mcp__comfyui__render_svg`.

### Example 4: Multi-Platform Campaign (Batch)

**Scenario:** Launch campaign — need Instagram, Facebook, and Story versions.

**Step 1: Generate master at 16:9**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "Professional person confidently using laptop with holographic AI interface, warm gold (#FFD44A) and dark (#0A0A0A) palette, cinematic lighting, subject positioned in right third, ample negative space on left for text, commercial advertising quality, photorealistic, 16:9",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 42,
      "batch_count": 4,
      "aspect_ratio": "16:9",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 4,
      "keep_alive": true
    }
  },
  "2": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "banner_campaign_master"
    }
  }
}
```

**Step 2: Select best, re-generate per platform with same seed**

Repeat with `aspect_ratio: "1:1"` for Instagram, `"9:16"` for Story, keeping same seed for consistency.

---

## CROSS-REFERENCES

**Related tasks:**
- `create-thumbnail.md` — YouTube thumbnails (specialized version of banner)
- `create-brand-asset.md` — Brand assets with design system
- `svg-to-image.md` — SVG rendering technique details
- `batch-variations.md` — Batch generation for A/B testing
- `generate-image.md` — Core txt2img generation

**Knowledge bases:**
- `KB-capabilities.md` — Gemini API aspect ratios and limits
- `KB-prompting-heuristics.md` — Narrative prompting, composition
- `KB-workflow-patterns.md` — Pipeline patterns
- `data/platform-specs.yaml` — Complete platform dimension reference
- `brand-config.yaml` — Brand colors (user-configurable)

**Workflow JSONs:**
- `workflows/txt2img-gemini-upscale.json` — Base pipeline for photographic banners
- `workflows/txt2img-gemini-face.json` — Pipeline with face restore (for person-focused banners)
- `workflows/img2img-gemini.json` — For compositing text over background
