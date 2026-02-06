# Setup Model - Download e Configuração de Modelos
## Instala e configura modelos para pipeline de pós-processamento e capacidades avançadas

---
type: TASK
squad: comfyui-ops
priority: HIGH
created: 2026-02-06
version: 2.0.0
requires: [KB-system-inventory.md, KB-capabilities.md]
agents: [forge]
---

## PRE-FLIGHT

```
ANTES DE QUALQUER INSTALAÇÃO:
1. mcp__comfyui__get_status → Verificar ComfyUI rodando
2. mcp__comfyui__list_models (type: all) → Inventário atual
3. Consultar KB-system-inventory.md Seções 3, 4, 6 (inventário completo)
4. Verificar espaço em disco (modelos variam de 64MB a 12GB)
5. PARAR ComfyUI antes de instalar (ideal, não obrigatório)
```

**IMPORTANTE:** ComfyUI neste setup usa **Gemini API para geração de imagens** (não checkpoints locais). Modelos aqui são para:
1. **Pós-processamento** (upscale, face restore) — ALTA PRIORIDADE
2. **Capacidades futuras** (geração local, ControlNet) — BAIXA PRIORIDADE

---

## WORKFLOW COMPLETO

### 1. Diagnosticar Estado Atual

**Via MCP:**
```
mcp__comfyui__list_models
  Params: { type: "all" }
  Output: Lista de todos os modelos instalados por categoria
```

**Via MCP (específico):**
```
mcp__comfyui__list_models (type: "checkpoints")  → SD/SDXL/Flux
mcp__comfyui__list_models (type: "upscale_models")  → ESRGAN, UltraSharp
mcp__comfyui__list_models (type: "facerestore_models")  → CodeFormer, GFPGAN
mcp__comfyui__list_models (type: "loras")  → LoRAs
mcp__comfyui__list_models (type: "controlnet")  → ControlNet models
mcp__comfyui__list_models (type: "vae")  → VAE models
```

**Verificar contra inventário:**
- Consultar `KB-system-inventory.md` Seção 3 (Modelos Instalados)
- Comparar o que está instalado vs o que falta

### 2. Perguntar ao Usuário (Elicitation)

| Pergunta | Opções | Por Que Importa |
|----------|--------|-----------------|
| **Qual o uso principal?** | Imagens gerais / Banners / Thumbnails / Vídeo / Produtos | Define prioridade de instalação |
| **Prioridade:** | Velocidade / Qualidade / Balance | Afeta escolha de modelo (FP8 vs full precision) |
| **Hardware:** | GPU VRAM (ex: 8GB, 12GB, 16GB+) | Limita modelos viáveis |
| **Objetivo imediato:** | Melhorar qualidade das gerações Gemini / Habilitar geração local / Ambos | Define se instala post-processing ou checkpoints |

### 3. Recomendar Modelo (Decision Matrix)

#### Perfil 1: "Melhorar Outputs Gemini" (Post-Processing)

**PRIORIDADE ALTA** — Funciona com setup atual (Gemini API):

| Modelo | Tipo | VRAM | Benefício | Prioridade |
|--------|------|------|-----------|-----------|
| **ComfyUI-ProPost** | Custom Node | N/A | Color grading com LUT (comercial) | ALTA |
| **GFPGAN** | facerestore | ~300MB | Face restoration alternativa | ALTA |
| **4x-Nomos8kSC.pth** | upscale | ~64MB | Upscale otimizado para fotos | MÉDIA |

**Já Instalados:**
- RealESRGAN_x4plus.pth (upscale)
- 4x-UltraSharp.pth (upscale)
- codeformer.pth (face restore)

#### Perfil 2: "Habilitar Geração Local" (Checkpoints)

**PRIORIDADE BAIXA** — Para casos onde Gemini API não é ideal:

| Modelo | Arquivo | VRAM | Velocidade | Melhor Para |
|--------|---------|------|-----------|-------------|
| **Flux Schnell FP8** | flux1-schnell-fp8.safetensors | 8GB+ | 4 steps | Iteração rápida, começo |
| **Flux Dev FP8** | flux1-dev-fp8.safetensors | 10GB+ | 20 steps | Qualidade máxima, final |
| **SDXL Base 1.0** | sd_xl_base_1.0.safetensors | 8GB+ | 25 steps | Versátil, LoRAs |
| **SD 1.5** | v1-5-pruned.safetensors | 4GB+ | 20 steps | Hardware limitado |
| **SD3.5 Large FP8** | sd3.5_large_fp8_scaled.safetensors | 10GB+ | 28 steps | Melhor geração de texto |
| **SVD XT** | svd_xt.safetensors | 12GB+ | 25 steps | Vídeo |

#### Perfil 3: "Controle Avançado" (ControlNet, IP-Adapter)

**PRIORIDADE BAIXA** — Para casos específicos:

| Componente | Requer | Benefício |
|------------|--------|-----------|
| **ComfyUI-Impact-Pack** | Custom node + YOLOv8 model | FaceDetailer, SAM, detectors |
| **ControlNet v1.1** | Checkpoint + ControlNet models | Controle de pose, depth, edge |
| **IP-Adapter** | Custom node + weights | Style transfer avançado |

### 4. Obter URL de Download (Via MCP)

```
mcp__comfyui__get_download_url
  Params: { modelName: "flux1-schnell-fp8" }
  Output: URL de download + instruções
```

**Fontes de modelos:**
- HuggingFace (oficial): https://huggingface.co/
- CivitAI (community): https://civitai.com/
- OpenModelDB (upscalers): https://openmodeldb.info/

### 5. Instruir Download (Por Tipo)

#### A) Custom Nodes (ComfyUI-Manager)

**Via Interface Web:**
1. Acessar http://127.0.0.1:8188
2. Clicar no botão "Manager" (canto superior direito)
3. "Install Custom Nodes"
4. Buscar: "ComfyUI-ProPost" ou "ComfyUI-Impact-Pack"
5. Clicar "Install"
6. Reiniciar ComfyUI

**Verificação:**
```
mcp__comfyui__list_nodes
  Params: {}
  Output: Verificar se nodes do pacote aparecem
```

#### B) Modelos de Upscale

**Flux Schnell FP8 (Recomendado para Começar):**
- URL: https://huggingface.co/Comfy-Org/flux1-schnell/tree/main
- Arquivo: `flux1-schnell-fp8.safetensors` (~11GB)
- Destino: `D:\ComfyUI\models\checkpoints\`
- Settings: 4 steps, CFG 1, 1024x1024, sampler euler, scheduler simple
- Prompting: Natural language, sem negative prompt

**Flux Dev FP8:**
- URL: https://huggingface.co/Comfy-Org/flux1-dev/tree/main
- Arquivo: `flux1-dev-fp8.safetensors` (~11GB)
- Destino: `D:\ComfyUI\models\checkpoints\`
- Settings: 20 steps, CFG 3.5, 1024x1024

**SDXL Base 1.0:**
- URL: https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0
- Arquivo: `sd_xl_base_1.0.safetensors` (~6.5GB)
- Destino: `D:\ComfyUI\models\checkpoints\`
- Settings: 25 steps, CFG 7, 1024x1024

**SD3.5 Large FP8:**
- URL: https://huggingface.co/Comfy-Org/stable-diffusion-3.5-fp8
- Arquivo: `sd3.5_large_fp8_scaled.safetensors` (~10GB)
- Destino: `D:\ComfyUI\models\checkpoints\`
- Settings: 28 steps, CFG 4.5, 1024x1024

**4x-Nomos8kSC (Upscale para fotos):**
- URL: https://openmodeldb.info/ (buscar "Nomos8k")
- Arquivo: `4x-Nomos8kSC.pth` (~64MB)
- Destino: `D:\ComfyUI\models\upscale_models\`

**GFPGAN (Face Restore):**
- URL: https://github.com/TencentARC/GFPGAN/releases
- Arquivo: `GFPGANv1.4.pth` (~332MB)
- Destino: `D:\ComfyUI\models\facerestore_models\`

#### C) Download via Comando

**Windows (PowerShell):**
```powershell
# Exemplo: Flux Schnell FP8
Invoke-WebRequest -Uri "https://huggingface.co/Comfy-Org/flux1-schnell/resolve/main/flux1-schnell-fp8.safetensors" `
  -OutFile "D:\ComfyUI\models\checkpoints\flux1-schnell-fp8.safetensors"
```

**Linux/Mac:**
```bash
# Exemplo: Flux Schnell FP8
wget -O "D:\ComfyUI\models\checkpoints\flux1-schnell-fp8.safetensors" \
  "https://huggingface.co/Comfy-Org/flux1-schnell/resolve/main/flux1-schnell-fp8.safetensors"
```

**Usando cURL (cross-platform):**
```bash
curl -L -o "D:\ComfyUI\models\checkpoints\flux1-schnell-fp8.safetensors" \
  "https://huggingface.co/Comfy-Org/flux1-schnell/resolve/main/flux1-schnell-fp8.safetensors"
```

### 6. Verificar Instalação

**Via MCP:**
```
mcp__comfyui__list_models
  Params: { type: "checkpoints" }  # Ou o tipo relevante
  Output: Modelo deve aparecer na lista
```

**Via REST API (fallback):**
```bash
curl -s http://127.0.0.1:8188/api/system_stats
```

**Visual (Interface Web):**
- Acessar ComfyUI interface
- Criar node "CheckpointLoaderSimple" (ou equivalente)
- Verificar se modelo aparece no dropdown

### 7. Teste de Geração

**Obter workflow recomendado:**
```
mcp__comfyui__recommend_workflow
  Params: { modelName: "flux1-schnell-fp8" }
  Output: Workflow JSON otimizado para o modelo
```

**Prompts de teste por modelo:**

| Modelo | Prompt de Teste |
|--------|-----------------|
| **Flux** | "A beautiful mountain landscape with a crystal clear lake at golden hour, dramatic clouds" |
| **SDXL** | "A beautiful mountain landscape with a crystal clear lake at golden hour, dramatic clouds, photorealistic, 8k" |
| **SD 1.5** | "beautiful landscape, mountains, lake, golden hour, masterpiece, best quality, highly detailed" |

**Executar teste:**
```
mcp__comfyui__run_workflow
  Params: {
    workflow: {workflow recomendado},
    name: "test_[modelo]",
    sync: true,
    imageFormat: "jpeg",
    imageQuality: 90
  }
```

**Verificar resultado:**
- Se imagem gerada = SUCCESS
- Se placeholder cinza = FAILURE (verificar logs)
- Se erro de VRAM = Modelo incompatível com hardware

### 8. Salvar Configuração

**Via MCP Note:**
```
mcp__comfyui__save_note
  Params: {
    topic: "model-setup",
    content: "Modelo [nome] instalado e testado com sucesso. Path: [path]. Settings: steps [X], CFG [Y], sampler [Z], scheduler [W], resolução [AxB]. Hardware: [GPU], VRAM OK.",
    tags: ["setup", "modelo", "configuração", "tested"]
  }
```

**Salvar workflow de teste como template:**
```
mcp__comfyui__save_template
  Params: {
    name: "test_[modelo]_basic",
    workflow: {JSON do teste},
    description: "Workflow de teste básico para [modelo]. Comprovado funcional."
  }
```

### 9. ATUALIZAR KB-SYSTEM-INVENTORY.md (CRÍTICO)

**REGRA OBRIGATÓRIA:** Após QUALQUER instalação, atualizar o inventário:

```markdown
### [Tipo de Modelo] (D:\ComfyUI\models\[tipo]\)

| Modelo | Arquivo | Tamanho | Uso |
|--------|---------|---------|-----|
| [Nome] | [arquivo.pth] | ~[X]GB | [Descrição] |
```

**Mover item da seção "O Que Falta" para "Instalado".**

**Atualizar seção "Capacidades Atuais vs Desejadas":**
```markdown
| Capacidade | Status | Qualidade |
|------------|--------|-----------|
| [Nova capacidade] | OPERACIONAL | 8/10 |
```

---

## MAPEAMENTO DE PATHS

| Tipo | Path Completo |
|------|---------------|
| **Checkpoints** | D:\ComfyUI\models\checkpoints\ |
| **LoRAs** | D:\ComfyUI\models\loras\ |
| **Upscale** | D:\ComfyUI\models\upscale_models\ |
| **Face Restore** | D:\ComfyUI\models\facerestore_models\ |
| **ControlNet** | D:\ComfyUI\models\controlnet\ |
| **VAE** | D:\ComfyUI\models\vae\ |
| **CLIP** | D:\ComfyUI\models\clip\ |
| **CLIP Vision** | D:\ComfyUI\models\clip_vision\ |
| **Custom Nodes** | D:\ComfyUI\custom_nodes\ |

---

## MODELOS AUXILIARES (Instalar Depois do Core)

| Tipo | Modelo | Para Que | Prioridade |
|------|--------|----------|-----------|
| **VAE** | sdxl_vae.safetensors | Melhorar cores SDXL | MÉDIA |
| **LoRA** | Vários (CivitAI) | Estilos específicos (anime, realistic, etc) | BAIXA |
| **ControlNet** | v1.1 models | Controle de pose, depth, edge | BAIXA |
| **CLIP Vision** | clip_vision_g.safetensors | Style transfer avançado | BAIXA |
| **SVD** | svd_xt.safetensors | Gerar vídeo (image-to-video) | BAIXA |
| **AnimateDiff** | mm_sd_v15_v2.ckpt | Vídeo (text-to-video) | NÃO PRIORIZAR |

**Como baixar auxiliares:**
```
mcp__comfyui__get_download_url
  Params: { modelName: "sdxl_vae" }

mcp__comfyui__get_model_guide
  Params: { modelType: "vae" }
```

---

## PRIORIDADES DE INSTALAÇÃO (Atualizado)

### ALTA PRIORIDADE (Melhoram workflows existentes)

| Item | Tipo | Tamanho | Benefício | ETA Install |
|------|------|---------|-----------|-------------|
| **ComfyUI-ProPost** | Custom Node | <10MB | Color grading comercial (LUT) | 5min |
| **GFPGAN** | Model | ~330MB | Face restore alternativa | 10min |
| **4x-Nomos8kSC** | Model | ~64MB | Upscale otimizado para fotos | 5min |

### MÉDIA PRIORIDADE (Expandem capacidades)

| Item | Tipo | Tamanho | Benefício | ETA Install |
|------|------|---------|-----------|-------------|
| **ComfyUI-Impact-Pack** | Custom Node + Models | ~500MB | FaceDetailer, SAM, YOLOv8 | 15min |
| **Flux Schnell FP8** | Checkpoint | ~11GB | Geração local rápida | 30-60min |

### BAIXA PRIORIDADE (Casos específicos)

| Item | Tipo | Tamanho | Benefício | ETA Install |
|------|------|---------|-----------|-------------|
| **SD 1.5** | Checkpoint | ~2-4GB | Inpainting, img2img local | 20-40min |
| **SDXL Base** | Checkpoint | ~6.5GB | Alta qualidade local | 30-60min |
| **Flux Dev FP8** | Checkpoint | ~11GB | Geração state-of-art | 30-60min |
| **ControlNet models** | Models | ~1.4GB/cada | Pose, depth, edge control | 15min/cada |
| **IP-Adapter** | Node + Model | ~1GB | Style transfer avançado | 20min |

### NÃO PRIORIZAR (Fora do escopo atual)

| Item | Motivo |
|------|--------|
| Stable Diffusion local (quando Gemini funciona) | Gemini API supre necessidade |
| AnimateDiff | Vídeo não é prioridade |
| ComfyUI-VideoHelperSuite | Vídeo não é prioridade |
| Audio models | Fora do escopo |

---

## TROUBLESHOOTING

| Problema | Causa Provável | Solução |
|----------|----------------|---------|
| **Modelo não aparece na lista** | ComfyUI não reiniciou | Reiniciar ComfyUI completamente |
| **Out of memory (VRAM)** | Modelo muito grande para GPU | Usar versão FP8 ou modelo menor (SD 1.5) |
| **Geração muito lenta** | Modelo pesado + DirectML | Usar Flux Schnell (4 steps) ou reduzir resolução |
| **Cores erradas/lavadas** | Falta VAE específico | Baixar VAE do modelo (ex: sdxl_vae.safetensors) |
| **Workflow não aceita modelo** | Template incompatível | Verificar se template é para o tipo correto (SD 1.5 vs SDXL vs Flux) |
| **Download interrompido** | Arquivo corrompido | Deletar arquivo parcial e retentar download |
| **ComfyUI-Manager não encontra node** | Catálogo desatualizado | Manager → "Update All" → Retry |
| **Modelo existe mas não carrega** | Arquivo corrompido | Re-download, verificar hash SHA256 se disponível |
| **Erro "safetensors" | DirectML incompatibility | Alguns modelos requerem CUDA. Verificar compatibilidade. |
| **Path errado** | Arquivo no diretório errado | Consultar seção "Mapeamento de Paths" acima |

---

## HARDWARE CONSIDERATIONS

### DirectML (AMD/Intel)

**Modelos compatíveis:**
- Flux FP8 ✅
- SDXL ✅
- SD 1.5 ✅
- Upscalers ✅
- Face Restore ✅

**Limitações:**
- Sem suporte CUDA (alguns custom nodes podem falhar)
- Performance inferior a NVIDIA equivalente (~30-50% mais lento)
- Alguns modelos requerem ajustes de settings

### VRAM Requirements

| VRAM | Modelos Viáveis |
|------|----------------|
| **4-6GB** | SD 1.5, Upscalers, Face Restore |
| **8GB** | SDXL, Flux Schnell FP8, todos upscalers |
| **10-12GB** | Flux Dev FP8, SD3.5, múltiplos modelos simultâneos |
| **16GB+** | Flux full precision, SVD, pipelines complexos |

**Optimizações para low VRAM:**
- Usar modelos FP8 (metade do tamanho FP16)
- Reduzir batch size para 1
- Reduzir resolução (512×512 ou 768×768)
- Habilitar "lowvram" flag no startup

---

## VERIFICAÇÃO PÓS-INSTALAÇÃO (Checklist)

```
[ ] Modelo aparece em mcp__comfyui__list_models
[ ] Teste de geração bem-sucedido (não placeholder cinza)
[ ] Workflow salvo como template
[ ] Note criado com settings e confirmação
[ ] KB-system-inventory.md ATUALIZADO
[ ] ComfyUI reiniciado (se custom node)
```

---

## CROSS-REFERENCES

### Knowledge Bases
- **KB-system-inventory.md** Seção 3: Modelos Instalados (SEMPRE ATUALIZAR)
- **KB-system-inventory.md** Seção 4: Diretórios de Modelos
- **KB-system-inventory.md** Seção 6: Prioridades de Instalação
- **KB-system-inventory.md** Seção 10: Checklist Pre-Operação
- **KB-capabilities.md** Seção 4: Modelos Suportados (Gemini)

### Related Tasks
- `generate-image.md` → Usar modelos após instalação
- `upscale-image.md` → Usar upscalers instalados
- `inpaint-edit.md` → Requer checkpoint local (SD 1.5 ou SDXL)

---

## DELEGAÇÃO PARA SPECIALIST

### Quando Delegar

| Situação | Delegar Para | Motivo |
|----------|-------------|--------|
| Workflow complexo pós-instalação | @forge (Workflow Engineer) | Construir pipeline com novo modelo |
| Teste não funciona | @iris (Quality Inspector) | Validar setup e diagnosticar |

---

*Setup Model Task v2.0.0 | ComfyUI Ops Squad*
*Post-processing priority | Hardware-aware recommendations | Inventory integration*
*Criado: 2026-02-06 | Quality: 9/10*
