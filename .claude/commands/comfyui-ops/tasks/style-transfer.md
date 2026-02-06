# Style Transfer - Transferir Estilo Visual Entre Imagens
## Aplica o estilo visual de uma imagem de referência sobre novo conteúdo

---
type: TASK
squad: comfyui-ops
priority: HIGH
created: 2026-02-06
version: 2.0.0
requires: [KB-capabilities.md, KB-prompting-heuristics.md, KB-system-inventory.md]
agents: [lyra, forge, iris]
---

## PRE-FLIGHT

```
ANTES DE QUALQUER STYLE TRANSFER:
1. mcp__comfyui__get_status → Verificar ComfyUI rodando
2. mcp__comfyui__get_capabilities → Detectar capacidades
3. Consultar KB-capabilities.md Seção 6 (Image Input)
4. Consultar KB-prompting-heuristics.md Seção 7 (Negative Semântico)
5. Verificar imagens de referência estão acessíveis
```

**Pré-requisitos:**
- ComfyUI rodando (http://127.0.0.1:8188)
- IFGeminiNode disponível
- Gemini API key configurada
- LoadImage node (core ComfyUI)

---

## WORKFLOW COMPLETO

### 1. Elicitar Inputs do Usuário

| Input | Tipo | Obrigatório | Exemplo |
|-------|------|-------------|---------|
| **Imagem de estilo** | Path ou URL | SIM | Pintura Van Gogh, foto cinematográfica, ilustração anime |
| **Descrição conteúdo** | Texto | SIM | "Retrato de mulher em jardim" |
| **Intensidade estilo** | Dropdown | NÃO (default: balanced) | `subtle` / `balanced` / `heavy` |
| **Aspect ratio** | Dropdown | NÃO (default: 1:1) | 16:9, 1:1, 9:16, 4:3, 3:4 |

### 2. Carregar Imagem de Referência

**Via LoadImage:**
```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": {
      "image": "style_reference.jpg"
    }
  }
}
```

**Regra:** Colocar imagem de estilo no diretório `D:\ComfyUI\input\` antes de executar.

### 3. Selecionar Estratégia de Prompting (Decision Matrix)

| Caso de Uso | Estratégia | Complexity |
|-------------|-----------|-----------|
| **Estilo fotográfico simples** (golden hour, film stock) | A) Single Reference | Baixa |
| **Estilo artístico forte** (Van Gogh, anime, pintura) | A) Single Reference + descrição detalhada | Média |
| **Fusão de múltiplos estilos** | B) Multi-Reference (até 16 imagens) | Alta |
| **Iteração precisa** | C) Chat Refinement (keep_alive=true) | Alta |

**Recomendação padrão:** Estratégia A (Single Reference) resolve 80% dos casos.

### 4. Craftar Prompt de Style Transfer

**Template Master:**
```
Apply the visual style of the reference image to [SUBJECT].
Maintain the [STYLE ELEMENTS] of the reference (color palette, lighting,
texture, brushstrokes) while preserving the [IDENTITY/COMPOSITION] of
the subject. [SPECIFIC GUIDANCE].
```

**Variáveis:**

| Variável | O Que Controlar | Exemplo |
|----------|----------------|---------|
| `[SUBJECT]` | Descrição do conteúdo novo | "a portrait of a woman in a garden" |
| `[STYLE ELEMENTS]` | Aspectos do estilo a transferir | "thick oil brushstrokes, warm golden palette" |
| `[IDENTITY/COMPOSITION]` | O que preservar do sujeito | "facial features and pose" |
| `[SPECIFIC GUIDANCE]` | Instruções adicionais | "impressionist style, visible canvas texture" |

**Exemplo Completo:**
```
Apply the visual style of the reference image to a portrait of a young woman
standing in a garden at sunset. Maintain the thick oil brushstrokes, warm
golden palette, and impressionist technique of the reference while preserving
the subject's facial features, expression, and pose. Emphasize visible brush
texture and soft edges characteristic of impressionist painting. 1:1 format.
```

### 5. Definir Intensidade de Estilo (Via Temperature)

| Intensidade | Temperature | seed | batch_count | Efeito |
|-------------|-----------|------|-------------|--------|
| **Subtle** | 0.7 | fixo (42) | 1 | Leve influência, sujeito dominante |
| **Balanced** | 1.0 | 0 (random) | 4 | Mix equilibrado (RECOMENDADO) |
| **Heavy** | 1.0 | 0 | 4 → selecionar | Estilo domina, sujeito pode se perder |

**Heurística:** Temperature NÃO controla intensidade diretamente. Intensidade vem do **prompt specificity**:
- Subtle: Mencionar estilo brevemente ("inspired by the reference style")
- Heavy: Detalhar TODOS os elementos ("thick brushstrokes, visible canvas weave, specific color palette")

### 6. Montar Workflow JSON

**Workflow: LoadImage → IFGeminiNode → SaveImage**

```json
{
  "1": {
    "class_type": "LoadImage",
    "inputs": {
      "image": "style_reference.jpg"
    }
  },
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "PROMPT_AQUI",
      "images": ["1", 0],
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 0,
      "batch_count": 4,
      "aspect_ratio": "1:1",
      "api_provider": "gemini",
      "max_images": 1,
      "keep_alive": false
    }
  },
  "3": {
    "class_type": "SaveImage",
    "inputs": {
      "images": ["2", 0],
      "filename_prefix": "style_transfer"
    }
  }
}
```

**Multi-Reference (até 16 imagens):**
- Conectar múltiplos LoadImage nodes
- Aumentar `max_images` para 16
- **Protocolo de ordenação:**
  - `IMAGE_1` = SUJEITO principal
  - Imagens intermediárias = referências de estilo
  - ÚLTIMA imagem = BACKGROUND/ambiente

### 7. Validar e Executar

**Via MCP:**
```
STEP 1: mcp__comfyui__validate_workflow
  Params: { workflow: {JSON acima} }
  Output: Validação de sintaxe e nodes

STEP 2: mcp__comfyui__run_workflow
  Params: {
    workflow: {JSON},
    name: "style_transfer_van_gogh_portrait",
    sync: true,
    outputMode: "paths",
    imageFormat: "jpeg",
    imageQuality: 95
  }
  Output: { status, images, taskId }

STEP 3: mcp__comfyui__get_image
  Params: { filename: resultado[0] }
  Output: Imagem gerada
```

**Sync vs Async:**
- `sync: true` → Bloqueia até conclusão (recomendado para 1-4 imagens)
- `sync: false` → Retorna taskId, usar `get_task_result` depois (para batch >4)

### 8. Iterar Via Chat Mode (Refinamento)

**Ativar chat iterativo:**
```json
{
  "2": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "chat_mode": true,
      "keep_alive": true,
      ...
    }
  }
}
```

**Prompts de iteração:**
```
Turn 1: [Prompt completo original]
Turn 2: "Keep everything identical except increase the brushstroke intensity"
Turn 3: "Add more warm golden tones to the color palette"
Turn 4: "Make the background softer and more blurred, impressionist style"
```

**Regra de Ouro:** UMA mudança por turno = 94% consistência (vs 67% com múltiplas mudanças).

### 9. Apresentar Resultado e Coletar Feedback

Mostrar imagem(s) gerada(s) ao usuário. Perguntar:

| Feedback | Ação |
|----------|------|
| "Muito do estilo original" | Reduzir menção de elementos de estilo no prompt |
| "Pouco do estilo" | Aumentar detalhamento de características do estilo |
| "Cores erradas" | Adicionar descrição explícita de paleta ("warm golden tones, burnt sienna, ochre") |
| "Sujeito deformado" | Reforçar preservação de identidade no prompt |
| "Perfeito" | Salvar template: `mcp__comfyui__save_template` |

---

## STYLE CATEGORIES & PROMPTING GUIDELINES

### Fotografia (Film Stocks)

| Estilo | Prompt Template | Elementos-Chave |
|--------|----------------|-----------------|
| **Golden Hour** | "Apply the warm, directional lighting of golden hour..." | Lighting direction, warm color cast, long shadows |
| **Cinematic** | "Apply the cinematic color grading and lighting of the reference..." | Teal-and-orange, volumetric light, film grain |
| **Film Stock** | "Apply the characteristics of [Kodak Portra 400] film stock..." | Grain structure, color science, dynamic range |

### Artístico (Estilos de Pintura)

| Estilo | Prompt Template | Elementos-Chave |
|--------|----------------|-----------------|
| **Impressionist** | "Apply the impressionist painting style with visible brushstrokes..." | Thick brushwork, broken color, soft edges |
| **Oil Painting** | "Apply the rich oil painting technique with impasto texture..." | Canvas texture, layered paint, glazing |
| **Watercolor** | "Apply the translucent watercolor wash technique..." | Water stains, color bleeding, paper texture |
| **Anime/Manga** | "Apply the anime illustration style with clean linework..." | Cell shading, vibrant colors, stylized features |

### Comercial (Branding)

| Estilo | Prompt Template | Elementos-Chave |
|--------|----------------|-----------------|
| **Product Photo** | "Apply the studio lighting and backdrop style..." | Three-point lighting, reflections, clean background |
| **Editorial Fashion** | "Apply the editorial fashion photography aesthetic..." | High contrast, muted palette, dramatic composition |
| **Vintage** | "Apply the vintage [1960s] photography look..." | Era-specific color, grain, vignetting, fading |

---

## PARAMETERS REFERENCE

### IFGeminiNode (Style Transfer Specific)

| Parameter | Recommended Value | Range | Rationale |
|-----------|------------------|-------|-----------|
| `operation_mode` | "generate_images" | fixed | Sempre para geração |
| `model` | "gemini-2.5-flash" | — | Auto-map para gemini-2.5-flash-image |
| `temperature` | 1.0 | 0.0-2.0 | Criatividade máxima (Google recommendation) |
| `seed` | 0 (random) | 0-2^32 | Fixar só se precisar reproducibilidade |
| `batch_count` | 4 | 1-20 | Exploração: 4-8, Final: 1 |
| `max_images` | 1 (single ref) / 16 (multi-ref) | 1-16 | Aumentar para multi-referência |
| `aspect_ratio` | "1:1" | 8 opções | SEMPRE setar, nunca "none" |
| `chat_mode` | false (default) / true (refine) | bool | Ativar para edição iterativa |
| `keep_alive` | false / true | bool | True mantém contexto entre turnos |

---

## TROUBLESHOOTING

| Problema | Causa Provável | Solução |
|----------|----------------|---------|
| **Estilo não aparece na imagem** | Prompt muito vago sobre estilo | Detalhar elementos específicos (brushstrokes, palette, texture) |
| **Sujeito irreconhecível** | Estilo dominou identidade | Reforçar preservação: "while maintaining exact facial features" |
| **Cores da referência não transferem** | Gemini interpretou semanticamente | Adicionar cores específicas no prompt: "warm golden palette, burnt sienna" |
| **Resultado genérico** | Falta de especificidade | Usar vocabulário técnico: KB-prompting-heuristics.md Seções 3-5 |
| **Placeholder cinza retorna** | operation_mode errado | Verificar se é "generate_images" (não "analysis") |
| **LoadImage falha** | Imagem não está em input/ | Mover imagem para D:\ComfyUI\input\ |
| **Multi-ref não funciona** | max_images não aumentado | Setar max_images para número de referências |
| **Chat mode perde contexto** | >15-20 turnos | Recomeçar com clear_history=true |
| **Batch timeout** | Muitas imagens simultâneas | Reduzir batch_count ou usar sync=false + polling |

---

## CROSS-REFERENCES

### Knowledge Bases
- **KB-capabilities.md** Seção 6: Image Input (como referências funcionam)
- **KB-capabilities.md** Seção 7: 38 Task Presets (Tasks 1-4 são style transfer específicos)
- **KB-prompting-heuristics.md** Seção 2: Master Template (estrutura base)
- **KB-prompting-heuristics.md** Seção 5: Film Stock Vocabulary (para estilo fotográfico)
- **KB-prompting-heuristics.md** Seção 7: Negative Semântico (como evitar artefatos)
- **KB-prompting-heuristics.md** Seção 8: Edição Iterativa (refinamento via chat)

### Related Tasks
- `generate-image.md` → Geração base (usar este task quando NÃO há referência de estilo)
- `generate-from-reference.md` → Img2img genérico
- `create-brand-asset.md` → Aplicação prática de style transfer para branding

### Banana Task Presets (IFTaskPromptManager)
| Task ID | Nome | Uso |
|---------|------|-----|
| **Task 2** | Style Transfer (Identity-Preserving) | Transferir estilo mantendo identidade |
| **Task 20** | DSLR-Style Photo Upgrade | Elevar qualidade fotográfica |

---

## DELEGAÇÃO PARA SPECIALISTS

### Quando Delegar

| Situação | Delegar Para | Motivo |
|----------|-------------|--------|
| Prompt não está ótimo | @lyra (Prompt Architect) | Crafting de prompts narrativos |
| Workflow complexo precisa de ajuste | @forge (Workflow Engineer) | Construção/debug de workflows |
| Resultado precisa de QA | @iris (Quality Inspector) | Validação visual, refinamento |

### Handoff para Lyra

```yaml
context:
  - Usuário quer style transfer
  - Imagem de referência: [descrição]
  - Conteúdo desejado: [descrição]
  - Intensidade: subtle/balanced/heavy
inputs:
  - style_reference.jpg
  - KB-prompting-heuristics.md (Seções 2, 5, 7)
expected_output:
  - Prompt narrativo otimizado para style transfer
```

---

## SAVE SUCCESSFUL WORKFLOWS

Após resultado bem-sucedido:

```
mcp__comfyui__save_template
  Params: {
    name: "style_transfer_[estilo]_[aspectRatio]",
    workflow: {JSON completo},
    description: "Style transfer para [caso de uso]. Tested with [modelo]."
  }
```

**Também salvar note:**
```
mcp__comfyui__save_note
  Params: {
    topic: "style-transfer-success",
    content: "Estilo [X] transferido para [Y]. Prompt: [prompt]. Settings: temp 1.0, batch 4, aspect 1:1.",
    tags: ["style-transfer", "template", "success"]
  }
```

---

*Style Transfer Task v2.0.0 | ComfyUI Ops Squad*
*Multi-reference support | Chat refinement | 38 Banana Task presets*
*Criado: 2026-02-06 | Quality: 9/10*
