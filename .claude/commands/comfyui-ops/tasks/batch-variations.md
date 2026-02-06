# Batch Variations - Multiplas Variacoes para Selecao

Gera múltiplas variações de um conceito visual para comparação, seleção e A/B testing usando estratégias de variação inteligentes.

---

## PRE-FLIGHT CHECKS

### 1. Knowledge Base Status
Confirm access to:
- `knowledge/KB-capabilities.md` - IFGeminiNode batch_count parameter & seed control
- `knowledge/KB-prompting-heuristics.md` - Prompt variation techniques
- `knowledge/KB-workflow-patterns.md` - Batch workflow patterns
- `knowledge/KB-system-inventory.md` - Available presets for style variations

### 2. MCP Connection
```bash
# Verify ComfyUI is running and responsive
curl -s http://127.0.0.1:8188/system_stats
```

Test MCP tools:
```
mcp__comfyui__get_status
mcp__comfyui__list_tasks  # Check for running tasks before starting batch
```

### 3. Base Concept Defined
Before generating variations, ensure:
- [ ] Base prompt is validated (generated at least 1 successful image)
- [ ] Target aspect ratio confirmed
- [ ] Desired preset identified
- [ ] Number of variations decided (recommended: 4-10 per strategy)

---

## WORKFLOW

### 1. Define Variation Strategy (5-10 minutes)

Choose strategy based on **what you want to explore**:

| Strategy | What Changes | Use When | Example |
|----------|-------------|----------|---------|
| **A) Seed Variations** | Random composition/layout | Explore different interpretations of same concept | Same prompt, seeds: 42, 123, 999, 1337 |
| **B) Prompt Variations** | Scene elements/description | Test different angles/messaging | "entrepreneur working" vs "entrepreneur celebrating success" |
| **C) Temperature Variations** | Creative randomness (0.6-1.3) | Find sweet spot between boring and chaotic | Same prompt, temp: 0.7, 0.9, 1.1, 1.3 |
| **D) Style/Preset Variations** | Visual style (photographic/cinematic/anime) | Compare artistic approaches | Same prompt, presets: photographic, cinematic, 3d_model |

**Decision Tree:**

```
Do you want to...
├─ Explore different COMPOSITIONS of same idea?
│  └─ Use SEED variations (easiest)
├─ Test different MESSAGES/ANGLES?
│  └─ Use PROMPT variations (most control)
├─ Find optimal CREATIVITY level?
│  └─ Use TEMPERATURE variations (technical)
└─ Compare different VISUAL STYLES?
   └─ Use PRESET variations (artistic)
```

### 2. Cost Analysis & Planning

**Gemini API Pricing (2026):**
- Text-to-image: ~R$0.01 per generation
- img2img: ~R$0.01 per generation (same cost)

**Batch Cost Calculation:**
```
Cost = variations × (generation + optional_upscale)
```

**Examples:**
- 10 seed variations (no upscale): 10 × R$0.01 = **R$0.10**
- 5 prompt variations + upscale winners (2): (5 × R$0.01) + (2 × R$0.01) = **R$0.07**
- 4 preset variations + full pipeline on winner: (4 × R$0.01) + R$0.02 = **R$0.06**

**Recommended Batch Sizes:**
- Initial exploration: 4-6 variations
- Refinement: 8-10 variations
- Production/final: Select top 1-2, apply full pipeline

### 3. Strategy A: Seed Variations (Composition Exploration)

**When to use:** You have a working prompt, want to see different compositions/layouts.

**How it works:** Same prompt, different seeds = different random interpretations.

#### Step 3A.1: Generate Base Image with Seed Lock
```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "Your validated base prompt here",
        "preset": "photographic",
        "aspect_ratio": "16:9",
        "temperature": 0.8,
        "seed": 42,
        "batch_count": 1
      },
      "outputs": {"image": {"node": 2, "input": "images"}}
    },
    {
      "id": 2,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "variation_seed_42_",
        "images": {"node": 1, "output": "image"}
      }
    }
  ]
}
```

#### Step 3A.2: Generate Variations with Different Seeds
**Seeds to try:** 42, 123, 456, 789, 999, 1337, 2048, 3141, 4242, 5555

Execute each with MCP:
```
for seed in [42, 123, 456, 789, 999]:
  mcp__comfyui__run_workflow(
    workflow: [workflow_with_seed],
    name: f"seed_variation_{seed}",
    sync: false  # Run async for faster batch
  )
```

**OR use batch_count parameter (single API call):**
```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "Your base prompt",
        "seed": -1,  // Random seeds
        "batch_count": 10  // Generate 10 variations in one call
      }
    }
  ]
}
```

**Note:** batch_count generates multiple images in **parallel** but you cannot control individual seeds. Use multiple workflows for seed control.

### 4. Strategy B: Prompt Variations (Message/Angle Testing)

**When to use:** Testing different hooks, emotions, or perspectives of same concept.

**How it works:** Modify prompt elements while keeping structure.

#### Step 4B.1: Create Prompt Matrix

**Base:** "Entrepreneur in modern office"

**Variations:**
1. "Entrepreneur **working focused** in modern office"
2. "Entrepreneur **celebrating success** in modern office"
3. "Entrepreneur **stressed overwhelmed** in modern office"
4. "Entrepreneur **collaborating with team** in modern office"
5. "Entrepreneur **presenting to clients** in modern office"

#### Step 4B.2: Generate Each Variation

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "[variation_1_prompt]",
        "seed": 42,  // Lock seed for fair comparison
        "temperature": 0.8,
        "preset": "photographic"
      }
    },
    {
      "id": 2,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "prompt_var_focused_",
        "images": {"node": 1, "output": "image"}
      }
    }
  ]
}
```

**Batch execution:**
```python
prompts = [
  "Entrepreneur working focused in modern office",
  "Entrepreneur celebrating success in modern office",
  "Entrepreneur stressed overwhelmed in modern office",
  "Entrepreneur collaborating with team in modern office",
  "Entrepreneur presenting to clients in modern office"
]

for idx, prompt in enumerate(prompts):
  workflow = build_workflow(prompt, f"prompt_var_{idx+1}")
  mcp__comfyui__run_workflow(
    workflow: workflow,
    name: f"prompt_variation_{idx+1}",
    sync: false
  )
```

### 5. Strategy C: Temperature Variations (Creativity Tuning)

**When to use:** Finding optimal balance between predictable and creative.

**How it works:** Temperature controls randomness in generation.

#### Step 5C.1: Temperature Scale Guide

| Temperature | Behavior | Best For |
|-------------|----------|----------|
| **0.6** | Very predictable, conservative | Brand assets needing consistency |
| **0.7** | Slight variation, professional | Business/corporate |
| **0.8** | Balanced creativity (default) | General purpose |
| **0.9** | More experimental | Marketing/creative |
| **1.0** | Noticeably creative | Abstract/artistic |
| **1.1** | Bold interpretations | Exploration |
| **1.2** | Very creative, unpredictable | Experimental |
| **1.3+** | Chaotic, may break coherence | Testing limits |

#### Step 5C.2: Generate Temperature Sweep

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "Your base prompt",
        "seed": 42,  // Lock seed
        "temperature": 0.7,  // Vary this: 0.7, 0.8, 0.9, 1.0, 1.1, 1.2
        "preset": "photographic"
      }
    }
  ]
}
```

**Recommended testing sequence:** 0.7 → 0.8 → 0.9 → 1.1 (skip 1.0, test extremes)

### 6. Strategy D: Preset/Style Variations (Visual Style Comparison)

**When to use:** Comparing photographic vs cinematic vs illustrated approaches.

**How it works:** Same prompt, different IFGeminiNode presets.

#### Step 6D.1: Preset Categories

**Available presets** (check full list with `mcp__comfyui__get_capabilities()`):

| Category | Presets | Characteristics |
|----------|---------|-----------------|
| **Photorealistic** | photographic, enhance, photographic_highcontrast | Studio lighting, realistic textures |
| **Cinematic** | cinematic, cinematic_warm, cinematic_cool | Film grain, color grading, dramatic |
| **3D** | 3d_model, 3d_character, isometric | CGI aesthetic, controlled lighting |
| **Illustrated** | digital_art, anime, comic_book | Stylized, hand-drawn feel |
| **Fantasy** | fantasy_art, alien, neonpunk | Surreal, otherworldly |

#### Step 6D.2: Generate Style Comparisons

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "IFGeminiNode",
      "inputs": {
        "prompt": "Your base prompt",
        "seed": 42,
        "temperature": 0.8,
        "preset": "photographic"  // Vary: photographic, cinematic, 3d_model, anime
      }
    }
  ]
}
```

**Recommended comparison set:**
1. `photographic` (baseline)
2. `cinematic` (dramatic alternative)
3. `3d_model` (stylized)
4. `digital_art` (illustrated)

### 7. Execute Batch (Async Strategy)

**For batches of 4+:** Use async execution to run parallel.

#### Step 7.1: Submit All Workflows
```python
task_ids = []

for variation in variations:
  result = mcp__comfyui__run_workflow(
    workflow: variation.workflow,
    name: variation.name,
    sync: false,  # Don't wait
    outputMode: "minimal"  # Reduce overhead
  )
  task_ids.append(result.task_id)
```

#### Step 7.2: Monitor Progress
```python
while True:
  tasks = mcp__comfyui__list_tasks(status: "working")

  if len(tasks) == 0:
    break  # All complete

  print(f"Running: {len(tasks)} | Complete: {len(task_ids) - len(tasks)}")
  wait(5 seconds)
```

#### Step 7.3: Collect Results
```python
results = []

for task_id in task_ids:
  result = mcp__comfyui__get_task_result(taskId: task_id)
  results.append({
    "task_id": task_id,
    "filename": result.output.filename,
    "url": result.output.url
  })
```

### 8. Present Side-by-Side for Selection

**Layout for comparison:**

```
┌─────────────────┬─────────────────┬─────────────────┐
│   Variation 1   │   Variation 2   │   Variation 3   │
│   [Image]       │   [Image]       │   [Image]       │
│   Seed: 42      │   Seed: 123     │   Seed: 456     │
│   Temp: 0.8     │   Temp: 0.8     │   Temp: 0.8     │
└─────────────────┴─────────────────┴─────────────────┘
```

**Use MCP get_image to display:**
```python
for result in results:
  image_data = mcp__comfyui__get_image(
    filename: result.filename,
    folder_type: "output"
  )
  display_image(image_data, caption=result.task_id)
```

### 9. Apply Full Pipeline to Winners (Post-Selection)

After selecting top 1-3 variations:

#### Step 9.1: Upscale Selected Images
```json
{
  "nodes": [
    {
      "id": 1,
      "type": "LoadImage",
      "inputs": {"image": "variation_seed_42_00001_.png"}
    },
    {
      "id": 2,
      "type": "UpscaleModelLoader",
      "inputs": {"model_name": "4x-UltraSharp.pth"}
    },
    {
      "id": 3,
      "type": "ImageUpscaleWithModel",
      "inputs": {
        "upscale_model": {"node": 2, "output": "model"},
        "image": {"node": 1, "output": "image"}
      }
    },
    {
      "id": 4,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "winner_upscaled_",
        "images": {"node": 3, "output": "image"}
      }
    }
  ]
}
```

#### Step 9.2: Face Restoration (If Has People)
```json
{
  "nodes": [
    {
      "id": 1,
      "type": "LoadImage",
      "inputs": {"image": "winner_upscaled_00001_.png"}
    },
    {
      "id": 2,
      "type": "CodeFormer",
      "inputs": {
        "fidelity": 0.7,
        "image": {"node": 1, "output": "image"}
      }
    },
    {
      "id": 3,
      "type": "SaveImage",
      "inputs": {
        "filename_prefix": "winner_final_",
        "images": {"node": 2, "output": "image"}
      }
    }
  ]
}
```

### 10. Save Winning Workflow as Template

```python
mcp__comfyui__save_template(
  name: "winning_variation_seed_42",
  workflow: winning_workflow_json,
  description: f"Best variation from batch - Seed 42, Temp 0.8, {preset}",
  tags: ["winner", "batch", f"seed_42", f"{strategy_type}"]
)
```

### 11. Document Selection in Notes

```python
mcp__comfyui__save_note(
  title: f"Batch Variations - {project_name}",
  content: f"""
    Date: {date}
    Strategy: {strategy_type}
    Total Variations: {total_count}

    Parameters Tested:
    {parameters_list}

    Winner:
    - Seed: {winner_seed}
    - Temperature: {winner_temp}
    - Preset: {winner_preset}
    - Filename: {winner_filename}
    - Reason: {selection_reason}

    Runner-ups:
    {runner_ups_list}

    Template Saved: {template_name}
  """
)
```

---

## BATCH SIZE RECOMMENDATIONS

| Purpose | Variations | Strategy |
|---------|-----------|----------|
| **Quick exploration** | 3-4 | Seed variations |
| **Message testing** | 5-7 | Prompt variations |
| **Style comparison** | 4-6 | Preset variations |
| **Production selection** | 8-12 | Mixed strategies |
| **A/B test candidates** | 2 | Best 2 from larger batch |

**Rule of thumb:** Generate 2-3x more variations than you need. Expect 30-40% to be unusable.

---

## SELECTION PROTOCOL

### Evaluation Criteria Checklist

Evaluate each variation on:
- [ ] **Composition** - Balanced, clear focal point?
- [ ] **Brand alignment** - Colors/mood match brand?
- [ ] **Message clarity** - Communicates intended message?
- [ ] **Technical quality** - No artifacts, distortions, blurriness?
- [ ] **Uniqueness** - Stands out from generic stock?
- [ ] **Usability** - Works at target size/format?

### Scoring System (1-5)

| Score | Meaning |
|-------|---------|
| **5** | Perfect - use as-is |
| **4** | Great - minor tweaks needed |
| **3** | Good - usable as fallback |
| **2** | Meh - specific issues |
| **1** | Bad - discard |

### Selection Decision Tree

```
Is variation score ≥4?
├─ YES
│  └─ Is it better than current winner?
│     ├─ YES → Mark as new winner
│     └─ NO → Keep as runner-up
└─ NO
   └─ Score 3? → Keep as backup
      └─ Score <3? → Discard
```

---

## NAMING CONVENTION

**Format:** `[project]_[strategy]_[identifier]_[version]`

**Examples:**
- `starter_hero_seed_42_v1`
- `pro_banner_prompt_celebrating_v2`
- `enterprise_social_temp_09_v1`
- `brand_asset_preset_cinematic_v3`

**MCP name generation:**
```python
mcp__comfyui__name_generation(
  context: "Brand asset for [product] product",
  type: "variation",
  attributes: {"strategy": "seed", "value": 42}
)
```

---

## TROUBLESHOOTING

### Issue 1: All Variations Look Too Similar
**Symptom:** Despite different seeds/params, outputs are nearly identical.

**Diagnosis:**
- Prompt is too constraining ("exactly like stock photo #12345")
- Temperature too low (<0.6)
- Seed range too narrow (42, 43, 44...)

**Solution:**
- Loosen prompt: remove overly specific constraints
- Increase temperature to 0.9-1.1
- Use diverse seeds: 42, 500, 1337, 3141, 9999
- Try different strategy (prompt variations instead of seed)

### Issue 2: All Variations Look Too Different
**Symptom:** Can't compare variations because they're completely unrelated.

**Diagnosis:**
- Temperature too high (>1.2)
- Prompt too vague/open-ended
- No seed lock (seed=-1 on all)

**Solution:**
- Reduce temperature to 0.7-0.8
- Make prompt more specific
- Lock seed when testing other parameters
- Keep base concept consistent across variations

### Issue 3: Batch Taking Too Long
**Symptom:** Waiting 10+ minutes for results.

**Diagnosis:**
- Running sync=true sequentially
- ComfyUI server overloaded
- Too many variations in single batch (>20)

**Solution:**
- Use sync=false for parallel execution
- Check server status: `mcp__comfyui__get_status()`
- Split into smaller batches (5-10 per batch)
- Monitor with `list_tasks` to see if stuck

### Issue 4: Batch Failing Midway
**Symptom:** Some variations complete, others timeout/error.

**Diagnosis:**
- Gemini API rate limit hit
- ComfyUI memory exhausted
- Invalid workflow in one variation

**Solution:**
- Add delays between submissions (5-10 seconds)
- Validate all workflows before batch: `mcp__comfyui__validate_workflow`
- Reduce batch size
- Check error messages in failed tasks: `get_task_result()`

### Issue 5: Can't Decide Between Variations
**Symptom:** All look good, analysis paralysis.

**Diagnosis:**
- Too many options (>10)
- Unclear selection criteria
- All are actually similar quality

**Solution:**
- Eliminate bottom 50% immediately (score <3)
- Apply specific criteria: "Which best matches brand?" "Which converts better?"
- Ask external feedback (colleague, test audience)
- When in doubt, pick 2-3 and A/B test in production

### Issue 6: Winner Needs Different Processing
**Symptom:** Selected variation needs upscale but others don't.

**Diagnosis:**
- Didn't plan post-processing strategy
- Selected variation has faces (needs CodeFormer)

**Solution:**
- Generate variations at base resolution first
- Select winner
- Apply custom pipeline to winner only (saves cost/time)
- Use conditional workflows: if has_faces → CodeFormer → Upscale

### Issue 7: Can't Retrieve Variation Images
**Symptom:** `get_image` returns errors for some variations.

**Diagnosis:**
- Filenames don't match (ComfyUI added counter)
- Images in wrong folder (temp vs output)

**Solution:**
- Check actual filename in ComfyUI/output/
- Use `get_task_result()` to get correct filename from task
- Search by prefix: `ls ComfyUI/output/ | grep variation_seed_`

### Issue 8: Variations Not Different Enough for A/B Test
**Symptom:** Need 2 distinct candidates, all variations too similar.

**Diagnosis:**
- Used same strategy (seed) for all
- Need to test fundamentally different approaches

**Solution:**
- Use mixed strategies:
  - Batch 1: Original concept (photographic preset)
  - Batch 2: Alternative concept (cinematic preset)
  - Batch 3: Completely different angle (prompt variation)
- Pick best from each batch for A/B test

---

## CROSS-REFERENCES

**Knowledge Base:**
- KB-capabilities.md → IFGeminiNode parameters (batch_count, seed, temperature, presets)
- KB-prompting-heuristics.md → Prompt variation techniques
- KB-workflow-patterns.md → Batch workflow patterns & async execution
- KB-system-inventory.md → Section 4 (Presets), Section 5 (Seeds)

**Related Tasks:**
- `generate-image.md` → Base generation workflow
- `upscale-image.md` → Post-selection upscaling
- `create-brand-asset.md` → Brand-specific variation strategies
- `create-banner.md` → Batch variations for ad testing

**MCP Documentation:**
- `run_workflow` → sync parameter for batch control
- `list_tasks` → Monitor running batch
- `get_task_result` → Retrieve individual results
- `save_template` → Preserve winning workflows

---

## SUCCESS CRITERIA

Batch variation session is complete when:
- [ ] 4+ variations generated per strategy
- [ ] All variations evaluated with scoring system
- [ ] Winner selected with documented reasoning
- [ ] Winner processed with full pipeline (upscale/restore if needed)
- [ ] Winning workflow saved as reusable template
- [ ] Runner-ups identified as backups
- [ ] Selection decision logged in MCP notes
- [ ] Cost tracked and within budget

---

## ADVANCED: MULTI-STRATEGY BATCH

Combine multiple strategies for maximum exploration:

```python
# Stage 1: Seed exploration (find best composition)
seed_batch = [42, 123, 456, 789, 999]
winner_seed = run_batch(base_prompt, vary_seeds=seed_batch)

# Stage 2: Temperature tuning (optimize creativity)
temp_batch = [0.7, 0.8, 0.9, 1.0, 1.1]
winner_temp = run_batch(base_prompt, seed=winner_seed, vary_temps=temp_batch)

# Stage 3: Preset comparison (style options)
preset_batch = ["photographic", "cinematic", "3d_model"]
winner_preset = run_batch(base_prompt, seed=winner_seed, temp=winner_temp, vary_presets=preset_batch)

# Final: Generate ultimate winner with optimized parameters
final_image = generate(base_prompt, seed=winner_seed, temp=winner_temp, preset=winner_preset)
```

**Total cost:** (5 + 5 + 3 + 1) = 14 generations × R$0.01 = **R$0.14**

---

*ComfyUI Ops Task - Batch Variations*
*Version: 1.0 | Updated: 2026-02-06*
*KB: capabilities, prompting-heuristics, workflow-patterns, system-inventory*
