# KB-CAPABILITIES: Mapa Completo do IF_Gemini + ComfyUI
## ComfyUI Ops Squad | Knowledge Base
### Fonte: Engenharia Reversa do Source Code + API Research

---
type: KB
priority: P0
squad: comfyui-ops
created: 2026-02-06
version: 1.0.0
source: gemini_node.py (1500+ linhas) + API docs + community research
always_load_before: [generate-image, create-brand-asset, create-banner, create-thumbnail]
---

## COMO USAR ESTE DOCUMENTO

1. **Antes de qualquer geração:** Verificar qual operation_mode e model usar
2. **Antes de montar workflow:** Verificar quais inputs/outputs estão disponíveis
3. **Para tasks especializadas:** Consultar os 38 presets (seção 7)
4. **Para troubleshooting:** Verificar seção de limitações e fallbacks

---

## 1. ARQUITETURA: 3 NODES REGISTRADOS

### 1.1 IFGeminiNode ("IF Gemini") — O Motor Principal

| Aspecto | Detalhe |
|---------|---------|
| **Display Name** | IF Gemini |
| **Categoria** | ImpactFrames/LLM |
| **Classe** | IFGeminiAdvanced |
| **Função** | generate_content |
| **Outputs** | STRING (text) + IMAGE (tensor batch ou placeholder cinza) |

Faz tudo: gera imagens, analisa, chat, batch, sequencial, img2img, áudio, vídeo.

### 1.2 IFTaskPromptManager ("IF Task Prompt Manager") — 38 Presets

Carrega presets de `presets/banana-tasks.json`. Cada preset é um prompt estruturado para uma task específica (style transfer, face swap, DSLR upgrade, etc).

| Input | Tipo | Descrição |
|-------|------|-----------|
| `task` | Dropdown | 38 presets disponíveis |
| `custom_instruction` | STRING | Instrução adicional do usuário |
| `append_mode` | Dropdown | `before` / `after` / `replace` |
| `raw_prompt_only` | BOOLEAN | Se true, retorna só o prompt raw |

**Output:** STRING → conecta no prompt do IFGeminiNode

### 1.3 IFPromptCombiner ("IF Prompt Combiner") — Utilidade

Concatena dois prompts. Útil para combinar system prompt + user prompt.

| Input | Tipo |
|-------|------|
| `prompt1` | STRING (required) |
| `prompt2` | STRING (optional) |
| `combine_mode` | `append` / `prepend` / `replace` |

---

## 2. OPERATION MODES

### 2.1 `generate_images` — Geração de Imagens

| Aspecto | Detalhe |
|---------|---------|
| **Quando usar** | Sempre que quiser gerar imagem |
| **Auto-mapping** | `gemini-2.5-flash` → `gemini-2.5-flash-image` automaticamente |
| **Response modalities** | `["Text", "Image"]` (setado internamente) |
| **Suporta referência** | Sim, até 16 imagens de input |
| **Batch** | Sim, 1-20 por execução |
| **Sequencial** | Sim, com memória contextual entre steps |

### 2.2 `analysis` — Análise Multimodal

| Aspecto | Detalhe |
|---------|---------|
| **Quando usar** | Analisar imagens, vídeo, áudio |
| **Output IMAGE** | Sempre placeholder cinza 1024x1024 (RGB 99,99,99) |
| **Auto-detect** | Se imagens conectadas, muda pra análise de imagem automaticamente |
| **Structured output** | Pode retornar JSON com `structured_output=True` |

### 2.3 `generate_text` — Geração de Texto

Idêntico a analysis em pipeline. Retorna texto + placeholder.

---

## 3. TODOS OS PARÂMETROS

### 3.1 Parâmetros Obrigatórios

| Parâmetro | Tipo | Default | Range | Heurística |
|-----------|------|---------|-------|------------|
| `prompt` | STRING | (template análise) | — | **NARRATIVO > KEYWORDS** (94% vs 61%) |
| `operation_mode` | Dropdown | generate_images | 3 opções | Quase sempre `generate_images` |
| `model_name` | Dropdown | gemini-2.5-flash | Ver seção 4 | Flash pra iteração, Pro pra final |
| `temperature` | FLOAT | 0.8 | 0.0-1.0 | **USAR 1.0** pra imagens (recomendação Google) |

### 3.2 Parâmetros Opcionais (CRÍTICOS)

| Parâmetro | Tipo | Default | Range | Quando Mudar |
|-----------|------|---------|-------|--------------|
| `images` | IMAGE | None | — | **IMG2IMG**: conectar imagem de referência |
| `seed` | INT | 0 | 0-4294967295 | 0 = random. Fixar pra reproducibilidade |
| `sequential_generation` | BOOLEAN | False | — | **True** pra stories/sequências coerentes |
| `batch_count` | INT | 4 | 1-20 | Exploração: 4-8. Final: 1 |
| `aspect_ratio` | Dropdown | none | 8 opções | **SEMPRE SETAR** (nunca deixar `none`) |
| `chat_mode` | BOOLEAN | False | — | **True** pra edição iterativa (15-20 turnos) |
| `clear_history` | BOOLEAN | False | — | True pra resetar conversa |
| `max_images` | INT | 6 | 1-16 | Aumentar pra multi-referência |
| `use_random_seed` | BOOLEAN | False | — | True pra máxima variação em batch |
| `api_call_delay` | FLOAT | 1.0 | 0.0-60.0 | Rate limiting entre batch calls |
| `external_api_key` | STRING | "" | — | Colar key direto (override de .env) |
| `structured_output` | BOOLEAN | False | — | True pra JSON response |
| `max_output_tokens` | INT | 8192 | 1-32768 | Aumentar pra análises longas |

---

## 4. MODELOS SUPORTADOS

### Gemini API Direta

| Model ID | Gera Imagem? | Custo/img | Melhor Para |
|----------|:---:|-----------|-------------|
| `gemini-2.5-flash` | Sim (auto-map) | ~$0.039 | Iteração rápida, exploração |
| `gemini-2.5-flash-image` | Sim | ~$0.039 | Target do auto-mapping |
| `gemini-2.5-flash-image-preview` | Sim | ~$0.039 | Explicitamente image model |
| `gemini-2.5-flash-002` | Sim (auto-map) | ~$0.039 | Versão específica |
| `gemini-2.5-pro` | Não | — | Análise de texto/imagem |
| `gemini-2.0-flash` | Não | — | Texto rápido |

### Auto-Mapping (Interno)

```
gemini-2.5-flash     → gemini-2.5-flash-image  ✅
gemini-2.5-flash-002 → gemini-2.5-flash-image  ✅
qualquer outro       → gemini-2.5-flash-image  ✅ (fallback)
```

### OpenRouter (Proxy)

| Model ID | Nota |
|----------|------|
| `google/gemini-2.5-flash-image-preview:free` | **GRÁTIS** mas só análise, não gera imagem |
| `google/gemini-2.5-flash` | Via proxy OpenRouter |
| `google/gemini-2.5-pro` | Via proxy OpenRouter |

**Limitação OpenRouter:** Chat mode cai pra single-message. Sequential degrada pra batch.

---

## 5. ASPECT RATIOS E RESOLUÇÕES

| Aspect Ratio | Resolução (WxH) | Uso |
|:---:|:---:|:---|
| `none` | 1024x1024 | **EVITAR** — sempre setar explicitamente |
| `1:1` | 1024x1024 | Social media, avatar |
| `16:9` | 1408x768 | Widescreen, banners, thumbnails |
| `9:16` | 768x1408 | Stories, mobile, vertical |
| `4:3` | 1280x896 | Landscape padrão |
| `3:4` | 896x1280 | Portrait padrão |
| `5:4` | 1024x819 | Medium landscape |
| `4:5` | 819x1024 | Instagram post |

**Como funciona:** A resolução é comunicada via texto no prompt ("Generate with dimensions 1408x768"). O modelo interpreta, não é enforcement pixel-perfect.

---

## 6. IMAGE INPUT (IMG2IMG / STYLE TRANSFER)

### Como Referências Funcionam

1. Input IMAGE tensor → converte pra PIL images (max `max_images`, default 6)
2. Resize com LANCZOS pra caber em `max(target_w, target_h)`
3. Enviado como: `[prompt_text, pil_image_1, pil_image_2, ...]`
4. Modelo usa como guia de estilo, composição ou conteúdo

### O Que Dá Pra Fazer

| Operação | Como | Qualidade |
|----------|------|-----------|
| **Style Transfer** | Imagem de estilo + prompt descritivo | Boa |
| **Img2Img** | Imagem + "modify this image to..." | Boa |
| **Multi-ref Composition** | Até 16 imagens de referência | Boa |
| **Face Consistency** | Upload ref photo + descrever cenas | Moderada |

### O Que NÃO Dá Pra Fazer

| Operação | Motivo |
|----------|--------|
| **Inpainting com máscara** | Não existe input MASK |
| **ControlNet conditioning** | Referências são contextuais, não estruturais |
| **Negative prompts** | API Gemini não suporta |

---

## 7. OS 38 TASK PRESETS (Banana Tasks)

**Arquivo:** `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\presets\banana-tasks.json`

**Protocolo de Ordenação de Imagens:**
- `IMAGE_1` = sempre o SUJEITO principal
- `ÚLTIMA imagem` = sempre o BACKGROUND
- Imagens do meio = referências opcionais (roupa, estilo, pose, props)

### Fotografia & Realismo

| # | Task | O Que Faz | Quando Usar |
|---|------|-----------|-------------|
| 17 | **Macro Photography** | Close-ups hiper-realistas com texturas | Produtos, natureza, detalhes |
| 20 | **DSLR-Style Photo Upgrade** | Eleva foto ruim pra qualidade DSLR profissional | Melhorar fotos de celular/low-res |
| 15 | **High-Angle View** | Transforma pra perspectiva aérea | Flat lays, overhead shots |
| 16 | **First-Person POV + Background Blur** | POV com bokeh | Imersão, tutoriais |
| 35 | **Professional Photo Retouching** | Remove manchas preservando marcas naturais | Retouch de retratos |
| 34 | **Old Photo Restoration** | Recorta, repara, coloriza, melhora fotos antigas | Restauração |

### Identidade & Consistência

| # | Task | O Que Faz | Quando Usar |
|---|------|-----------|-------------|
| 1 | **Background & Outfit Replace** | Troca cena + roupa preservando identidade | Marketing, e-commerce |
| 2 | **Style Transfer (Identity-Preserving)** | Aplica estilo visual mantendo o sujeito | Branding, campanhas |
| 3 | **Pose & Expression Transfer** | Copia pose/expressão de referência | Consistência entre shots |
| 4 | **Face/Head Replace** | Troca rosto mantendo iluminação | Compositing |
| 8 | **Outfit Try-On** | Virtual try-on com selfie + roupa | E-commerce fashion |

### Composição & Edição

| # | Task | O Que Faz | Quando Usar |
|---|------|-----------|-------------|
| 5 | **Prop Add/Swap** | Adiciona objetos com perspectiva correta | Product placement |
| 6 | **Text/Logo Add/Replace** | Coloca texto/logo com perspectiva | Branding em cenas |
| 9 | **Accessory Swap** | Troca acessórios mantendo face | Variações de produto |
| 10 | **Product Placement** | Produto na mão com integração natural | E-commerce, hero shots |

### Conteúdo Comercial

| # | Task | O Que Faz | Quando Usar |
|---|------|-----------|-------------|
| 12 | **Multi-Panel Montage** | Gera montagem multi-painel | Storyboards, ads |
| 13 | **Logo-Integrated Ad** | Embed logo em cenas | Branded content |
| 14 | **Product Breakdown** | Isola produtos de cenas complexas | Catálogo |
| 22 | **YouTube Thumbnail** | Design de thumbnail com personagem + texto | YouTube, cursos |
| 21 | **Instagram Nine-Grid** | Layout 9-grid com imagens complementares | Social media |

### Criativo & Artístico

| # | Task | O Que Faz | Quando Usar |
|---|------|-----------|-------------|
| 18 | **Storyboard/B-Roll** | Sequência visual multi-frame | Narrativa, vídeo |
| 24 | **Stop-Motion Puppet** | Estética handmade com felt/fabric | Branding criativo |
| 27 | **Character Design Sheet** | Board completo de design de personagem | Jogos, animação |
| 29 | **Sci-Fi Landscape** | Paisagens sci-fi vibrantes | Concept art |
| 32 | **Illustration to Figurine** | 2D → render 3D de figurine | Merchandise |

### Técnico & Analítico

| # | Task | O Que Faz | Quando Usar |
|---|------|-----------|-------------|
| 19 | **Pose Adjustment** | Altera pose/direção do olhar | Correção de composição |
| 30 | **Street View Annotation** | Anotações AR-style em screenshots reais | Navegação, turismo |
| 31 | **3D Mask Edits** | Edição volumétrica com máscaras coloridas | Modelagem 3D |
| 37 | **Image Counting** | Conta elementos e incorpora em nova imagem | Análise visual |
| 38 | **Game Manual & Character Sheet** | Páginas de manual com ilustrações | Game design |

---

## 8. GERAÇÃO SEQUENCIAL vs BATCH

### Batch (Default: `sequential_generation=False`)

```
Loop batch_count vezes:
  → Gera imagem independente com seed (base + i)
  → Sem contexto compartilhado
  → Todas acumuladas num tensor batch único
```

**Usar quando:** Explorar variações, A/B testing, gerar opções.

### Sequencial (`sequential_generation=True`)

```
Step 1: Envia prompt + referências → "Generate sequence of N images"
Step 2..N: Envia "Next image. Step X of N" + HISTÓRICO COMPLETO
  → Texto + imagens dos steps anteriores são contexto
  → Cada step sabe do anterior
```

**Usar quando:** Stories coerentes, séries com consistência, evolução progressiva.

### Heurística de Escolha

| Objetivo | Modo | batch_count |
|----------|------|-------------|
| Explorar variações | Batch | 4-8 |
| Uma imagem perfeita | Batch | 1 |
| Série coerente (antes/depois) | Sequential | 2-4 |
| Storyboard narrativo | Sequential | 4-8 |
| A/B testing de prompts | Batch | 2 |

---

## 9. CHAT MODE (Edição Iterativa)

### Como Funciona

1. `chat_mode=True` → mantém `ChatHistory` entre execuções
2. Cada execução adiciona user msg + model response ao histórico
3. Modelo tem contexto de TODAS as interações anteriores
4. `clear_history=True` reseta

### Regras de Ouro

| Regra | Impacto |
|-------|---------|
| **UMA mudança por turno** | 94% consistência (vs 67% com múltiplas) |
| **"Keep everything identical except..."** | Previne drift |
| **Recovery > Regeneration** | 89% resolve em 2-3 turnos (vs 5-7 pra refazer) |
| **Máximo ~15-20 turnos** | Depois disso, contexto degrada |

### Exemplo de Fluxo

```
Turn 1: "Retrato fotorealista de mulher com vestido vermelho, golden hour, 85mm f/1.8"
Turn 2: "Keep everything identical, but change dress to deep blue"
Turn 3: "Add subtle rain drops on shoulders, keep warm lighting"
Turn 4: "Increase background blur, more bokeh"
Turn 5: "Apply warm color grade, Kodak Portra 400"
```

---

## 10. MULTIMODAL: VÍDEO E ÁUDIO

### Vídeo

- Input via `video` (IMAGE tensor com frames > 1)
- Amostra até 16 frames uniformemente espaçados
- Resize cada frame pra max 1024px
- **Só análise** (não gera imagem a partir de vídeo)

### Áudio

- Input via `audio` (ComfyUI AUDIO dict)
- Auto-resample pra 16kHz mono WAV
- **Só análise/transcrição**

---

## 11. RESOLUÇÃO DE API KEY (Prioridade)

```
1. external_api_key no node     ← MAIS ALTA
2. GEMINI_API_KEY env var
3. Shell configs (~/.zshrc, ~/.bashrc)
4. .env files (node dir → parent → ComfyUI root)
5. Cache da última key usada    ← MAIS BAIXA
```

**Setup atual:** `.env` em `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\.env`

---

## 12. SAFETY & ERROR HANDLING

### Safety Settings

Todas 4 categorias setadas como `BLOCK_NONE` por default. Se `IMAGE_SAFETY` é triggered, a sequência para e reporta no texto.

### Fallbacks

| Erro | Comportamento |
|------|---------------|
| Sem API key | Mensagem de erro + placeholder cinza |
| Key inválida | "Invalid Gemini API key" |
| Quota excedida | Mensagem específica |
| Imagem bloqueada | "blocked for safety reasons" |
| Falha em batch step | Log erro, continua com os demais |
| Nenhuma imagem gerada | Texto de status + placeholder |

### O Placeholder Cinza

Quando geração falha silenciosamente, retorna imagem 1024x1024 com RGB (99, 99, 99). **Se viu imagem cinza = falhou.**

---

## 13. LIMITAÇÕES CONHECIDAS

| Limitação | Workaround |
|-----------|------------|
| Sem inpainting/máscara | Usar prompt: "change only the [area]" |
| Sem ControlNet | Usar imagem de referência como guia |
| Sem negative prompt | Usar "semantic negative" (descrever positivamente) |
| Aspect ratio é sugestão | Combinar texto no prompt + parâmetro |
| OpenRouter sem chat | Usar Gemini direto pra edição iterativa |
| Sem streaming | Operações são blocking |
| Sem system_instruction | Usar IFTaskPromptManager ou IFPromptCombiner |

---

## 14. CHAINING COM OUTROS NODES

### Inputs Aceitos

| Tipo | Fonte |
|------|-------|
| IMAGE (images) | LoadImage, KSampler, VAEDecode, outro IFGemini |
| IMAGE (video) | Video loaders, AnimateDiff |
| AUDIO | Audio loaders |
| STRING (prompt) | IFTaskPromptManager, IFPromptCombiner, qualquer text node |

### Outputs Conectáveis

| Tipo | Destino |
|------|---------|
| STRING (text) | Display Text, Save Text, outro LLM, IFPromptCombiner |
| IMAGE (image) | SaveImage, PreviewImage, **Upscale**, **FaceRestore**, **LUT**, outro IFGemini |

---

## 15. PATHS DE REFERÊNCIA

| Arquivo | Path |
|---------|------|
| Node principal | `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\gemini_node.py` |
| Task presets | `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\presets\banana-tasks.json` |
| API key (.env) | `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\.env` |
| Image utils | `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\image_utils.py` |
| Web UI JS | `D:\ComfyUI\custom_nodes\ComfyUI-IF_Gemini\web\js\gemini_node.js` |

---

## REGRA FINAL

```
ANTES de montar qualquer workflow:
1. Verificar operation_mode correto
2. Setar aspect_ratio SEMPRE
3. Setar temperature=1.0 pra imagens
4. Usar batch_count > 1 pra exploração
5. Ativar chat_mode pra refinamento

SE output é placeholder cinza → geração falhou silenciosamente
SE precisa de consistência → sequential_generation=True
SE precisa de máxima variação → batch + use_random_seed=True
```

---

*KB-Capabilities v1.0.0 | ComfyUI Ops Squad*
*Fonte: Source code analysis + API research*
*Criado: 2026-02-06*
