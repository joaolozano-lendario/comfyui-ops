# KB-WORKFLOW-PATTERNS: Arquiteturas de Workflow ComfyUI
## ComfyUI Ops Squad | Knowledge Base
### Fonte: Community Research + Production Testing + Node Architecture

---
type: KB
priority: P0
squad: comfyui-ops
created: 2026-02-06
version: 1.0.0
source: ComfyUI community workflows + node documentation + production testing
always_load_before: [generate-image, create-brand-asset, create-banner, create-thumbnail, upscale-image, style-transfer]
---

## COMO USAR ESTE DOCUMENTO

1. **Antes de montar qualquer workflow:** Consultar a Decision Matrix (seção 1)
2. **Para entender conexões:** Verificar os diagramas de cada arquitetura
3. **Para batch/iteração:** Seção 6 tem patterns para alto volume
4. **Para otimizar custo:** Seção 7 tem estratégias de economia de tokens

---

## 1. DECISION MATRIX: QUAL WORKFLOW USAR

### Por Objetivo

| Objetivo | Workflow | Qualidade | Velocidade | Custo API |
|----------|----------|-----------|------------|-----------|
| Ideação rápida | Quick Generate | 7/10 | Rápido | Baixo |
| Post social media | Quick Quality | 8/10 | Rápido | Baixo |
| Portrait profissional | Professional Portrait | 9/10 | Médio | Médio |
| Banner/ad comercial | Commercial Pipeline | 9/10 | Médio | Médio |
| Máxima qualidade | Maximum Quality | 10/10 | Lento | Alto |
| Batch de variações | Batch Generator | 7-8/10 | Rápido×N | Médio |
| Edição iterativa | Chat Refine | 8/10 | Variável | Alto |

### Por Use Case Marketing

| Necessidade | Workflow | Aspect Ratio | Notas |
|-------------|----------|--------------|-------|
| Instagram post | Quick Quality | 1:1 | 1024×1024 |
| Instagram story | Quick Quality | 9:16 | 768×1365 |
| YouTube thumbnail | Commercial Pipeline | 16:9 | Precisa de texto bold |
| Facebook ad | Commercial Pipeline | 1:1 ou 4:5 | Cores vibrantes |
| Sales page hero | Maximum Quality | 16:9 | Máxima qualidade |
| Email header | Quick Quality | 3:1 (custom) | Widescreen crop |
| Avatar/headshot | Professional Portrait | 1:1 | Face restore obrigatório |
| Product shot | Commercial Pipeline | 4:3 | Background controlado |

---

## 2. WORKFLOW 1: QUICK GENERATE (Ideação)

### Quando Usar
- Exploração rápida de conceitos
- Brainstorming visual
- Primeira iteração

### Diagrama

```
[IFGeminiNode] → [PreviewImage]
```

### Configuração

```yaml
IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"    # Flash = rápido + barato
  temperature: 1.0              # Criatividade máxima
  seed: 0                       # Aleatório
  batch_count: 1
  aspect_ratio: "16:9"          # ou conforme necessidade
  keep_alive: true
```

### JSON ComfyUI

```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "PROMPT_AQUI",
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
    "class_type": "PreviewImage",
    "inputs": {
      "images": ["1", 0]
    }
  }
}
```

### Resultado Esperado
- 1 imagem em ~5-15 segundos
- Qualidade boa para validação de conceito
- Sem pós-processamento

---

## 3. WORKFLOW 2: QUICK QUALITY (Social Media)

### Quando Usar
- Posts para redes sociais
- Conteúdo que precisa ser bom mas não perfeito
- Alto volume de produção

### Diagrama

```
[IFGeminiNode] → [ImageUpscaleWithModel] → [SaveImage]
                        ↑
              [UpscaleModelLoader]
              (RealESRGAN_x4plus)
```

### Configuração

```yaml
IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"
  temperature: 1.0
  seed: 0
  batch_count: 1
  aspect_ratio: "1:1"          # Instagram
  keep_alive: true

UpscaleModelLoader:
  model_name: "RealESRGAN_x4plus.pth"

ImageUpscaleWithModel:
  # Recebe imagem do Gemini + modelo upscale
  # Output: imagem 4× maior (ex: 1024→4096)

SaveImage:
  filename_prefix: "social_post"
```

### Notas
- RealESRGAN 4× é suficiente para social media
- Se imagem tem rostos, adicionar CodeFormer ANTES do upscale
- Para Instagram Stories, mudar aspect_ratio para "9:16"

---

## 4. WORKFLOW 3: PROFESSIONAL PORTRAIT

### Quando Usar
- Avatares profissionais
- Headshots para perfis
- Qualquer imagem com rosto em destaque

### Diagrama

```
[IFGeminiNode] → [FaceRestoreCFCodeFormer] → [ImageUpscaleWithModel] → [ImageCompositeMasked] → [SaveImage]
                                                       ↑
                                            [UpscaleModelLoader]
                                            (4x-UltraSharp)
```

### Configuração

```yaml
IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"
  temperature: 0.9              # Levemente mais controlado para rostos
  seed: 42                      # Reprodutível
  batch_count: 1
  aspect_ratio: "1:1"
  keep_alive: true

FaceRestoreCFCodeFormer:
  fidelity: 0.7                 # 0.5=mais bonito, 1.0=mais fiel
  # Nota: CodeFormer não precisa de checkpoint model

UpscaleModelLoader:
  model_name: "4x-UltraSharp.pth"  # Melhor para detalhes de pele

ImageUpscaleWithModel:
  # 4× upscale após face restore

SaveImage:
  filename_prefix: "portrait"
```

### Por Que 4x-UltraSharp para Portraits
- Preserva textura de pele sem over-smoothing
- Mantém detalhes de cabelo, olhos, cílios
- RealESRGAN tende a suavizar demais rostos

### Por Que CodeFormer (e não GFPGAN)
- CodeFormer é superior para rostos gerados por IA
- Melhor preservação de identidade
- Menos artefatos em expressões complexas
- Não precisa de checkpoint model (autocontido)

---

## 5. WORKFLOW 4: MAXIMUM QUALITY (Sales Page / Hero)

### Quando Usar
- Imagem hero de sales page
- Banner principal de campanha
- Material impresso ou alta resolução
- Qualquer asset que precisa ser "o melhor possível"

### Diagrama

```
[IFGeminiNode] → [FaceRestoreCFCodeFormer]* → [ImageUpscaleWithModel] → [ImageBlend]** → [SaveImage]
                                                        ↑
                                             [UpscaleModelLoader]
                                             (4x-UltraSharp)

* Só se tiver rosto
** Opcional: blend com correção de cor
```

### Configuração

```yaml
IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"
  temperature: 0.8              # Mais controlado para resultado previsível
  seed: 42
  batch_count: 4                # Gerar 4 candidatos, escolher o melhor
  aspect_ratio: "16:9"
  keep_alive: true

# Fase 1: Face Restore (condicional)
FaceRestoreCFCodeFormer:
  fidelity: 0.6                 # Levemente mais cosmético

# Fase 2: Upscale
UpscaleModelLoader:
  model_name: "4x-UltraSharp.pth"

# Fase 3: Salvar
SaveImage:
  filename_prefix: "hero_max"
```

### Estratégia "Gerar 4, Escolher 1"
1. batch_count: 4 gera 4 imagens independentes
2. Revisar manualmente qual é a melhor
3. Pegar o seed da melhor (no filename ou metadata)
4. Re-gerar com esse seed + upscale pipeline
5. Resultado: melhor imagem possível com qualidade máxima

---

## 6. WORKFLOW 5: BATCH GENERATOR (Alto Volume)

### Quando Usar
- Criar variações de um conceito
- A/B testing visual para ads
- Gerar biblioteca de assets

### Diagrama

```
[IFGeminiNode] → [SaveImage]
  batch_count: N
```

### Configuração

```yaml
IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"
  temperature: 1.0              # Máxima diversidade
  seed: 0                       # Aleatório para cada
  batch_count: 10               # 10 variações (máx 20)
  aspect_ratio: "1:1"
  keep_alive: true

SaveImage:
  filename_prefix: "batch_v"    # batch_v_00001_, batch_v_00002_, ...
```

### Limites e Heurísticas
- **batch_count máximo:** 20 por execução
- **Custo:** Cada imagem = 1 chamada API
- **Tempo:** ~5-15s por imagem (sequencial internamente)
- **Estratégia:** batch_count: 10 → selecionar top 3 → refinar com upscale

### Batch com Pós-Processamento

Para processar batch com upscale, ComfyUI processa cada imagem do batch:

```
[IFGeminiNode] → [ImageUpscaleWithModel] → [SaveImage]
  batch_count: 5        ↑
                [UpscaleModelLoader]
```

Cada imagem do batch passa pelo upscaler individualmente.

---

## 7. WORKFLOW 6: CHAT REFINE (Edição Iterativa)

### Quando Usar
- Refinar uma imagem específica
- Fazer ajustes incrementais
- "Mais X, menos Y" iterativo

### Diagrama

```
[IFGeminiNode]  ←→  [ChatHistory]
     ↓                   ↑
[SaveImage]        (memória latente)
```

### Configuração

```yaml
IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"
  temperature: 0.7              # Mais conservador para manter coerência
  seed: 42                      # Mesmo seed para consistência
  batch_count: 1
  keep_alive: true              # CRÍTICO: mantém chat ativo

# Iteração 1: Gerar base
# prompt: "A professional headshot of a 40-year-old business woman..."

# Iteração 2: Refinar
# prompt: "Make her smile more natural, add subtle warm lighting from the left"

# Iteração 3: Ajustar
# prompt: "Change the background to a blurred modern office"
```

### Regras do Chat Refine
1. **keep_alive DEVE ser true** — sem isso, contexto se perde
2. **Máximo recomendado: 15-20 turnos** — qualidade degrada depois
3. **Referências incrementais:** "make the X more Y" funciona melhor que redescrever tudo
4. **Se perder coerência:** Resetar com seed diferente e começar novo chat
5. **Cada turno acumula contexto** — custo de tokens aumenta progressivamente

---

## 8. WORKFLOW 7: IMG2IMG (Referência Visual)

### Quando Usar
- Usar foto existente como base
- Transferir estilo de uma imagem para outra
- Manter composição mas mudar estilo

### Diagrama

```
[LoadImage] → [IFGeminiNode] → [SaveImage]
               ↑
          (prompt descreve
           transformação)
```

### Configuração

```yaml
LoadImage:
  image: "referencia.png"       # Imagem de entrada

IFGeminiNode:
  operation_mode: "generate_images"
  model: "gemini-2.5-flash"
  temperature: 0.8
  seed: 42
  batch_count: 1
  aspect_ratio: "16:9"
  keep_alive: true
  # A imagem de entrada é passada como contexto visual
  # O prompt descreve o que MUDAR, não o que manter

# Exemplo de prompt para img2img:
# "Transform this photo into a cinematic scene. Keep the same composition
#  and person, but change the lighting to dramatic golden hour with
#  volumetric fog. Add a film grain texture like Kodak Portra 400."
```

### Protocolo de Imagens (do banana-tasks.json)
```
IMAGE_1 = Subject principal (pessoa/objeto)
LAST IMAGE = Background/referência de ambiente
MIDDLE IMAGES = Referências de estilo/textura/cor
```

### Heurísticas img2img
- Prompt deve descrever a TRANSFORMAÇÃO, não a imagem original
- Gemini entende "keep X, change Y" muito bem
- Para style transfer: ser específico sobre qual estilo (filme, pintor, época)
- Aspect ratio deve combinar com imagem de entrada

---

## 9. PATTERNS DE CONEXÃO ENTRE NODES

### Pattern A: Linear Simples
```
Generate → Save
```
Uso: Ideação, draft rápido

### Pattern B: Generate + Upscale
```
Generate → Upscale → Save
```
Uso: Social media, web assets

### Pattern C: Generate + Face + Upscale
```
Generate → FaceRestore → Upscale → Save
```
Uso: Portraits, headshots, qualquer imagem com rosto

### Pattern D: Generate + Batch Select + Pipeline
```
Generate(batch:N) → Save(preview)
     ↓ (manual select)
Generate(seed:X) → FaceRestore → Upscale → Save(final)
```
Uso: Maximum quality, hero images

### Pattern E: Reference + Generate + Pipeline
```
LoadImage → Generate(transform) → FaceRestore → Upscale → Save
```
Uso: img2img, style transfer, brand consistency

### Pattern F: Task Preset + Generate + Pipeline
```
TaskPromptManager(preset) → PromptCombiner(+custom) → Generate → Pipeline → Save
```
Uso: Tasks especializadas (DSLR upgrade, product placement, etc.)

---

## 10. OTIMIZAÇÃO DE CUSTOS

### Custo por Modelo Gemini

| Modelo | Custo Input | Custo Output | Recomendação |
|--------|-------------|--------------|--------------|
| gemini-2.5-flash | Baixo | Baixo | Default para tudo |
| gemini-2.0-flash | Baixo | Baixo | Alternativa se flash falhar |
| gemini-1.5-pro | Alto | Alto | Evitar para imagem |

### Estratégias de Economia

1. **Flash sempre:** gemini-2.5-flash é o melhor custo-benefício para imagem
2. **batch_count consciente:** Gerar 4, não 20 (selecionar é mais barato que gerar)
3. **Upscale local:** Pós-processamento não custa API tokens
4. **Chat reset:** Após 10 turnos, resetar chat (contexto acumulado = mais tokens)
5. **Seed fixo para refinamento:** Não gerar novo, ajustar o existente

### Custo Estimado por Workflow

| Workflow | Chamadas API | Custo Estimado |
|----------|-------------|----------------|
| Quick Generate | 1 | ~R$0.01 |
| Quick Quality | 1 | ~R$0.01 |
| Professional Portrait | 1 | ~R$0.01 |
| Maximum Quality | 4 | ~R$0.04 |
| Batch (10) | 10 | ~R$0.10 |
| Chat Refine (5 turns) | 5+ | ~R$0.15+ |

---

## 11. TROUBLESHOOTING WORKFLOWS

### Imagem Cinza (Placeholder)
```
CAUSA: Gemini retornou texto em vez de imagem
FIX: Verificar operation_mode = "generate_images"
FIX: Verificar que prompt não é uma pergunta
FIX: Verificar que model suporta geração de imagem
```

### Conexão Quebrada
```
CAUSA: Tipo de output não combina com input
FIX: IFGeminiNode output "IMAGE" → conectar em input tipo "IMAGE"
FIX: Nunca conectar STRING output em IMAGE input
```

### Upscale Muito Lento
```
CAUSA: Imagem muito grande para GPU/DirectML
FIX: Usar RealESRGAN em vez de UltraSharp (mais leve)
FIX: Verificar se batch não está multiplicando tamanho
```

### Face Restore Não Detecta Rosto
```
CAUSA: Rosto muito pequeno na imagem
FIX: Gerar com aspect_ratio que enquadre melhor o rosto
FIX: Crop antes do face restore
FIX: Usar fidelity mais baixo (0.5)
```

### Chat Perde Coerência
```
CAUSA: Muitos turnos (>15-20)
FIX: Resetar chat (keep_alive: false, depois true)
FIX: Re-descrever tudo no próximo prompt em vez de incremental
```

---

## 12. TEMPLATES DE WORKFLOW JSON

### Template: Quick Quality
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "{{PROMPT}}",
      "operation_mode": "generate_images",
      "model": "gemini-2.5-flash",
      "temperature": 1.0,
      "seed": 0,
      "batch_count": 1,
      "aspect_ratio": "{{ASPECT_RATIO}}",
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
      "filename_prefix": "{{PREFIX}}"
    }
  }
}
```

### Template: Professional Portrait
```json
{
  "1": {
    "class_type": "IFGeminiNode",
    "inputs": {
      "prompt": "{{PROMPT}}",
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

---

## REGRA DE OURO DE WORKFLOWS

```
SIMPLES > COMPLEXO

Começar com o workflow mais simples que resolve.
Adicionar nodes APENAS quando a qualidade não é suficiente.

Quick Generate → não bom? → adicionar Upscale
Upscale ruim em rostos? → adicionar FaceRestore
Precisa de mais opções? → adicionar Batch
```

---

*KB-Workflow-Patterns - Arquiteturas de Workflow*
*7 workflows documentados | Decision matrix | JSON templates*
*Criado: 2026-02-06*
