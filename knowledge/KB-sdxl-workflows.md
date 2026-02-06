# KB-sdxl-workflows.md - SDXL Workflows & Hybrid Pipeline
# ComfyUI Ops Squad - Knowledge Base
# Version: 1.0.0
# Updated: 2026-02-06
# Sources: 7 deep research agents (520+ web sources analyzed)

---

## COMO USAR ESTE DOCUMENTO

1. **txt2img puro:** Seção 2 (Workflow SDXL Básico)
2. **Pipeline híbrido Gemini+SD:** Seção 3 (Workflow Híbrido)
3. **Sem face swap:** Seção 4 (Workflow Simplificado)
4. **Settings reference:** Seção 5 (Quick Reference Card)

---

## 1. DECISION MATRIX

### Quando Usar Cada Pipeline

| Cenário | Pipeline | Motivo |
|---------|----------|--------|
| Ideação rápida / drafts | Gemini-only | Velocidade |
| Social media (texto pesado) | Gemini-only | Textura não importa em small sizes |
| **Hero images para sales pages** | **Híbrido** | Qualidade máxima necessária |
| **Portraits / headshots** | **Híbrido ou SDXL puro** | Textura de pele é crítica |
| **Product photography** | **Híbrido** | Efeitos de lente e profundidade |
| Diagramas / infográficos | Gemini-only | SD degradaria clareza |
| Batch (10+ imagens) | Gemini-only | Custo de tempo do híbrido |
| **3-5 assets hero finais** | **Híbrido** | Vale os 30s extras por imagem |
| Controle fino de estilo/LoRA | SDXL puro | API não suporta LoRAs |

### Tempo Estimado por Pipeline

| Pipeline | Tempo/imagem | Qualidade |
|----------|-------------|-----------|
| Gemini-only | ~5-15s | Composição excelente, textura média |
| Gemini + SDXL refinement | ~30-65s | Composição + textura |
| SDXL puro (DirectML) | ~2-5min | Controle total, mais lento |

---

## 2. WORKFLOW SDXL BÁSICO (txt2img + LoRA + ReActor + Upscale)

### Diagrama de Fluxo

```
[CheckpointLoader] ─MODEL─> [LoRA #1] ─MODEL─> [LoRA #2] ─MODEL─> [KSampler]
                   ─CLIP──> [LoRA #1] ─CLIP──> [LoRA #2] ─CLIP──> [CLIPTextEncode+]
                   ─VAE─────────────────────────────────────────> [VAEDecode]
                                                                      │
[EmptyLatentImage] ─LATENT─> [KSampler] ─LATENT─> [VAEDecode]        │
                                                       │              │
                                                       v              │
[LoadImage (hero)] ─IMAGE─> [ReActorFaceSwap] <────────┘
                                    │
                                    v
[UpscaleModelLoader] ─MODEL─> [ImageUpscaleWithModel]
                                    │
                                    v
                              [SaveImage]
```

### Workflow JSON (API Format)

```json
{
  "1": {
    "class_type": "CheckpointLoaderSimple",
    "inputs": {
      "ckpt_name": "juggernautXL_v9Rdphoto2.safetensors"
    }
  },
  "10": {
    "class_type": "LoraLoader",
    "inputs": {
      "lora_name": "skin_realism_sdxl.safetensors",
      "strength_model": 0.5,
      "strength_clip": 0.5,
      "model": ["1", 0],
      "clip": ["1", 1]
    }
  },
  "11": {
    "class_type": "LoraLoader",
    "inputs": {
      "lora_name": "better_portraits_eyes_v2.safetensors",
      "strength_model": 0.6,
      "strength_clip": 0.6,
      "model": ["10", 0],
      "clip": ["10", 1]
    }
  },
  "20": {
    "class_type": "CLIPTextEncode",
    "inputs": {
      "text": "POSITIVE_PROMPT_HERE",
      "clip": ["11", 1]
    }
  },
  "21": {
    "class_type": "CLIPTextEncode",
    "inputs": {
      "text": "<lora:EasyFix:1> (overfit style:1.0), (worst quality, low quality:1.4), blurry, jpeg artifacts, bad anatomy, deformed, mutated, disfigured, text, watermark, (cartoon, anime, illustration:1.3), 3d render, cgi, plastic skin, airbrushed, extra fingers, mutated hands, poorly drawn hands, poorly drawn face, distorted face",
      "clip": ["11", 1]
    }
  },
  "25": {
    "class_type": "EmptyLatentImage",
    "inputs": {
      "width": 832,
      "height": 1216,
      "batch_size": 1
    }
  },
  "30": {
    "class_type": "KSampler",
    "inputs": {
      "model": ["11", 0],
      "positive": ["20", 0],
      "negative": ["21", 0],
      "latent_image": ["25", 0],
      "seed": 42,
      "steps": 25,
      "cfg": 6.0,
      "sampler_name": "dpmpp_2m_sde",
      "scheduler": "karras",
      "denoise": 1.0
    }
  },
  "50": {
    "class_type": "VAEDecode",
    "inputs": {
      "samples": ["30", 0],
      "vae": ["1", 2]
    }
  },
  "55": {
    "class_type": "LoadImage",
    "inputs": {
      "image": "hero_face.jpg"
    }
  },
  "60": {
    "class_type": "ReActorFaceSwap",
    "inputs": {
      "enabled": true,
      "input_image": ["50", 0],
      "source_image": ["55", 0],
      "swap_model": "inswapper_128.onnx",
      "facedetection": "retinaface_resnet50",
      "face_restore_model": "codeformer.pth",
      "face_restore_visibility": 1.0,
      "codeformer_weight": 0.5,
      "detect_gender_input": "no",
      "detect_gender_source": "no",
      "input_faces_index": "0",
      "source_faces_index": "0",
      "console_log_level": 1
    }
  },
  "65": {
    "class_type": "UpscaleModelLoader",
    "inputs": {
      "model_name": "4x-UltraSharp.pth"
    }
  },
  "70": {
    "class_type": "ImageUpscaleWithModel",
    "inputs": {
      "upscale_model": ["65", 0],
      "image": ["60", 0]
    }
  },
  "80": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["70", 0],
      "filename_prefix": "sdxl_photorealistic"
    }
  }
}
```

---

## 3. WORKFLOW HÍBRIDO (Gemini + SDXL Refinement)

### Conceito

```
PROBLEMA: Gemini = composição excelente, textura "AI sheen"
SOLUÇÃO: Gemini gera base → SDXL+LoRA refina textura → CodeFormer → Upscale

Gemini: Semântica + composição + layout
SDXL+LoRA: Poros, film grain, profundidade de lente, imperfeições naturais
```

### Denoise Guide (CRÍTICO)

| Denoise | Efeito | Quando Usar |
|---------|--------|-------------|
| 0.15-0.20 | Quase invisível | Polish final |
| **0.25-0.35** | **Adiciona textura SEM mudar composição** | **Refinamento primário** |
| 0.40-0.50 | Shift de estilo notável | Transferência de estilo mais forte |
| 0.55+ | Perde composição original | NÃO usar para refinamento |

**RECOMENDADO: 0.30** como ponto de partida.

**Multi-pass (melhor resultado):**
1. Pass 1 com denoise 0.35 -- estabelece textura
2. Pass 2 com denoise 0.20 -- refina sem perturbar

### Diagrama de Fluxo

```
[IFGeminiNode] ─IMAGE─> [VAEEncode] ─LATENT─> [KSampler] ─LATENT─> [VAEDecode]
                              ^                     ^                     │
[CheckpointLoader] ─VAE──────┘                     │                     v
                   ─MODEL─> [LoRA] ─MODEL──────────┤          [CodeFormer]
                   ─CLIP──> [LoRA] ─CLIP──> [CLIP+] ──────────┤          │
                                            [CLIP-] ──────────┘          v
                                                              [UpscaleWithModel]
                                                                        │
                                                                        v
                                                                   [SaveImage]
```

### Workflow JSON (API Format)

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "GEMINI_PROMPT_HERE",
      "operation_mode": "generate_images",
      "model_name": "gemini-3-pro-image-preview",
      "temperature": 1.0,
      "batch_count": 1,
      "aspect_ratio": "3:4",
      "seed": 42,
      "use_random_seed": false,
      "keep_alive": true
    }
  },
  "10": {
    "class_type": "CheckpointLoaderSimple",
    "inputs": {
      "ckpt_name": "juggernautXL_v9Rdphoto2.safetensors"
    }
  },
  "11": {
    "class_type": "LoraLoader",
    "inputs": {
      "lora_name": "TouchOfRealism_v2.safetensors",
      "strength_model": 0.5,
      "strength_clip": 0.5,
      "model": ["10", 0],
      "clip": ["10", 1]
    }
  },
  "12": {
    "class_type": "LoraLoader",
    "inputs": {
      "lora_name": "TouchOfGrain.safetensors",
      "strength_model": 0.4,
      "strength_clip": 0.4,
      "model": ["11", 0],
      "clip": ["11", 1]
    }
  },
  "20": {
    "class_type": "CLIPTextEncode",
    "inputs": {
      "text": "photorealistic, natural skin texture, film grain, shallow depth of field, Kodak Portra 400, 85mm lens, touch-of-realismV2, touch-of-grain",
      "clip": ["12", 1]
    }
  },
  "21": {
    "class_type": "CLIPTextEncode",
    "inputs": {
      "text": "cartoon, illustration, painting, drawing, digital art, 3d render, plastic skin, smooth skin, airbrushed, blurry, deformed",
      "clip": ["12", 1]
    }
  },
  "30": {
    "class_type": "VAEEncode",
    "inputs": {
      "pixels": ["1", 1],
      "vae": ["10", 2]
    }
  },
  "31": {
    "class_type": "KSampler",
    "inputs": {
      "model": ["12", 0],
      "positive": ["20", 0],
      "negative": ["21", 0],
      "latent_image": ["30", 0],
      "seed": 42,
      "steps": 20,
      "cfg": 7.0,
      "sampler_name": "dpmpp_2m",
      "scheduler": "karras",
      "denoise": 0.30
    }
  },
  "32": {
    "class_type": "VAEDecode",
    "inputs": {
      "samples": ["31", 0],
      "vae": ["10", 2]
    }
  },
  "40": {
    "class_type": "FaceRestoreCFCodeFormer",
    "inputs": {
      "image": ["32", 0],
      "fidelity": 0.7
    }
  },
  "50": {
    "class_type": "UpscaleModelLoader",
    "inputs": {
      "model_name": "4x-UltraSharp.pth"
    }
  },
  "51": {
    "class_type": "ImageUpscaleWithModel",
    "inputs": {
      "upscale_model": ["50", 0],
      "image": ["40", 0]
    }
  },
  "60": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["51", 0],
      "filename_prefix": "hybrid_gemini_sd"
    }
  }
}
```

### Pitfalls do Pipeline Híbrido

| Pitfall | Causa | Mitigação |
|---------|-------|-----------|
| Composition drift | Denoise > 0.45 | Manter 0.25-0.35 |
| LoRA overpowering | Strength > 0.7 | Começar em 0.4, subir 0.1 por vez |
| Architecture mismatch | LoRA SD1.5 + SDXL | Todos SDXL |
| Resolution mismatch | Gemini output ≠ SDXL esperado | Resize para 1024x1024 antes de VAEEncode |
| VRAM exhaustion | SDXL 6.5GB + LoRA + VAE | `--lowvram --force-fp16` |
| Color shift | VAE encode/decode cycle | Denoise baixo minimiza efeito |

---

## 4. WORKFLOW SIMPLIFICADO (Sem Face Swap)

Para imagens sem rostos ou quando identidade não importa.

```json
{
  "1": {
    "class_type": "CheckpointLoaderSimple",
    "inputs": {"ckpt_name": "juggernautXL_v9Rdphoto2.safetensors"}
  },
  "10": {
    "class_type": "LoraLoader",
    "inputs": {
      "lora_name": "skin_realism_sdxl.safetensors",
      "strength_model": 0.5, "strength_clip": 0.5,
      "model": ["1", 0], "clip": ["1", 1]
    }
  },
  "20": {
    "class_type": "CLIPTextEncode",
    "inputs": {"text": "POSITIVE_PROMPT", "clip": ["10", 1]}
  },
  "21": {
    "class_type": "CLIPTextEncode",
    "inputs": {"text": "NEGATIVE_PROMPT", "clip": ["10", 1]}
  },
  "25": {
    "class_type": "EmptyLatentImage",
    "inputs": {"width": 1024, "height": 1024, "batch_size": 1}
  },
  "30": {
    "class_type": "KSampler",
    "inputs": {
      "model": ["10", 0], "positive": ["20", 0],
      "negative": ["21", 0], "latent_image": ["25", 0],
      "seed": 42, "steps": 25, "cfg": 6.0,
      "sampler_name": "dpmpp_2m_sde", "scheduler": "karras",
      "denoise": 1.0
    }
  },
  "50": {
    "class_type": "VAEDecode",
    "inputs": {"samples": ["30", 0], "vae": ["1", 2]}
  },
  "65": {
    "class_type": "UpscaleModelLoader",
    "inputs": {"model_name": "4x-UltraSharp.pth"}
  },
  "70": {
    "class_type": "ImageUpscaleWithModel",
    "inputs": {"upscale_model": ["65", 0], "image": ["50", 0]}
  },
  "80": {
    "class_type": "SaveImage",
    "inputs": {"images": ["70", 0], "filename_prefix": "sdxl_simple"}
  }
}
```

---

## 5. QUICK REFERENCE CARD

### KSampler Presets por Modo

| Modo | Steps | CFG | Sampler | Scheduler | Denoise |
|------|:-----:|:---:|---------|:---------:|:-------:|
| **SDXL txt2img** | **25** | **6.0** | `dpmpp_2m_sde` | `karras` | 1.0 |
| SDXL + muitos LoRAs | 30 | 5.0 | `dpmpp_2m_sde` | `karras` | 1.0 |
| **Hybrid refinement** | **20** | **7.0** | `dpmpp_2m` | `karras` | **0.30** |
| Light refinement | 15 | 7.0 | `dpmpp_2m` | `karras` | 0.20 |
| RealVisXL Lightning | 6 | 2.0 | `dpmpp_sde` | `karras` | 1.0 |
| LCM LoRA | 6 | 1.5 | `lcm` | `sgm_uniform` | 1.0 |

### Tempo Estimado (DirectML)

| Estágio | Tempo (1024x1024) |
|---------|:-----------------:|
| Checkpoint + LoRA loading | 10-30s (primeira vez) |
| KSampler (25 steps) | 60-180s |
| KSampler (20 steps, denoise 0.3) | 15-30s |
| VAEDecode | 5-10s |
| ReActor face swap + restore | 5-15s |
| 4x Upscale (UltraSharp) | 15-45s |
| **Pipeline SDXL completo** | **~2-5 min** |
| **Pipeline Híbrido** | **~30-65s** |

### Resolução Após 4x Upscale

| Input | Após 4x | Uso |
|:-----:|:-------:|:----|
| 1024x1024 | 4096x4096 | Print, alta resolução |
| 1344x768 | 5376x3072 | Banner ultra-wide |
| 832x1216 | 3328x4864 | Portrait tall |

---

## 6. PYTHON SCRIPT TEMPLATE

Template baseado no padrão de `D:\ComfyUI\generate_template.py`.

```python
"""
SDXL Photorealistic Pipeline
Checkpoint + LoRA chain + KSampler + VAEDecode + ReActor + 4x Upscale

PREREQUISITES:
  - SDXL checkpoint in D:\ComfyUI\models\checkpoints\
  - LoRA files in D:\ComfyUI\models\loras\
  - ReActor + CodeFormer (já instalado)
  - 4x-UltraSharp.pth (já instalado)
"""
import json, urllib.request, time, sys, random

API = "http://127.0.0.1:8188/prompt"
HISTORY = "http://127.0.0.1:8188/history"

# ============================================================
# CONFIG
# ============================================================
CHECKPOINT = "juggernautXL_v9Rdphoto2.safetensors"
UPSCALER = "4x-UltraSharp.pth"
FACE_MODE = "identity"  # "identity" | "self" | "none"
HERO_IMAGE = "hero_face.jpg"
PROJECT = "sdxl_photorealistic"

LORAS = [
    {"name": "skin_realism_sdxl.safetensors", "sm": 0.5, "sc": 0.5},
    {"name": "better_portraits_eyes_v2.safetensors", "sm": 0.6, "sc": 0.6},
]

NEGATIVE = (
    "<lora:EasyFix:1> (overfit style:1.0), "
    "(worst quality, low quality:1.4), blurry, jpeg artifacts, "
    "bad anatomy, deformed, mutated, disfigured, text, watermark, "
    "(cartoon, anime, illustration:1.3), 3d render, cgi, plastic skin, "
    "airbrushed, extra fingers, mutated hands, distorted face"
)

SCENES = [
    {
        "name": "business_portrait",
        "prompt": (
            "Professional DSLR photograph of a 35-year-old business woman "
            "in a modern office, wearing a navy blazer, natural soft lighting "
            "from large window, shallow depth of field, Canon EOS R5 85mm f/1.4, "
            "natural skin pores and texture visible, warm color temperature"
        ),
        "prefix": f"{PROJECT}/business_portrait",
        "width": 832, "height": 1216,
        "steps": 25, "cfg": 6.0,
    },
]

# ============================================================
# WORKFLOW BUILDER
# ============================================================
def build_workflow(scene):
    n = {}
    n["1"] = {"class_type": "CheckpointLoaderSimple",
              "inputs": {"ckpt_name": CHECKPOINT}}

    prev_m, prev_c = ["1", 0], ["1", 1]
    for i, l in enumerate(LORAS):
        nid = str(10 + i)
        n[nid] = {"class_type": "LoraLoader",
                  "inputs": {"lora_name": l["name"],
                             "strength_model": l["sm"],
                             "strength_clip": l["sc"],
                             "model": prev_m, "clip": prev_c}}
        prev_m, prev_c = [nid, 0], [nid, 1]

    n["20"] = {"class_type": "CLIPTextEncode",
               "inputs": {"text": scene["prompt"], "clip": prev_c}}
    n["21"] = {"class_type": "CLIPTextEncode",
               "inputs": {"text": NEGATIVE, "clip": prev_c}}
    n["25"] = {"class_type": "EmptyLatentImage",
               "inputs": {"width": scene.get("width", 1024),
                          "height": scene.get("height", 1024),
                          "batch_size": 1}}
    n["30"] = {"class_type": "KSampler",
               "inputs": {"model": prev_m, "positive": ["20", 0],
                          "negative": ["21", 0], "latent_image": ["25", 0],
                          "seed": random.randint(0, 2**32-1),
                          "steps": scene.get("steps", 25),
                          "cfg": scene.get("cfg", 6.0),
                          "sampler_name": "dpmpp_2m_sde",
                          "scheduler": "karras", "denoise": 1.0}}
    n["50"] = {"class_type": "VAEDecode",
               "inputs": {"samples": ["30", 0], "vae": ["1", 2]}}

    img_out = ["50", 0]
    if FACE_MODE == "identity":
        n["55"] = {"class_type": "LoadImage",
                   "inputs": {"image": HERO_IMAGE}}
        n["60"] = {"class_type": "ReActorFaceSwap",
                   "inputs": {"enabled": True,
                              "input_image": ["50", 0],
                              "source_image": ["55", 0],
                              "swap_model": "inswapper_128.onnx",
                              "facedetection": "retinaface_resnet50",
                              "face_restore_model": "codeformer.pth",
                              "face_restore_visibility": 1.0,
                              "codeformer_weight": 0.5,
                              "detect_gender_input": "no",
                              "detect_gender_source": "no",
                              "input_faces_index": "0",
                              "source_faces_index": "0",
                              "console_log_level": 1}}
        img_out = ["60", 0]
    elif FACE_MODE == "self":
        n["60"] = {"class_type": "ReActorFaceSwap",
                   "inputs": {"enabled": True,
                              "input_image": ["50", 0],
                              "source_image": ["50", 0],
                              "swap_model": "inswapper_128.onnx",
                              "facedetection": "retinaface_resnet50",
                              "face_restore_model": "codeformer.pth",
                              "face_restore_visibility": 1.0,
                              "codeformer_weight": 0.5,
                              "detect_gender_input": "no",
                              "detect_gender_source": "no",
                              "input_faces_index": "0",
                              "source_faces_index": "0",
                              "console_log_level": 1}}
        img_out = ["60", 0]

    n["65"] = {"class_type": "UpscaleModelLoader",
               "inputs": {"model_name": UPSCALER}}
    n["70"] = {"class_type": "ImageUpscaleWithModel",
               "inputs": {"upscale_model": ["65", 0], "image": img_out}}
    n["80"] = {"class_type": "SaveImage",
               "inputs": {"images": ["70", 0],
                          "filename_prefix": scene["prefix"]}}
    return {"prompt": n}

# ============================================================
# EXECUTION
# ============================================================
def submit(wf, name):
    data = json.dumps(wf).encode("utf-8")
    req = urllib.request.Request(API, data=data,
                                headers={"Content-Type": "application/json"})
    try:
        resp = urllib.request.urlopen(req)
    except Exception as e:
        print(f"[ERROR] ComfyUI not running at {API}")
        sys.exit(1)
    r = json.loads(resp.read())
    pid = r.get("prompt_id", "?")
    print(f"[SUBMITTED] {name} -> {pid}")
    return pid

def wait(pid, name, timeout=600):
    start = time.time()
    while time.time() - start < timeout:
        try:
            resp = urllib.request.urlopen(f"{HISTORY}/{pid}", timeout=5)
            h = json.loads(resp.read())
            if pid in h:
                for nid, nout in h[pid].get("outputs", {}).items():
                    if "images" in nout:
                        for img in nout["images"]:
                            if img.get("type") == "output":
                                fn = img["filename"]
                                sf = img.get("subfolder", "")
                                path = f"{sf}/{fn}" if sf else fn
                                print(f"[DONE] {name} -> {path}")
                                return fn
                st = h[pid].get("status", {})
                if st.get("status_str") == "error":
                    print(f"[ERROR] {name}: {st.get('messages', '?')}")
                    return None
                return None
        except Exception:
            pass
        time.sleep(3)
    print(f"[TIMEOUT] {name}")
    return None

if __name__ == "__main__":
    print(f"PROJECT: {PROJECT} | CHECKPOINT: {CHECKPOINT}")
    print(f"LORAS: {len(LORAS)} | FACE: {FACE_MODE} | SCENES: {len(SCENES)}")
    for i, s in enumerate(SCENES):
        print(f"\n--- {i+1}/{len(SCENES)}: {s['name']} ---")
        wf = build_workflow(s)
        pid = submit(wf, s["name"])
        fn = wait(pid, s["name"], 600)
        status = "OK" if fn else "FAIL"
        print(f"  [{status}] {s['name']}: {fn}")
    print(f"\nOutput: D:\\ComfyUI\\output\\{PROJECT}\\")
```

---

## 7. LoRA USAGE NO COMFYUI

### Path dos LoRAs

```
D:\ComfyUI\models\loras\
```

### Node: LoraLoader

```yaml
class_type: LoraLoader
display_name: "Load LoRA (Model and CLIP)"
inputs:
  model: MODEL  # Do checkpoint ou LoRA anterior
  clip: CLIP    # Do checkpoint ou LoRA anterior
  lora_name: string  # Dropdown dos arquivos em models/loras/
  strength_model: float  # -100.0 a 100.0, default 1.0
  strength_clip: float   # -100.0 a 100.0, default 1.0
outputs:
  0: MODEL  # Modificado
  1: CLIP   # Modificado
```

### Stacking (Chain)

```
Checkpoint ──> LoRA #1 ──> LoRA #2 ──> LoRA #3 ──> KSampler
                                                ──> CLIPTextEncode
```

Cada LoRA recebe MODEL+CLIP do anterior e passa adiante.
Ambos CLIPTextEncode (positivo e negativo) usam o CLIP do **último** LoRA.

### Após Adicionar Novos LoRAs

```
OBRIGATÓRIO: Reiniciar ComfyUI para novos LoRAs aparecerem no dropdown.
```

---

## 8. TROUBLESHOOTING

| Problema | Causa Provável | Solução |
|----------|---------------|---------|
| Imagem toda preta | VAE bug DirectML | Reiniciar ComfyUI |
| "lora key not loaded" | LoRA incompatível com checkpoint | Verificar base model |
| "SHAPE MISMATCH" | LoRA SD1.5 + Checkpoint SDXL | Usar mesmo base |
| OOM crash | VRAM insuficiente | `--lowvram --force-fp16` |
| Faces deformadas | LoRA strength > 0.8 | Reduzir para 0.5 |
| Composição mudou (híbrido) | Denoise > 0.4 | Reduzir para 0.30 |
| Cores estranhas | VAE encode/decode shift | Denoise mais baixo |
| Muito lento | DirectML normal | Aceitar ou usar Lightning |

---

*KB-sdxl-workflows.md - SDXL Workflows & Hybrid Pipeline*
*Consolidado de 7 pesquisas profundas (520+ fontes web)*
*Criado: 2026-02-06*
