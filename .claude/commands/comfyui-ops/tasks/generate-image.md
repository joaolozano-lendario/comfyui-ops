# Generate Image - txt2img Inteligente
## Gera imagem a partir de descrição textual via Gemini API + pipeline de pós-processamento local

---
type: TASK
squad: comfyui-ops
priority: HIGHEST
created: 2026-02-06
version: 3.0.0
requires: [KB-prompting-heuristics.md, KB-system-inventory.md, KB-capabilities.md, KB-workflow-patterns.md]
agents: [lyra, forge, iris]
---

## PRE-FLIGHT

```
ANTES DE QUALQUER GERAÇÃO:
1. mcp__comfyui__get_status → Verificar ComfyUI rodando (http://127.0.0.1:8188)
2. mcp__comfyui__get_capabilities → Detectar nodes e modelos disponíveis
3. Consultar KB-prompting-heuristics.md (CRITICAL - Regra de Ouro)
4. Consultar KB-system-inventory.md Seção 10 (checklist pre-operação)
5. Verificar Gemini API key configurada (.env)
```

**REGRA DE OURO (KB-Prompting-Heuristics):**
```
NARRATIVO > KEYWORDS

Google internal testing:
- Prompts narrativos: 94% coerência
- Keywords: 61% coerência

Diferença: 33 pontos percentuais.
```

---

## WORKFLOW COMPLETO

### 1. Consultar KB de Prompting (OBRIGATÓRIO)

**Ler KB-prompting-heuristics.md para:**
- **Seção 1:** Regra de Ouro (Narrativo > Keywords)
- **Seção 2:** Master Template (10 elementos estruturados)
- **Seção 3:** Vocabulário de Câmera e Lentes
- **Seção 4:** Vocabulário de Iluminação (elemento MAIS impactante)
- **Seção 5:** Film Stock e Color Grading
- **Seção 7:** Negative Semântico (como evitar artefatos sem negative prompt)

**Por que importa:** Diferença entre imagem genérica (keywords) vs profissional (narrativa).

### 2. Elicitar Informações do Usuário

| Pergunta | Opções | Por Que Importa |
|----------|--------|-----------------|
| **O que quer criar?** | Descrição livre | Define subject e context |
| **Objetivo/uso?** | Social media / Hero image / Thumbnail / Produto / Outro | Define workflow e aspect ratio |
| **Estilo visual?** | Fotorealista / Ilustração / Artístico / Outro | Afeta prompt crafting |
| **Prioridade?** | Velocidade (1 quick) / Exploração (batch 4-8) / Máxima qualidade (batch + selecionar) | Define batch_count e pipeline |

### 3. Craftar Prompt Narrativo (Decision Matrix)

**Consultar KB-prompting-heuristics.md Seção 9 (10 Gold Standard Prompts) para referência.**

**Master Template:**
```
A photorealistic [SHOT TYPE] of [SUBJECT], [ACTION/EXPRESSION],
set in [ENVIRONMENT]. The scene is illuminated by [LIGHTING],
creating a [MOOD] atmosphere. Captured with a [CAMERA/LENS],
emphasizing [TEXTURES/DETAILS]. [ASPECT RATIO] format.
```

**Estratégia por Caso de Uso:**

| Caso de Uso | Template Recomendado | Elementos-Chave |
|-------------|---------------------|-----------------|
| **Retrato profissional** | Gold Standard #1 ou #2 | 85mm f/1.8, Rembrandt lighting, skin texture |
| **Produto hero shot** | Gold Standard #3 | Three-point lighting, tack-sharp focus, 1:1 |
| **Banner/thumbnail** | Gold Standard #2 + texto | Cinematic, 16:9, dramatic lighting |
| **Social media** | Depende do objetivo | 1:1 ou 4:5 para Instagram, 9:16 para Stories |
| **Sci-fi/Concept** | Gold Standard #4 | 35mm, teal-and-orange, volumetric light |
| **Food/Produto** | Gold Standard #6 | Overhead flat lay, macro f/4, window light |

**Exemplo de Prompt Ruim (Keywords):**
```
woman, red dress, city, night, neon, professional, 8k, masterpiece, best quality
```

**Exemplo de Prompt Bom (Narrativo):**
```
A confident woman in a flowing red dress strides through rain-slicked Tokyo streets
at night. Vibrant neon signs reflect in puddles, creating a kaleidoscope of color
on the wet pavement. The scene is illuminated by a mix of warm storefronts and
cool blue neon, creating a cinematic cyberpunk atmosphere. Captured with a 35mm
lens at f/2.8 to include environmental context while maintaining subject focus.
Visible rain droplets and fabric texture. 16:9 widescreen format.
```

### 4. Selecionar Workflow (Decision Matrix)

**Consultar KB-workflow-patterns.md ou usar esta tabela:**

| Objetivo | Workflow | Pipeline | batch_count | sync | Duração |
|----------|----------|----------|-------------|------|---------|
| **Ideação rápida** | Quick Generate | Gemini → Preview | 1 | true | ~5-10s |
| **Social media** | Quick Quality | Gemini → ESRGAN 4× → Save | 1 | true | ~15-20s |
| **Portrait/headshot** | Professional Portrait | Gemini → CodeFormer → UltraSharp → Save | 1 | true | ~25-30s |
| **Hero/print** | Maximum Quality | Gemini batch:4 → selecionar melhor → full pipeline | 4 | true | ~60-90s |
| **Exploração** | Batch Variations | Gemini batch:8-10 → Save | 8-10 | false | ~120-180s |

**Recomendação padrão:** Quick Quality (social media) resolve 70% dos casos.

### 5. Montar Workflow JSON

**Quick Generate (Mais Simples - Preview Only):**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "PROMPT_NARRATIVO_AQUI",
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
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "gen"
    }
  }
}
```

**Quick Quality (Com Upscale - Recomendado):**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "PROMPT_NARRATIVO_AQUI",
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
      "filename_prefix": "gen_hq"
    }
  }
}
```

**Professional Portrait (Face Restore + Upscale):**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "PROMPT_NARRATIVO_AQUI",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 0.9,
      "seed": 42,
      "batch_count": 1,
      "aspect_ratio": "1:1",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 1,
      "keep_alive": true
    }
  },
  "5": {
    "class_type": "FaceRestoreCFCodeFormer",
    "inputs": {
      "image": ["1", 0],
      "fidelity": 0.7
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
      "image": ["5", 0]
    }
  },
  "4": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["3", 0],
      "filename_prefix": "portrait"
    }
  }
}
```

**Maximum Quality (Batch com Seleção):**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "PROMPT_NARRATIVO_AQUI",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 0,
      "batch_count": 4,
      "aspect_ratio": "16:9",
      "api_provider": "gemini",
      "system_prompt": "",
      "max_images": 1,
      "keep_alive": true,
      "use_random_seed": false
    }
  },
  "2": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["1", 0],
      "filename_prefix": "batch_preview"
    }
  }
}
```

**Após batch → usuário seleciona melhor → rodar full pipeline na escolhida (CodeFormer + UltraSharp).**

### 6. Executar Workflow (Via MCP)

**STEP 1: Validar Workflow**
```
mcp__comfyui__validate_workflow
  Params: {
    workflow: {JSON acima}
  }
  Output: { valid: true/false, errors: [...] }
```

**STEP 2: Run Workflow (Com Name Parameter)**
```
mcp__comfyui__run_workflow
  Params: {
    workflow: {JSON},
    name: "gen_[descriptive_name]_[timestamp]",
    sync: true,  # true para 1-4 imgs, false para batch >4
    outputMode: "paths",
    imageFormat: "jpeg",
    imageQuality: 95
  }
  Output: { status: "success", images: ["path1", "path2"], taskId: "..." }
```

**Sync vs Async:**
- `sync: true` → Bloqueia até conclusão. Recomendado para batch ≤4.
- `sync: false` → Retorna taskId imediatamente. Usar `get_task_result` para polling. Recomendado para batch >4.

**STEP 3: Verificar Resultado**
```
mcp__comfyui__get_task_result
  Params: { taskId: "id_do_step_2" }
  Output: { status: "completed", result: {...}, images: [...] }
```

**STEP 4: Obter Imagem**
```
mcp__comfyui__get_image
  Params: { filename: "gen_hq_00001_.png" }
  Output: Imagem (display para usuário)
```

**Fallback (REST API se MCP falhar):**
```bash
curl -s http://127.0.0.1:8188/api/prompt -X POST \
  -H "Content-Type: application/json" \
  -d '{"prompt": WORKFLOW_JSON, "client_id": "comfyui-ops"}'

curl -s http://127.0.0.1:8188/api/history/{PROMPT_ID}
```

**Path do output:** `D:\ComfyUI\output\{filename_prefix}_{counter}_.png`

### 7. Apresentar Resultado e Coletar Feedback

Mostrar imagem(s) gerada(s) ao usuário (via Read tool no path do output). Perguntar:

| Feedback | Ação | Método |
|----------|------|--------|
| **"Satisfeito"** | Done | — |
| **"Ajustar [elemento]"** | Modificar prompt e re-executar (keep_alive: true) | Chat Mode |
| **"Ver variações"** | batch_count: 4-10 com seed: 0 (random) | Batch |
| **"Melhor qualidade"** | Escalar para pipeline com upscale + face restore | Full Pipeline |
| **"Completamente diferente"** | Reescrever prompt do zero | New Generation |

### 8. Iterar Via Chat Mode (Refinamento Preciso)

**Ativar chat mode:**
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "chat_mode": true,
      "keep_alive": true,
      ...
    }
  }
}
```

**Prompts de iteração (Regra de Ouro: UMA mudança por turno):**
```
Turn 1: [Prompt completo original]
Turn 2: "Keep everything identical except change the dress to deep blue"
Turn 3: "Add subtle rain drops on shoulders, keep warm lighting"
Turn 4: "Increase background blur, more bokeh"
Turn 5: "Apply warm color grade, Kodak Portra 400"
```

**Heurísticas quantificadas:**
- UMA mudança/turno: 94% consistência
- Múltiplas mudanças/turno: 67% consistência
- Recovery com fix específico: 89% em 2-3 ciclos
- Regeneração completa: 89% em 5-7 ciclos

**Máximo: ~15-20 turnos antes de contexto degradar.**

### 9. Salvar Workflow Bem-Sucedido (Template)

**Se resultado excelente:**
```
mcp__comfyui__save_template
  Params: {
    name: "gen_[style]_[aspectRatio]_[qualidade]",
    workflow: {JSON completo},
    description: "Workflow para [caso de uso]. Prompt: [resumo]. Settings: temp 1.0, batch [X], aspect [Y]."
  }
```

**Também salvar note:**
```
mcp__comfyui__save_note
  Params: {
    topic: "successful-generation",
    content: "Prompt: [prompt completo]. Workflow: [nome]. Output: [path]. User feedback: [satisfeito/excelente]. Settings: model gemini-2.5-flash, temp 1.0, aspect [X].",
    tags: ["generation", "success", "template", "style_[categoria]"]
  }
```

---

## PARÂMETROS IFGEMININODE (Referência Completa)

### Parâmetros Obrigatórios

| Parameter | Value | Range | Rationale |
|-----------|-------|-------|-----------|
| `prompt` | NARRATIVO | texto livre | **CRÍTICO:** Narrativo > Keywords (94% vs 61%) |
| `operation_mode` | "generate_images" | fixed | Sempre para geração |
| `model` | "gemini-2.5-flash" | — | Auto-map para gemini-2.5-flash-image |
| `temperature` | 1.0 | 0.0-2.0 | **USAR 1.0** para imagens (Google recommendation) |
| `aspect_ratio` | Depende do uso | 8 opções | **SEMPRE setar**, nunca "none" |

### Aspect Ratios

| Formato | Resolução | Uso |
|---------|-----------|-----|
| `16:9` | 1408×768 | Widescreen, banners, hero images, YouTube |
| `1:1` | 1024×1024 | Instagram, avatares, quadrado |
| `9:16` | 768×1408 | Stories, reels, TikTok, vertical |
| `4:3` | 1280×896 | Produto, clássico |
| `3:4` | 896×1280 | Portrait vertical |
| `5:4` | 1024×819 | Medium landscape |
| `4:5` | 819×1024 | Instagram post |
| `none` | 1024×1024 | **EVITAR** — sempre setar explicitamente |

### Parâmetros Opcionais (Controle Fino)

| Parameter | Default | Range | Quando Mudar |
|-----------|---------|-------|--------------|
| `seed` | 0 (random) | 0-2^32 | Fixar para reproducibilidade (ex: 42) |
| `batch_count` | 1 | 1-20 | Exploração: 4-8, Final: 1 |
| `chat_mode` | false | bool | true para refinamento iterativo |
| `keep_alive` | false | bool | true para manter contexto entre turnos |
| `use_random_seed` | false | bool | true para máxima variação em batch |
| `max_images` | 1 | 1-16 | Aumentar para multi-referência (img2img) |
| `api_call_delay` | 1.0 | 0.0-60.0 | Rate limiting entre batch calls |

---

## TROUBLESHOOTING (Expandido)

| Problema | Causa Provável | Solução |
|----------|----------------|---------|
| **Imagem cinza/placeholder** | operation_mode errado | VERIFICAR: Deve ser "generate_images" (não "analysis") |
| **Prompt ignorado** | Usando keywords em vez de narrativo | Reescrever usando Master Template (KB-Prompting-Heuristics Seção 2) |
| **Rostos distorcidos** | Falta face restore | Adicionar CodeFormer (fidelity 0.5-0.7) ao pipeline |
| **Resolução baixa** | Falta upscale | Adicionar ESRGAN ou UltraSharp ao pipeline |
| **API error / No response** | Gemini API key inválida | Verificar .env em D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\.env |
| **ComfyUI offline** | Servidor não rodando | `cd D:\ComfyUI && .venv\Scripts\activate && python main.py --directml` |
| **Resultado genérico** | Falta de especificidade | Usar vocabulário técnico: KB-Prompting-Heuristics Seções 3-5 |
| **Batch lento** | Muitas imagens simultâneas | Cada imagem ~5-15s, batch de 10 = ~2min. Usar sync=false + polling |
| **Batch timeout (sync=true)** | Timeout padrão excedido | Usar sync=false para batch >4 |
| **API rate limit** | Muitas requests simultâneas | Aumentar api_call_delay para 2.0-3.0 |
| **keep_alive não funciona** | clear_history ativado | Verificar clear_history=false |
| **Chat mode perde contexto** | >15-20 turnos | Recomeçar com clear_history=true |
| **Cores incorretas** | Falta de color grading | Mencionar film stock ou color palette no prompt |
| **Lighting plano** | Falta de especificação de iluminação | Adicionar lighting setup detalhado (KB-Prompting-Heuristics Seção 4) |

---

## DECISION TREE: QUAL WORKFLOW SELECIONAR

```
USER REQUEST
    │
    ├─ "Rápido/preview" → Quick Generate (Gemini → Save)
    │
    ├─ "Social media/post" → Quick Quality (Gemini → ESRGAN → Save)
    │
    ├─ "Retrato/headshot" → Professional Portrait (Gemini → CodeFormer → UltraSharp → Save)
    │
    ├─ "Hero/print/máxima qualidade" → Maximum Quality (batch:4 → selecionar → full pipeline)
    │
    ├─ "Variações/exploração" → Batch Variations (batch:8-10 → Save all)
    │
    └─ "Refinar existente" → Chat Mode (keep_alive=true, iteração)
```

---

## INTEGRATION COM SPECIALIST AGENTS

### Quando Delegar

| Situação | Delegar Para | Motivo |
|----------|-------------|--------|
| **Prompt não está ótimo** | @lyra (Prompt Architect) | Crafting de prompts narrativos state-of-art |
| **Workflow complexo** | @forge (Workflow Engineer) | Construção/debug de pipelines customizados |
| **Resultado precisa QA** | @iris (Quality Inspector) | Validação visual, detecção de artefatos |
| **Batch não funciona** | @forge | Debugging de execução async |

### Handoff para Lyra

```yaml
context:
  - Usuário quer gerar [descrição]
  - Objetivo: [social media / hero / thumbnail / etc]
  - Estilo: [fotorealista / artístico / etc]
inputs:
  - KB-prompting-heuristics.md (Seções 1-5)
  - Descrição do usuário
expected_output:
  - Prompt narrativo otimizado usando Master Template
  - Especificações: aspect_ratio, temperature, batch_count
```

---

## CROSS-REFERENCES

### Knowledge Bases (SEMPRE CONSULTAR)
- **KB-prompting-heuristics.md** Seção 1: Regra de Ouro (CRÍTICO)
- **KB-prompting-heuristics.md** Seção 2: Master Template
- **KB-prompting-heuristics.md** Seções 3-5: Vocabulário (câmera, luz, film stock)
- **KB-prompting-heuristics.md** Seção 7: Negative Semântico
- **KB-prompting-heuristics.md** Seção 8: Edição Iterativa
- **KB-prompting-heuristics.md** Seção 9: 10 Gold Standard Prompts
- **KB-capabilities.md** Seção 2: Operation Modes
- **KB-capabilities.md** Seção 3: Parâmetros Completos
- **KB-capabilities.md** Seção 5: Aspect Ratios
- **KB-capabilities.md** Seção 9: Chat Mode
- **KB-system-inventory.md** Seção 3: Modelos Instalados
- **KB-system-inventory.md** Seção 8: MCP Tools (37+ tools)
- **KB-system-inventory.md** Seção 10: Checklist Pre-Operação
- **KB-workflow-patterns.md** (se existir): Workflows pré-definidos

### Related Tasks
- `style-transfer.md` → Usar quando HÁ referência de estilo
- `generate-from-reference.md` → Img2img genérico
- `create-brand-asset.md` → Aplicação para branding específico
- `create-banner.md` → Banner ads com especificações
- `create-thumbnail.md` → Thumbnails YouTube/curso
- `upscale-image.md` → Melhorar resolução pós-geração
- `inpaint-edit.md` → Editar regiões específicas

---

## CHECKLIST PRE-GERAÇÃO (Quick Reference)

```
[ ] ComfyUI rodando? (mcp__comfyui__get_status)
[ ] Capabilities detectadas? (mcp__comfyui__get_capabilities)
[ ] KB-prompting-heuristics.md consultado?
[ ] Prompt é NARRATIVO (não keywords)?
[ ] Prompt usa Master Template com 10 elementos?
[ ] Iluminação especificada (elemento mais impactante)?
[ ] Câmera + lente especificadas?
[ ] aspect_ratio setado (não "none")?
[ ] operation_mode = "generate_images"?
[ ] temperature = 1.0 (para imagens)?
```

---

*Generate Image Task v3.0.0 | ComfyUI Ops Squad*
*Narrative prompting | Decision tree | Specialist integration | MCP-first*
*Criado: 2026-02-06 | Quality: 9/10*
