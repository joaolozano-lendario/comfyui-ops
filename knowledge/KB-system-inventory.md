# KB-SYSTEM-INVENTORY: Estado Atual do Sistema ComfyUI
## ComfyUI Ops Squad | Knowledge Base
### Fonte: Inventário Real do Sistema | Atualizar Sempre Que Instalar Algo

---
type: KB
priority: P0
squad: comfyui-ops
created: 2026-02-06
version: 1.0.0
source: Inventário real do sistema em D:\ComfyUI
always_load_before: [setup-model, generate-image, upscale-image]
update_policy: "Atualizar este arquivo sempre que instalar/remover node ou modelo"
---

## COMO USAR ESTE DOCUMENTO

1. **Antes de qualquer operação:** Verificar se o node/modelo necessário está instalado
2. **Antes de propor workflow:** Verificar se todos os componentes estão disponíveis
3. **Para instalar algo:** Seguir prioridades na seção 6
4. **Após instalar:** ATUALIZAR ESTE ARQUIVO

---

## 1. AMBIENTE BASE

### ComfyUI Core

| Aspecto | Valor |
|---------|-------|
| **Path** | D:\ComfyUI |
| **Versão** | 0.12.3 |
| **Python** | Via .venv (D:\ComfyUI\.venv) |
| **GPU Backend** | DirectML (AMD/Intel) |
| **Flag de Startup** | `--directml` |
| **URL** | http://127.0.0.1:8188 |
| **Output** | D:\ComfyUI\output\ |
| **Workflows** | D:\ComfyUI\user\default\workflows\ |

### Como Iniciar

```bash
cd D:\ComfyUI
.venv\Scripts\activate
python main.py --directml
```

### Startup Otimizado (SDXL — usar quando checkpoint instalado)

```bash
cd D:\ComfyUI
.venv\Scripts\activate
python main.py --directml --lowvram --use-split-cross-attention --force-fp16 --fp16-unet --cpu-text-encoder --preview-method none
```

> **Sem checkpoint:** usar startup normal. **Com SDXL checkpoint:** usar otimizado para caber em 12GB VRAM.

### API REST

| Endpoint | Método | Uso |
|----------|--------|-----|
| /api/prompt | POST | Enviar workflow para execução |
| /api/history/{id} | GET | Verificar status de execução |
| /api/system_stats | GET | Stats do sistema |
| /api/queue | GET | Fila atual |
| /api/interrupt | POST | Cancelar execução atual |

---

## 2. CUSTOM NODES INSTALADOS

### ComfyUI-IF_Gemini
| Aspecto | Valor |
|---------|-------|
| **Path** | D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini |
| **Nodes** | IFGeminiNode, IFTaskPromptManager, IFPromptCombiner |
| **API Key** | Configurada em .env |
| **Model Patched** | gemini-2.5-flash → gemini-2.5-flash-image |
| **Presets** | 38 banana task presets |
| **Status** | OPERACIONAL |

### ComfyUI-Manager
| Aspecto | Valor |
|---------|-------|
| **Path** | D:\ComfyUI\custom_nodes\ComfyUI-Manager |
| **Função** | Instalação de custom nodes, modelos e atualizações |
| **UI** | Botão "Manager" no menu do ComfyUI |
| **Status** | INSTALADO |

### ComfyUI-ReActor
| Aspecto | Valor |
|---------|-------|
| **Path** | D:\ComfyUI\custom_nodes\ComfyUI-ReActor |
| **Nodes** | ReActorFaceSwap |
| **Dependencias** | insightface, onnxruntime |
| **Modelos** | inswapper_128.onnx, buffalo_l (5 .onnx) |
| **Status** | OPERACIONAL |
| **Uso principal** | Face swap (identity lock) + CodeFormer restore |
| **Trick** | Self-swap (source=input) = CodeFormer only, sem trocar face |

---

## 3. MODELOS INSTALADOS

### Upscale Models (D:\ComfyUI\models\upscale_models\)

| Modelo | Arquivo | Tamanho | Uso |
|--------|---------|---------|-----|
| RealESRGAN 4× Plus | RealESRGAN_x4plus.pth | ~64MB | General purpose upscale |
| 4x UltraSharp | 4x-UltraSharp.pth | ~64MB | Detail-preserving upscale |

### Face Restore Models (D:\ComfyUI\models\facerestore_models\)

| Modelo | Arquivo | Tamanho | Uso |
|--------|---------|---------|-----|
| CodeFormer | codeformer.pth | ~376MB | Face restoration |

### InsightFace / ReActor Models (D:\ComfyUI\models\insightface\)

| Modelo | Arquivo | Uso |
|--------|---------|-----|
| InSwapper | inswapper_128.onnx | Face swap (identity lock) |
| Buffalo_L | models/buffalo_l/*.onnx (5 files) | Face detection/analysis |

### Checkpoints (D:\ComfyUI\models\checkpoints\)
```
VAZIO - Nenhum checkpoint instalado.
```

### LoRAs (D:\ComfyUI\models\loras\)
```
VAZIO - Nenhum LoRA instalado.
```

### ControlNet (D:\ComfyUI\models\controlnet\)
```
VAZIO - Nenhum ControlNet instalado.
```

### VAE (D:\ComfyUI\models\vae\)
```
VAZIO - Nenhum VAE instalado.
```

### CLIP (D:\ComfyUI\models\clip\)
```
VAZIO - Nenhum CLIP instalado.
```

---

## 4. DIRETÓRIOS DE MODELOS DISPONÍVEIS

```
D:\ComfyUI\models\
├── audio_encoders\        # Vazio
├── checkpoints\           # Vazio (SD/SDXL/Flux iriam aqui)
├── clip\                  # Vazio
├── clip_vision\           # Vazio
├── configs\               # Configs padrão
├── controlnet\            # Vazio
├── diffusers\             # Vazio
├── diffusion_models\      # Vazio
├── embeddings\            # Vazio
├── facerestore_models\    # codeformer.pth
├── insightface\           # inswapper_128.onnx + buffalo_l/
├── gligen\                # Vazio
├── hypernetworks\         # Vazio
├── latent_upscale_models\ # Vazio
├── loras\                 # Vazio
├── model_patches\         # Vazio
├── photomaker\            # Vazio
├── style_models\          # Vazio
├── text_encoders\         # Vazio
├── unet\                  # Vazio
├── upscale_models\        # RealESRGAN ✅ + UltraSharp ✅
├── vae\                   # Vazio
└── vae_approx\            # Vazio
```

---

## 5. WORKFLOWS E TEMPLATES SALVOS

### Executable Workflow JSONs (comfyui-ops/workflows/)

Pre-built API-format workflows ready for `mcp__comfyui__run_workflow`:

| Arquivo | Pipeline | Quando Usar |
|---------|----------|-------------|
| txt2img-gemini-basic.json | IFGeminiNode → SaveImage | Ideacao rapida |
| txt2img-gemini-upscale.json | IFGeminiNode → RealESRGAN → SaveImage | Social media, web |
| txt2img-gemini-face.json | IFGeminiNode → CodeFormer → UltraSharp → SaveImage | Portraits |
| img2img-gemini.json | LoadImage → IFGeminiNode → SaveImage | Transformar imagem |
| svg-enhance.json | LoadImage → IFGeminiNode → RealESRGAN → SaveImage | SVG → imagem |
| upscale-only.json | LoadImage → UltraSharp → SaveImage | Upscale sem IA |

> **Primeira vez?** Rode `/comfyui-ops:tasks:config` — wizard interativo que detecta hardware, verifica dependencias, e roda smoke test.

### Templates Ativos (D:\ComfyUI\user\default\workflows\templates\)

| Arquivo | Descricao | Status |
|---------|-----------|--------|
| **gemini-architectural-v3.json** | **Models (sem face swap) + 9-Layer prompts** | **ATIVO** |
| **gemini-faceswap-v3.json** | **Identity lock (com hero face) + 9-Layer** | **ATIVO** |
| influencer-hyperreal-sota-v2.json | Pipeline v2 (pre-9-Layer) | Arquivo |
| influencer-hyperreal-refchain-v1.json | Reference chain basica | Arquivo |
| influencer-hyperreal-api.json | Gemini direto sem ref | Arquivo |
| PIPELINE-INDEX.md | Indice completo de tudo | Referencia |

### Script Template

| Arquivo | Descricao |
|---------|-----------|
| **D:\ComfyUI\generate_template.py** | **Template reutilizavel - copiar e customizar** |

### Canvas Workflows (D:\ComfyUI\user\default\workflows\)

| Arquivo | Descricao |
|---------|-----------|
| gemini-hyperreal-photo.json | IFGeminiNode -> SaveImage (template base legado) |

---

## 6. O QUE FALTA: PRIORIDADES DE INSTALAÇÃO

> **Detalhes completos:** Ver `KB-sdxl-models.md` (download plan 4 fases) e `KB-sdxl-workflows.md` (workflows prontos)

### Fase 1: SDXL Checkpoint (Prioridade CRITICA — ~6.6GB)

| Item | Arquivo | Tamanho | Destino | Fonte |
|------|---------|---------|---------|-------|
| **Juggernaut XL v9** | juggernautXL_v9Rundiffusion.safetensors | ~6.62GB | models\checkpoints\ | CivitAI |

**Por que:** Abre pipeline SD+LoRA inteiro. Baked VAE (nao precisa VAE separado). Melhor checkpoint SDXL para fotorealismo. DirectML compativel (12GB+ VRAM).

### Fase 2: LoRAs Core (Prioridade ALTA — ~200MB total)

| Item | Arquivo | Peso | Destino |
|------|---------|------|---------|
| Skin Realism SDXL | skin_realism_sdxl.safetensors | 0.5 | models\loras\ |
| Better Portraits Eyes v2 | better_portraits_eyes_v2.safetensors | 0.6 | models\loras\ |
| EasyFix Negative LoRA | easyfix_negative.safetensors | 1.0 (neg) | models\loras\ |

**Por que:** Stack de 3 LoRAs = poros, olhos, reduz artifacts. Testado e documentado em KB-sdxl-models.md.

### Fase 3: LoRAs Especializados (Prioridade MEDIA — ~300MB)

| Item | Arquivo | Peso | Uso |
|------|---------|------|-----|
| FilmGrain Aesthetic SDXL | filmgrain_aesthetic.safetensors | 0.3-0.5 | Grain cinematografico |
| Perfect Hands SDXL | perfect_hands_sdxl.safetensors | 0.6-0.8 | Corrigir maos |
| Detail Tweaker XL | detail_tweaker_xl.safetensors | -0.5 a +1.0 | Controle de detalhe |

### Fase 4: Checkpoints Alternativos + Extras (Prioridade BAIXA — ~14GB)

| Item | Tipo | Tamanho | Por Que |
|------|------|---------|---------|
| RealVisXL V5.0 Lightning | Checkpoint | ~6.5GB | Portraits + 8-step speed |
| epiCRealism XL LastFAME | Checkpoint | ~6.5GB | Textura cinematografica |
| ComfyUI-Impact-Pack | Custom Node | ~100MB | FaceDetailer, SAM |
| ComfyUI-ProPost | Custom Node | ~50MB | Color grading LUT |

### NAO PRIORIZAR

| Item | Motivo |
|------|--------|
| **Flux.1 (qualquer)** | **BROKEN no DirectML** — nao funciona, nao tentar |
| SD 1.5 Checkpoint | SDXL e superior em tudo exceto VRAM |
| ControlNet models | Requer checkpoint SD — avaliar apos Fase 1 |
| IP-Adapter | Crasha no DirectML (issues documentados) |
| AnimateDiff | Video nao e prioridade |
| Audio models | Fora do escopo |

---

## 7. CAPACIDADES ATUAIS vs DESEJADAS

### O Que PODEMOS Fazer Agora

| Capacidade | Status | Qualidade |
|------------|--------|-----------|
| Geracao txt2img (Gemini 3 Pro) | OPERACIONAL | 9/10 |
| Geracao img2img (Gemini 3 Pro) | OPERACIONAL | 9/10 |
| Face Swap (ReActor+InSwapper) | OPERACIONAL | 8/10 |
| Face Restore (CodeFormer via ReActor) | OPERACIONAL | 8/10 |
| Upscale 4x (ESRGAN) | OPERACIONAL | 9/10 |
| Upscale 4x (UltraSharp) | OPERACIONAL | 9/10 |
| Batch Generation | OPERACIONAL | 8/10 |
| 9-Layer Architectural Prompts | OPERACIONAL | 9/10 |
| Chat Refine (iterativo) | OPERACIONAL | 7/10 |
| Sequential Generation | OPERACIONAL | 7/10 |
| 38 Task Presets | OPERACIONAL | 8/10 |
| Analise de imagem (Gemini) | OPERACIONAL | 9/10 |

### O Que NAO Podemos Fazer Agora (Downloads Pendentes)

| Capacidade | Requer | Status | Prioridade |
|------------|--------|--------|------------|
| SDXL txt2img local | Juggernaut XL v9 (~6.6GB download) | **DOCUMENTADO** — KB-sdxl-workflows.md tem JSON pronto | **CRITICA** |
| SDXL + LoRA stacking | Checkpoint + 3 LoRAs (~200MB) | **DOCUMENTADO** — KB-sdxl-models.md tem configs | **ALTA** |
| Pipeline hibrido Gemini→SD | Checkpoint + img2img (denoise 0.30) | **DOCUMENTADO** — workflow JSON pronto | **ALTA** |
| Color grading (LUT) | ComfyUI-ProPost | Nao documentado | MEDIA |
| Face detection avancada | Impact-Pack | Nao documentado | MEDIA |
| Inpainting local | SD Checkpoint (mesma Fase 1) | Parcialmente documentado | MEDIA |
| ControlNet (pose/depth) | ControlNet models + checkpoint | Incompativel sem SD checkpoint | BAIXA |
| Style transfer avancado | IP-Adapter | **CRASHA no DirectML** — SKIP | SKIP |
| Video generation | AnimateDiff ou SVD | Fora de escopo | NAO PRIORIZAR |

> **NOTA:** Quando Fase 1 (Juggernaut XL) e Fase 2 (3 LoRAs) estiverem instalados, mover as 3 primeiras linhas para "Podemos Fazer Agora" e atualizar este arquivo.

---

## 8. MCP COMFYUI: TOOLS DISPONÍVEIS (37+ Tools)

**Fonte:** `D:\comfyui-mcp\src\index.ts` — MCP server com 37+ tools registrados.
**Invocação:** `mcp__comfyui__{tool_name}` (ex: `mcp__comfyui__run_workflow`)

### Status & Setup

| Tool MCP | Função |
|----------|--------|
| `get_status` | Verificar se ComfyUI está rodando |
| `get_install_guide` | Guia de instalação |
| `get_model_guide` | Guia de modelos disponíveis |

### Examples & Discovery

| Tool MCP | Função |
|----------|--------|
| `list_examples` | Listar workflows de exemplo |
| `get_example_workflow` | Obter workflow de exemplo específico |
| `extract_workflow` | Extrair workflow de imagem gerada |
| `recommend_workflow` | Recomendar workflow para um modelo |

### Prompting

| Tool MCP | Função |
|----------|--------|
| `get_prompting_guide` | Guia de prompting (best practices) |

### Generation (PRINCIPAL)

| Tool MCP | Função |
|----------|--------|
| `get_capabilities` | Detectar capabilities do ComfyUI |
| `run_workflow` | **EXECUTAR workflow** (sync/async) — principal para geração |
| `get_image` | Obter imagem gerada |

### Model & Node Discovery

| Tool MCP | Função |
|----------|--------|
| `list_models` | Listar checkpoints, loras, vae, upscalers, etc. |
| `list_nodes` | Listar todos os nodes disponíveis |
| `get_node_info` | Info detalhada de um node |
| `find_nodes_by_type` | Buscar nodes por tipo/categoria |
| `build_node` | Construir definição de node |
| `validate_workflow` | Validar workflow antes de executar |

### Templates

| Tool MCP | Função |
|----------|--------|
| `search_templates` | Buscar templates salvos |
| `get_template` | Recuperar template específico |
| `save_template` | Salvar workflow como template |
| `delete_template` | Deletar template |
| `get_download_url` | URL de download de modelo |

### Queue & History

| Tool MCP | Função |
|----------|--------|
| `get_queue` | Ver fila atual de execução |
| `cancel_job` | Cancelar job na fila |
| `interrupt` | Interromper execução atual |
| `get_history` | Histórico de execuções |

### Tasks (Async)

| Tool MCP | Função |
|----------|--------|
| `get_task` | Obter task em andamento |
| `get_task_result` | Resultado de task concluída |
| `list_tasks` | Listar todas as tasks |
| `cancel_task` | Cancelar task |
| `get_generation_by_name` | Buscar geração por nome |
| `name_generation` | Nomear uma geração |

### Notes (Persistência)

| Tool MCP | Função |
|----------|--------|
| `save_note` | Salvar nota/contexto |
| `get_notes` | Obter notas salvas |
| `search_notes` | Buscar em notas |
| `delete_note` | Deletar nota |
| `list_topics` | Listar tópicos de notas |

### Utilities

| Tool MCP | Função |
|----------|--------|
| `get_user_preferences` | Preferências do usuário |
| `render_svg` | Renderizar SVG para PNG |
| `download_font` | Baixar fonte do Google Fonts |
| `list_fonts` | Listar fontes disponíveis |

### Como Executar Workflow (Método Preferido: MCP)

```
mcp__comfyui__run_workflow  →  Envia workflow JSON para ComfyUI
mcp__comfyui__get_task_result  →  Obtém resultado da execução
mcp__comfyui__get_image  →  Obtém imagem gerada
```

### Fallback: REST API (Se MCP falhar)

```bash
curl -s http://127.0.0.1:8188/api/prompt -X POST \
  -H "Content-Type: application/json" \
  -d '{"prompt": {WORKFLOW_JSON}, "client_id": "comfyui-ops"}'

curl -s http://127.0.0.1:8188/api/history/{PROMPT_ID}
```

---

## 9. GEMINI API: CONFIGURAÇÃO

| Aspecto | Valor |
|---------|-------|
| **API Key Location** | D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\.env |
| **Key Variable** | GEMINI_API_KEY |
| **Default Model** | gemini-3-pro-image-preview (OBRIGATORIO para qualidade) |
| **Patch Location** | gemini_node.py linhas 935-947 |
| **Rate Limits** | Verificar em console.cloud.google.com |

### Modelos Gemini Suportados (Imagem)

| Model ID (no ComfyUI) | Model Real | Capacidade | Qualidade |
|------------------------|-----------|------------|-----------|
| **gemini-3-pro-image-preview** | gemini-3-pro-image-preview | Geracao + analise | **MELHOR - usar este** |
| gemini-2.5-flash | gemini-2.5-flash-image | Geracao + analise | Inferior - look artificial |
| gemini-2.0-flash | gemini-2.0-flash | Geracao + analise | Inferior |
| gemini-1.5-pro | gemini-1.5-pro | Analise (sem geracao) | N/A |
| gemini-1.5-flash | gemini-1.5-flash | Analise (sem geracao) | N/A |

**NOTA:** gemini-3-pro foi adicionado manualmente ao IF_Gemini. Arquivos editados:
- `comfy_api_nodes/nodes_gemini.py`
- `custom_nodes/ComfyUI-IF_Gemini/gemini_node.py`
- `custom_nodes/ComfyUI-IF_Gemini/api_routes.py`
- `custom_nodes/ComfyUI-IF_Gemini/web/js/gemini_node.js`
Reaplicar apos atualizar ComfyUI ou IF_Gemini.

---

## 10. CHECKLIST PRE-OPERAÇÃO

### Antes de Gerar Qualquer Imagem

```
[ ] ComfyUI esta rodando? (http://127.0.0.1:8188)
[ ] API key esta configurada? (.env no IF_Gemini)
[ ] Modelo = gemini-3-pro-image-preview? (NAO 2.5-flash)
[ ] Temperature = 1.0? (NAO 0.8)
[ ] operation_mode = "generate_images"?
[ ] aspect_ratio definido? (4:5 portrait, 3:4 editorial)
[ ] Prompt segue 9-Layer Architecture? (KB-prompt-patterns.md)
[ ] Prompt NAO tem anti-patterns? (8K, ultra-detailed, stunning)
```

### Antes de Usar Pipeline Completo

```
[ ] Checklist acima OK
[ ] codeformer.pth existe em facerestore_models/?
[ ] Upscaler escolhido existe em upscale_models/?
[ ] Output directory tem espaço?
```

### Antes de Instalar Algo Novo

```
[ ] ComfyUI está parado? (ideal para instalar)
[ ] ComfyUI-Manager está funcionando?
[ ] Sabe em qual diretório o modelo vai?
[ ] Este arquivo será atualizado após instalação?
```

---

## 11. PATHS RÁPIDOS

| O Que | Path |
|-------|------|
| ComfyUI root | D:\ComfyUI |
| Custom nodes | D:\ComfyUI\custom_nodes\ |
| IF_Gemini node | D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\ |
| ComfyUI Manager | D:\ComfyUI\custom_nodes\ComfyUI-Manager\ |
| Checkpoints | D:\ComfyUI\models\checkpoints\ |
| LoRAs | D:\ComfyUI\models\loras\ |
| Upscale models | D:\ComfyUI\models\upscale_models\ |
| Face restore models | D:\ComfyUI\models\facerestore_models\ |
| ControlNet models | D:\ComfyUI\models\controlnet\ |
| VAE | D:\ComfyUI\models\vae\ |
| Output images | D:\ComfyUI\output\ |
| Saved workflows | D:\ComfyUI\user\default\workflows\ |
| Banana presets | D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\presets\banana-tasks.json |
| Gemini .env | D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\.env |
| Gemini node source | D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\gemini_node.py |
| KB files | knowledge/ |

---

## REGRA: MANTER ATUALIZADO

```
TODA VEZ que instalar/remover QUALQUER coisa:
1. Atualizar a seção relevante deste arquivo
2. Mover item da lista "Falta" para "Instalado"
3. Atualizar status em "Capacidades Atuais vs Desejadas"

Este é o INVENTÁRIO REAL. Se não está aqui, não existe.
```

---

## 12. KBs RELACIONADOS

| KB | Conteudo | Quando Consultar |
|----|----------|------------------|
| **KB-sdxl-models.md** | Checkpoints, LoRAs, stacking rules, download plan | Antes de instalar modelos SD |
| **KB-sdxl-workflows.md** | Workflow JSONs, hybrid pipeline, Python script | Antes de rodar pipeline SD |
| KB-capabilities.md | IF_Gemini node, 38 presets, parametros | Pipeline Gemini |
| KB-prompting-heuristics.md | Narrativo>keywords, master template | Craftar prompts |
| KB-workflow-patterns.md | 7 workflows Gemini, decision matrix | Escolher workflow |
| KB-pipeline-reference.md | 4 estagios, face restore, upscale | Pipeline completo |
| KB-prompt-patterns.md | 9-Layer architecture | Prompts avancados |

---

*KB-System-Inventory - Estado Real do Sistema*
*3 custom nodes (IF_Gemini, Manager, ReActor) | 5+ modelos | Inventario completo*
*Criado: 2026-02-06 | Atualizado: 2026-02-06 (v4 — SDXL research consolidado)*
*KBs novos: KB-sdxl-models.md + KB-sdxl-workflows.md*
*Atualizar apos cada instalacao*
