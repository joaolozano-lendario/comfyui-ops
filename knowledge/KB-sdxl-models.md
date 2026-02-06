# KB-sdxl-models.md - SDXL Checkpoints, LoRAs & Compatibility
# ComfyUI Ops Squad - Knowledge Base
# Version: 1.0.0
# Updated: 2026-02-06
# Sources: 7 deep research agents (520+ web sources analyzed)

---

## COMO USAR ESTE DOCUMENTO

1. **Antes de baixar modelos:** Consultar seção CHECKPOINTS para escolher
2. **Antes de montar workflow:** Consultar seção LoRA STACK para combinar
3. **Antes de rodar no DirectML:** Consultar seção COMPATIBILIDADE
4. **Sempre:** Verificar KB-system-inventory.md para estado atual

---

## 1. CHECKPOINTS SDXL RECOMENDADOS

### Tier S (Recomendação Principal)

| # | Checkpoint | Tamanho | VAE | Faces | Versatilidade | DirectML | Downloads |
|---|-----------|---------|-----|-------|---------------|----------|-----------|
| **1** | **Juggernaut XL v9 (RDPhoto2)** | 6.62 GB | Baked | 9.5/10 | 10/10 | Testado | 18M+ |
| 2 | RealVisXL V5.0 Lightning BakedVAE | 6.46 GB | Baked | 10/10 | 8/10 | Testado | 142K+/mês |
| 3 | epiCRealism XL LastFAME | 6.46 GB | Baked | 9.5/10 | 7/10 | Menos testado | ~10K |

### Juggernaut XL v9 (RECOMENDADO #1)

```yaml
arquivo: juggernautXL_v9Rdphoto2.safetensors
tamanho: 6.62 GB
vae: Baked (sem necessidade de VAE externa)
licenca: Permissiva (merge, treinar, vender outputs)
fonte: https://civitai.com/models/133005/juggernaut-xl
destino: D:\ComfyUI\models\checkpoints\

settings_otimos:
  sampler: dpmpp_2m_sde
  scheduler: karras
  steps: 25-35
  cfg: 3-6  # Mais baixo = mais realista
  resolucao_portrait: 832x1216
  resolucao_square: 1024x1024
  negative: ""  # Começar vazio, adicionar só o que ver errado

porque_escolher:
  - Modelo SDXL mais baixado do mundo (520K+ downloads únicos)
  - Melhor all-around: portraits, cinematic, street, produto
  - Baked VAE = zero problemas de VAE no DirectML
  - Maior ecossistema de LoRAs (todo LoRA é testado contra Juggernaut)
  - CFG baixo = mais realista (combina com DirectML)
  - Licença permissiva
```

### RealVisXL V5.0 Lightning (ALTERNATIVA para Retratos)

```yaml
arquivo: realvisxlV50_v50LightningBakedvae.safetensors
tamanho: 6.46 GB
vae: Baked
fonte: https://civitai.com/models/139562/realvisxl-v50
destino: D:\ComfyUI\models\checkpoints\

settings_otimos:
  sampler: dpmpp_sde  # Ou dpmpp_2m_karras
  scheduler: karras
  steps: 4-6  # Lightning = ultra rápido
  cfg: 1-2  # Lightning requer CFG baixíssimo
  resolucao: 1024x1024 ou aspect ratios SDXL

quando_usar:
  - Foco principal em PESSOAS (melhor anatomia e pele)
  - Quando velocidade importa (Lightning = 4-6 steps)
  - Cores mais vibrantes que Juggernaut
  - Melhor responsividade a detalhes (cor de cabelo, olhos)

cuidados:
  - Lightning requer settings diferentes (CFG 1-2, steps 4-6)
  - Menos versátil para cenas/produtos
  - fp32 padrão é 12.92GB (USAR FP16 sempre)
```

### epiCRealism XL LastFAME (ALTERNATIVA para Textura Máxima)

```yaml
arquivo: epicrealismXL_vxviLastfame.safetensors
tamanho: 6.46 GB
vae: Baked
fonte: https://civitai.com/models/277058/epicrealism-xl
destino: D:\ComfyUI\models\checkpoints\

quando_usar:
  - Textura de pele é prioridade absoluta (poros, pelos, SSS)
  - Close-ups editoriais e beauty shots
  - Prompts simples (não precisa de "masterpiece", "8K", etc.)

cuidados:
  - Community menor = menos LoRAs testados
  - Nomeação de versões confusa (VXVI, VXV, VXVII)
  - NÃO usar versão "Pure_fix" (CLIPs com problema)
  - Usar LastFAME ou CrystalClear
```

### Checkpoints DESCARTADOS

| Checkpoint | Motivo | Status |
|-----------|--------|--------|
| Pony Diffusion XL | Anime/stylized, NÃO fotorrealismo | EXCLUÍDO |
| DreamShaper XL | Generalista, difícil fotorrealismo | NÃO recomendado como primeiro |
| Z-Image-Turbo | Arquitetura diferente (DiT), NÃO é SDXL | INCOMPATÍVEL |

---

## 2. LoRAs RECOMENDADOS

### REGRA FUNDAMENTAL

```
LoRAs são ARQUITETURA-ESPECÍFICOS:
- LoRA SDXL → SÓ funciona com checkpoint SDXL
- LoRA SD 1.5 → SÓ funciona com checkpoint SD 1.5
- LoRA Flux → SÓ funciona com checkpoint Flux
- Misturar = erro de shape / noise / crash
```

### Stack Recomendado (Top 3 Essenciais)

| Prioridade | LoRA | Peso | Função | Tamanho |
|-----------|------|------|--------|---------|
| **1** | **Skin Realism SDXL (Reworked)** | model: 0.5, clip: 0.5 | Poros, imperfeições, anti-plástico | ~100MB |
| **2** | **Better Portraits + Better Eyes v2.0** | model: 0.6, clip: 0.6 | Olhos, nariz, cabelo, iluminação | ~150MB |
| **3** | **EasyFix (Negative LoRA)** | 1.0 no NEGATIVO | Remove artefatos AI, overfitting | ~100MB |

### Skin Realism SDXL (ESSENCIAL #1)

```yaml
nome: Skin Realism (Acne/Details) REWORKED
civitai_id: 248951
arquivo: skin_realism_sdxl.safetensors  # nome exato varia
tamanho: ~100MB
base_model: SDXL
fonte: https://civitai.com/models/248951
destino: D:\ComfyUI\models\loras\

strength_model: 0.5  # NUNCA 1.0 (causa deformidades)
strength_clip: 0.5
trigger_words: "Detailed natural skin and blemishes without-makeup and acne"

o_que_faz:
  - Poros visíveis e naturais
  - Imperfeições de pele (blemishes, acne leve)
  - Remove "pele de plástico" do AI
  - Textura orgânica sem post-processing

stats: 17.9K downloads, 5/5 rating
```

### Better Portraits + Better Eyes v2.0 (ESSENCIAL #2)

```yaml
nome: Better Portraits - Better Eyes v2.0
civitai_id: 1492801
arquivo: better_portraits_eyes_v2.safetensors  # nome exato varia
tamanho: ~150MB
base_model: SDXL
fonte: https://civitai.com/models/1492801
destino: D:\ComfyUI\models\loras\

strength_model: 0.5-0.8
strength_clip: 0.5-0.8
trigger_words: "sharp eyes, detailed eyes, detailed pupils, iris detail"

o_que_faz:
  - Olhos com detalhes de íris e reflexo
  - Textura de testa e nariz
  - Detalhes de cabelo
  - Iluminação facial e color grading
  - Cobre face E olhos em um único LoRA (eficiente)

stats: Alta popularidade, 5/5 rating
```

### EasyFix Negative LoRA (ESSENCIAL #3)

```yaml
nome: EasyFix [Negative LoRA]
civitai_id: 149316
arquivo: EasyFix.safetensors
tamanho: ~100MB
base_model: SDXL
fonte: https://civitai.com/models/149316
destino: D:\ComfyUI\models\loras\

# ATENÇÃO: Este LoRA vai no PROMPT NEGATIVO
uso: "<lora:EasyFix:1> (overfit style:1.0)"  # No NEGATIVE prompt
strength: 1.0

o_que_faz:
  - Remove padrões repetitivos de AI
  - Corrige detalhes nonsensicais
  - Elimina manchas faciais de overfitting
  - Anti-AI artifact removal

stats: 666 reviews, 5/5 rating
```

### Stack Situacional (Adicionar Conforme Necessidade)

| Prioridade | LoRA | Quando Usar | Peso |
|-----------|------|------------|------|
| 4 | **DetailTweaker XL** | Boost global de qualidade | model: 0.5-1.0 |
| 5 | **PhotorealTouch** | Peach fuzz, subsurface scattering | model: 0.5-0.7 |
| 6 | **Touch of Realism V2** | Look cinematográfico, bokeh | model: 0.5 |
| 7 | **Touch of Grain** | Film grain analógico | model: 0.3-0.5 |
| 8 | **Better Faces Cultures** | Rostos não-ocidentais | model: 0.4 |
| 9 | **Expressions XL** | Expressões faciais específicas | model: 0.5-0.7 |

### DetailTweaker XL

```yaml
civitai_id: 122359
downloads: 390,374
rating: 5/5 (41,637 ratings)
tamanho: 217.87 MB
trigger_words: "detailed", "detail", "enhancer"
peso_range: -3 a +3 (positivo = mais detalhe)
peso_recomendado: 0.7-1.5
nota: Slider de detalhe GERAL (não específico para face)
uso: Base layer antes de LoRAs específicos
```

### Touch of Realism V2

```yaml
civitai_id: 1705430
trigger_words: "touch-of-realismV2"
peso: 0.5
o_que_faz: Bokeh, profundidade, comportamento de lente real
melhor_para: Look cinematográfico, editorial
```

### Touch of Grain

```yaml
civitai_id: 1789604
trigger_words: "touch-of-grain, film grain, grainy texture"
peso: 0.3-0.5
o_que_faz: Film grain, textura analógica, calor vintage
melhor_para: Estética fotográfica, anti-digital
```

---

## 3. REGRAS DE STACKING

### Limites

```yaml
maximo_loras: 3-4 (acima = degradação)
budget_total_peso: ~2.0 (soma de todos strength_model)
ordem: skin/style → detail → effects → negative(EasyFix)
teste: Sempre validar cada LoRA sozinho antes de stackar
controle: Manter render sem LoRA no mesmo seed para comparação
```

### Guia de Pesos por Tipo

| Tipo de LoRA | strength_model | strength_clip | Notas |
|-------------|---------------|---------------|-------|
| Textura de pele | 0.4-0.6 | 0.4-0.6 | Acima de 0.8 = deformidades |
| Detalhe de olhos/face | 0.5-0.7 | 0.5-0.7 | Balanceado |
| Film grain/estilo | 0.2-0.5 | 0.2-0.5 | Sutil é melhor |
| Personagem/identidade | 0.7-1.0 | 0.5-0.7 | Modelo alto, clip moderado |
| Negative LoRA (EasyFix) | 1.0 | N/A | No prompt negativo |
| Stacking (3+ LoRAs) | 0.4-0.7 cada | 0.4-0.7 cada | Reduzir individual |

### Combinações Proibidas

```
NUNCA combinar:
- FaceNaturalizer + Detail enhancers (se cancelam)
- 5+ LoRAs simultâneos (resultados imprevisíveis)
- LoRA SDXL + Checkpoint SD 1.5 (crash/noise)
- strength_model > 0.8 para skin LoRAs (deformidades)
```

---

## 4. COMPATIBILIDADE DIRECTML

### Por Arquitetura

| Arquitetura | DirectML | VRAM Mínimo | Velocidade | Recomendação |
|-------------|----------|-------------|------------|--------------|
| **SD 1.5** | Funciona bem | 8GB | ~15-30s/img | Sweet spot para 8GB |
| **SDXL** | Funciona (lento) | 12GB+ | ~2-5min/img | Possível com --lowvram |
| **Flux** | **NÃO FUNCIONA** | N/A | N/A | Crash confirmado |
| **Z-Image-Turbo** | Não testado | 12-16GB | Desconhecido | Risco alto |

### Resolução Máxima por VRAM (SDXL)

| VRAM | Resolução Máxima | LoRAs | img2img |
|------|-----------------|-------|---------|
| 8GB | 512x512 com --lowvram | 0-2 | Provavelmente OOM |
| 12GB | 1024x1024 | 3-5 | Funciona batch=1 |
| 16GB | 1024x1024 confortável | 10+ | Funciona |

### Flags de Lançamento Recomendados

```bash
# Para SDXL no DirectML:
python main.py --directml --lowvram --use-split-cross-attention --force-fp16 --fp16-unet --cpu-text-encoder --preview-method none

# Para SD 1.5 no DirectML:
python main.py --directml --medvram --use-split-cross-attention --force-fp16 --fp16-unet --preview-method none
```

### Samplers Seguros no DirectML

| Sampler | Status | Notas |
|---------|--------|-------|
| **euler** | Mais seguro | Recomendado para começar |
| **dpmpp_2m** | Funciona | Universal, boa qualidade |
| **dpmpp_2m_sde** | Funciona (mais lento) | Melhor qualidade |
| dpmpp_sde | Funciona | Mais lento que 2m |
| lcm | Funciona | Para modelos Lightning |
| ddim | **QUEBRADO com SDXL** | Só funciona com SD 1.5 |
| *_gpu variants | **CRASH** | Evitar completamente |

### Bugs Conhecidos DirectML

| Bug | Impacto | Workaround |
|-----|---------|------------|
| Detecção VRAM errada | ComfyUI vê 1GB em placa de 16GB | `--disable-smart-memory` |
| VAE decode preto | Funciona 1x depois fica preto | Reiniciar ComfyUI entre runs |
| bf16 crash | Qualquer operação bf16 falha | `--force-fp16` sempre |
| IP-Adapter hang | Workflow trava no KSampler | Evitar IP-Adapter no DirectML |
| Session reuse lento | Mudar resolução entre runs degrada performance | Manter resolução consistente |

---

## 5. RESOLUÇÕES SDXL

Todas devem totalizar ~1 megapixel.

| Aspecto | Largura | Altura | Uso |
|---------|---------|--------|-----|
| 1:1 | 1024 | 1024 | Avatar, social media |
| 16:9 | 1344 | 768 | Banner, thumbnail |
| 9:16 | 768 | 1344 | Stories, mobile |
| 4:3 | 1216 | 832 | Paisagem |
| **3:4** | **832** | **1216** | **Portrait (mais usado)** |
| 4:5 | 896 | 1152 | Instagram post |
| 5:4 | 1152 | 896 | Paisagem média |

---

## 6. NEGATIVE PROMPTS

### Universal (Fotorrealismo)

```
(worst quality, low quality:1.4), blurry, jpeg artifacts, bad anatomy,
deformed, mutated, disfigured, text, watermark,
(cartoon, anime, illustration:1.3), 3d render, cgi, plastic skin,
airbrushed, extra fingers, mutated hands, poorly drawn hands,
poorly drawn face, distorted face
```

### Adições para Portrait

```
double face, two faces, two heads, duplicate, cloned face,
asymmetric face, uneven eyes, crooked nose, bad teeth,
open mouth, tongue out
```

### Com EasyFix (Adicionar ao negativo)

```
<lora:EasyFix:1> (overfit style:1.0),
(worst quality, low quality:1.4), blurry, jpeg artifacts,
bad anatomy, deformed, text, watermark,
(cartoon, anime, illustration:1.3), 3d render, plastic skin,
extra fingers, mutated hands, distorted face
```

---

## 7. REALISTIC SNAPSHOT LoRA (Z-Image-Turbo)

### INCOMPATÍVEL COM SDXL

```yaml
status: NÃO USAR com SDXL
motivo: Treinado para Z-Image-Turbo (arquitetura DiT, 6B params)
base_model: Z-Image-Turbo (Alibaba Tongyi Lab)
arquitetura: S3-DiT / Lumina (NÃO é U-Net como SDXL)

para_usar_precisaria:
  - z_image_turbo_bf16.safetensors (~11.5 GB)
  - qwen_3_4b.safetensors (~8 GB)
  - ae.safetensors (~335 MB, Flux VAE)
  total_download: ~20 GB

veredicto: Excelente LoRA (5/5 rating, 11K+ downloads)
           mas ecossistema incompatível com nosso setup SDXL.
           Considerar APENAS se migrar para Z-Image-Turbo no futuro.
```

---

## 8. DOWNLOAD PLAN

### Fase 1: Fundação (~7GB)

```
[ ] Juggernaut XL v9 RDPhoto2
    Arquivo: juggernautXL_v9Rdphoto2.safetensors
    Tamanho: ~6.62 GB
    Fonte: CivitAI 133005
    Destino: D:\ComfyUI\models\checkpoints\
```

### Fase 2: LoRAs Core (~450MB)

```
[ ] Skin Realism SDXL (Reworked)
    Fonte: CivitAI 248951
    Destino: D:\ComfyUI\models\loras\

[ ] Better Portraits - Better Eyes v2.0
    Fonte: CivitAI 1492801
    Destino: D:\ComfyUI\models\loras\

[ ] EasyFix Negative LoRA
    Fonte: CivitAI 149316
    Destino: D:\ComfyUI\models\loras\
```

### Fase 3: LoRAs Suplementares (~500MB)

```
[ ] DetailTweaker XL
    Fonte: CivitAI 122359
    Destino: D:\ComfyUI\models\loras\

[ ] Touch of Realism V2
    Fonte: CivitAI 1705430
    Destino: D:\ComfyUI\models\loras\

[ ] Touch of Grain
    Fonte: CivitAI 1789604
    Destino: D:\ComfyUI\models\loras\
```

### Fase 4: Infraestrutura (Opcional)

```
[ ] ComfyUI-Impact-Pack (custom node, para FaceDetailer)
[ ] YOLOv8 Face (detection model, para Impact-Pack)
```

### Total de Download

| Fase | Tamanho | Prioridade |
|------|---------|-----------|
| Fase 1 | ~7 GB | CRÍTICA |
| Fase 2 | ~450 MB | ALTA |
| Fase 3 | ~500 MB | MÉDIA |
| Fase 4 | ~200 MB | BAIXA |
| **Total** | **~8.2 GB** | |

---

## 9. PRIMEIRO TESTE (Após Downloads)

```yaml
checkpoint: juggernautXL_v9Rdphoto2.safetensors
resolucao: 832x1216  # Portrait
sampler: dpmpp_2m_sde
scheduler: karras
steps: 25
cfg: 4.0
negative: ""  # Começar vazio

prompt_teste: >
  Professional DSLR photograph of a 35-year-old business woman
  in a modern office, wearing a navy blazer, natural soft lighting
  from large window, shallow depth of field, Canon EOS R5 85mm f/1.4,
  natural skin texture visible, warm color temperature

sequencia_teste:
  1. Checkpoint sozinho (sem LoRA) - verificar que funciona
  2. + Skin Realism (0.5) - comparar textura
  3. + Better Eyes (0.6) - comparar olhos
  4. + EasyFix no negativo (1.0) - comparar artefatos
  5. Stack completo - validar resultado final
```

---

*KB-sdxl-models.md - Checkpoints, LoRAs & Compatibility*
*Consolidado de 7 pesquisas profundas (520+ fontes web)*
*Criado: 2026-02-06*
