# KB-PIPELINE-REFERENCE: Pipeline de Pós-Processamento
## ComfyUI Ops Squad | Knowledge Base
### Fonte: Model Documentation + Community Benchmarks + Production Testing

---
type: KB
priority: P0
squad: comfyui-ops
created: 2026-02-06
version: 1.0.0
source: Model repos + ComfyUI docs + community benchmarks
always_load_before: [upscale-image, inpaint-edit, style-transfer, generate-image]
---

## COMO USAR ESTE DOCUMENTO

1. **Antes de pós-processar:** Consultar o pipeline de 4 estágios (seção 1)
2. **Para escolher upscaler:** Comparação na seção 3
3. **Para restaurar rostos:** Comparação na seção 2
4. **Para color grading:** Seção 4 tem referências de LUT e grain

---

## 1. O PIPELINE DE 4 ESTÁGIOS

```
ESTÁGIO 1        ESTÁGIO 2         ESTÁGIO 3          ESTÁGIO 4
GERAÇÃO    →    FACE RESTORE   →    UPSCALE     →    PÓS-PROCESSO
(Gemini API)    (CodeFormer)     (ESRGAN/Sharp)    (LUT/Grain/Vignette)

Obrigatório     Se tem rosto       Recomendado       Opcional
~5-15s          ~2-5s              ~5-15s             ~1-3s
```

### Quando Pular Estágios

| Situação | Estágio 1 | Estágio 2 | Estágio 3 | Estágio 4 |
|----------|-----------|-----------|-----------|-----------|
| Ideação rápida | Sim | Pular | Pular | Pular |
| Social media (sem rosto) | Sim | Pular | Sim | Pular |
| Social media (com rosto) | Sim | Sim | Sim | Pular |
| Portrait profissional | Sim | Sim | Sim | Opcional |
| Hero image / print | Sim | Sim | Sim | Sim |
| Batch exploratório | Sim | Pular | Pular | Pular |

---

## 2. ESTÁGIO 2: FACE RESTORATION

### 2.1 CodeFormer (Instalado)

| Aspecto | Detalhe |
|---------|---------|
| **Modelo** | codeformer.pth |
| **Path** | D:\ComfyUI\models\facerestore_models\ |
| **Node ComfyUI** | FaceRestoreCFCodeFormer |
| **Precisa checkpoint?** | NAO (autocontido) |
| **Melhor para** | Rostos gerados por IA, expressões complexas |

#### Parâmetro: fidelity

```
fidelity = 0.0  →  Máxima "beleza" (mais cosmético, menos fiel)
fidelity = 0.5  →  Balanço (bom default para marketing)
fidelity = 0.7  →  Mais fiel ao original (bom para portraits)
fidelity = 1.0  →  Máxima fidelidade (pode manter artefatos)
```

#### Recomendações por Use Case

| Use Case | fidelity | Motivo |
|----------|----------|--------|
| Avatar/headshot | 0.5 | Quer o rosto mais "limpo" possível |
| Portrait artístico | 0.7 | Manter caráter e expressão |
| Foto grupo | 0.5 | Múltiplos rostos, priorizar qualidade geral |
| Thumbnail YouTube | 0.6 | Balanço entre expressão e qualidade |
| Close-up extremo | 0.3 | Priorizar suavidade de pele |

### 2.2 GFPGAN (Não Instalado)

| Aspecto | Detalhe |
|---------|---------|
| **Status** | NAO INSTALADO |
| **Comparação com CodeFormer** | Inferior para IA-generated faces |
| **Quando instalar** | Só se CodeFormer falhar consistentemente |
| **Download** | Via ComfyUI-Manager ou manual |

### 2.3 FaceDetailer (Requer Checkpoint)

| Aspecto | Detalhe |
|---------|---------|
| **Status** | NAO DISPONIVEL (requer YOLO face model + checkpoint) |
| **Custom Node** | ComfyUI-Impact-Pack |
| **Vantagem** | Detecta + restaura faces automaticamente, melhor para grupos |
| **Desvantagem** | Requer checkpoint model (~2-7GB), mais pesado |
| **Quando instalar** | Se precisar de face inpainting avançado ou fotos de grupo frequentes |

### 2.4 Decision Matrix: Face Restore

```
ROSTO GERADO POR IA?
  ├─ Sim → CodeFormer (fidelity 0.5-0.7)
  └─ Não (foto real) → CodeFormer (fidelity 0.7-1.0)

MÚLTIPLOS ROSTOS?
  ├─ 1-2 → CodeFormer funciona bem
  └─ 3+ → Considerar FaceDetailer (precisa instalar)

CLOSE-UP EXTREMO?
  ├─ Sim → CodeFormer (fidelity 0.3-0.5)
  └─ Não → CodeFormer (fidelity 0.5-0.7)
```

---

## 3. ESTÁGIO 3: UPSCALING

### 3.1 Modelos Instalados

#### RealESRGAN_x4plus

| Aspecto | Detalhe |
|---------|---------|
| **Path** | D:\ComfyUI\models\upscale_models\RealESRGAN_x4plus.pth |
| **Fator** | 4× |
| **Tamanho** | ~64MB |
| **Node** | ImageUpscaleWithModel |
| **Melhor para** | General purpose, paisagens, objetos, cenas |
| **Ponto fraco** | Pode suavizar detalhes de pele/rosto |

#### 4x-UltraSharp

| Aspecto | Detalhe |
|---------|---------|
| **Path** | D:\ComfyUI\models\upscale_models\4x-UltraSharp.pth |
| **Fator** | 4× |
| **Tamanho** | ~64MB |
| **Node** | ImageUpscaleWithModel |
| **Melhor para** | Portraits, detalhes finos, texturas de pele, cabelo |
| **Ponto fraco** | Pode amplificar artefatos se imagem base for ruim |

### 3.2 Decision Matrix: Upscaler

```
TEM ROSTO EM DESTAQUE?
  ├─ Sim → 4x-UltraSharp (preserva textura de pele)
  └─ Não → RealESRGAN_x4plus (melhor general purpose)

PAISAGEM/ARQUITETURA?
  ├─ Sim → RealESRGAN_x4plus
  └─ Não → depende do conteúdo

PRODUTO/OBJETO?
  ├─ Sim → 4x-UltraSharp (preserva detalhes de textura)
  └─ Não → RealESRGAN_x4plus

QUALIDADE DA BASE?
  ├─ Boa → 4x-UltraSharp (amplifica detalhes)
  └─ Ruim → RealESRGAN_x4plus (mais forgiving)
```

### 3.3 Modelos Não Instalados (Para Futuro)

| Modelo | Quando Usar | Prioridade |
|--------|-------------|------------|
| 4x-AnimeSharp | Ilustrações, anime, cartoon | Baixa |
| LDSR (Latent Diffusion) | Máxima qualidade, lento | Média |
| SwinIR | Alternativa acadêmica | Baixa |
| 4x-Nomos8kSC | Fotografias realistas, 8K | Média |

### 3.4 Node: ImageUpscaleWithModel

```yaml
ImageUpscaleWithModel:
  inputs:
    upscale_model: [UpscaleModelLoader, 0]  # Modelo carregado
    image: [node_anterior, 0]                # Imagem para upscale
  output:
    IMAGE  # Imagem upscaled (4× resolução)
```

### 3.5 Resolução Resultante

| Input (Gemini) | Após 4× Upscale | Uso |
|----------------|------------------|-----|
| 512×512 | 2048×2048 | Social media HD |
| 768×768 | 3072×3072 | Print qualidade |
| 1024×1024 | 4096×4096 | Print alta resolução |
| 1024×576 (16:9) | 4096×2304 | Banner ultra-wide |
| 576×1024 (9:16) | 2304×4096 | Story alta resolução |

### 3.6 Performance e VRAM

```
DirectML (AMD/Intel):
- RealESRGAN 4× em 1024×1024: ~5-15 segundos
- UltraSharp 4× em 1024×1024: ~5-15 segundos
- Imagens maiores: tempo escala quadraticamente

DICA: Se travar por falta de VRAM:
1. Fechar outros programas usando GPU
2. Usar RealESRGAN (mais leve que UltraSharp)
3. Reduzir imagem de entrada antes do upscale
```

---

## 4. ESTÁGIO 4: POS-PROCESSAMENTO (Avancado)

### 4.1 Status Atual
```
NENHUM NODE DE POS-PROCESSAMENTO INSTALADO.
Este estágio é OPCIONAL e requer custom nodes adicionais.
Instalar via ComfyUI-Manager quando necessário.
```

### 4.2 Opções Disponíveis (Via ComfyUI-Manager)

#### ProPost (Film LUT)
- **Node:** Apply LUT to Image
- **O que faz:** Color grading via Look-Up Tables (como filtros de cinema)
- **Custom Node:** ComfyUI-ProPost
- **Quando instalar:** Quando precisar de color grading cinematográfico consistente

##### LUTs Recomendados para Marketing

| LUT | Efeito | Uso |
|-----|--------|-----|
| Teal & Orange | Contraste complementar | Ads dramáticos |
| Warm Gold | Tons dourados | Brand warm accent |
| Clean Bright | Clareza e vibração | Social media |
| Moody Dark | Sombras profundas | Conteúdo premium |
| Film Vintage | Nostalgia analógica | Storytelling |

#### Film Grain
- **O que faz:** Adiciona textura de grão de filme analógico
- **Quando usar:** Para look "autêntico" ou cinematográfico
- **Intensidade recomendada:** 0.02-0.05 (sutil) para marketing

#### Vignette
- **O que faz:** Escurece bordas da imagem, foca atenção no centro
- **Quando usar:** Portraits, hero images, product shots
- **Intensidade recomendada:** 0.1-0.3 (sutil)

#### Sharpen
- **O que faz:** Aumenta nitidez/contraste de bordas
- **Quando usar:** Após upscale para compensar leve blur
- **Cuidado:** Muito sharpen = artefatos de halo

### 4.3 Pipeline de Pós-Processamento Completo (Futuro)

```
[Upscale Output] → [LUT Color Grade] → [Film Grain] → [Vignette] → [Sharpen] → [Save]
```

**NOTA:** Este pipeline completo requer instalação de:
1. ComfyUI-ProPost (para LUT)
2. Nodes de grain/vignette/sharpen (geralmente inclusos em packs)

---

## 5. COMBINAÇÕES DE PIPELINE POR USE CASE

### 5.1 Instagram Post (Quick)
```
Gemini → RealESRGAN 4× → Save
Tempo: ~10-20s | Qualidade: 8/10
```

### 5.2 Instagram Post (Premium)
```
Gemini → CodeFormer(0.5) → 4x-UltraSharp → Save
Tempo: ~15-30s | Qualidade: 9/10
```

### 5.3 YouTube Thumbnail
```
Gemini(16:9) → RealESRGAN 4× → Save
Tempo: ~10-20s | Qualidade: 8/10
Nota: Adicionar texto em ferramenta externa (Canva/Figma)
```

### 5.4 Sales Page Hero
```
Gemini(batch:4) → [selecionar melhor] → CodeFormer(0.6) → 4x-UltraSharp → Save
Tempo: ~60-90s total | Qualidade: 10/10
```

### 5.5 Avatar Profissional
```
Gemini(1:1) → CodeFormer(0.5) → 4x-UltraSharp → Save
Tempo: ~15-30s | Qualidade: 9/10
```

### 5.6 Banner Wide (Ads)
```
Gemini(16:9) → RealESRGAN 4× → Save
Tempo: ~10-20s | Qualidade: 8/10
```

### 5.7 Product Shot
```
Gemini(4:3) → 4x-UltraSharp → Save
Tempo: ~10-20s | Qualidade: 9/10
Nota: Sem face restore (não tem rosto)
```

---

## 6. NODES BUILTIN DO COMFYUI (Relevantes)

### Image Processing

| Node | Função | Quando Usar |
|------|--------|-------------|
| ImageUpscaleWithModel | Upscale com modelo ESRGAN | Sempre que precisar aumentar resolução |
| UpscaleModelLoader | Carrega modelo de upscale | Pré-requisito do ImageUpscaleWithModel |
| ImageScale | Resize simples (bilinear/bicubic) | Crop/resize sem ML |
| ImageScaleBy | Resize por fator | Scale 0.5× para reduzir |
| ImageCrop | Crop por coordenadas | Recortar região específica |
| ImagePadForOutpaint | Padding para outpainting | Expandir canvas |
| ImageBlend | Mistura duas imagens | Composição, overlay |
| ImageCompositeMasked | Compõe com máscara | Colocar face restaurada de volta |
| ImageInvert | Inverte cores | Criar máscaras |
| ImageSharpen | Sharpen simples | Nitidez após upscale |

### Utility

| Node | Função | Quando Usar |
|------|--------|-------------|
| SaveImage | Salva em PNG | Output final |
| PreviewImage | Preview sem salvar | Debug/visualização |
| LoadImage | Carrega imagem | img2img, referência |
| LoadImageMask | Carrega máscara | Inpainting |

### Face Restore

| Node | Função | Status |
|------|--------|--------|
| FaceRestoreCFCodeFormer | CodeFormer restore | INSTALADO |
| FaceRestoreWithModel | GFPGAN restore | NAO INSTALADO |

---

## 7. REFERÊNCIA RÁPIDA: PIPELINE POR CONTEXTO

### Prompt → Pipeline Decision Tree

```
GERAR IMAGEM
│
├─ Tem rosto?
│   ├─ Sim
│   │   ├─ Close-up? → CodeFormer(0.3-0.5) → UltraSharp → Save
│   │   ├─ Medium shot? → CodeFormer(0.5-0.7) → UltraSharp → Save
│   │   └─ Grupo? → CodeFormer(0.5) → RealESRGAN → Save
│   └─ Não
│       ├─ Paisagem? → RealESRGAN → Save
│       ├─ Produto? → UltraSharp → Save
│       ├─ Abstrato? → RealESRGAN → Save
│       └─ Texto/UI? → UltraSharp → Save (texto fica mais nítido)
│
├─ Precisa de máxima qualidade?
│   ├─ Sim → Batch(4) → Selecionar → Full pipeline
│   └─ Não → Single → Pipeline adequado
│
└─ Volume alto?
    ├─ Sim → Batch(10-20) → Quick pipeline
    └─ Não → Single → Full pipeline
```

---

## 8. PROBLEMAS CONHECIDOS E SOLUÇÕES

### Upscale Amplifica Artefatos
```
CAUSA: Imagem base com artefatos visíveis
FIX: Usar RealESRGAN (mais "forgiving" que UltraSharp)
FIX: Gerar nova imagem com prompt mais específico
FIX: Aplicar face restore ANTES do upscale
```

### CodeFormer Muda Identidade
```
CAUSA: fidelity muito baixo
FIX: Aumentar fidelity para 0.7-0.8
FIX: Se identidade é crítica, usar fidelity 0.9
```

### Upscale Muito Lento/Trava
```
CAUSA: VRAM insuficiente com DirectML
FIX: Fechar navegador e outros programas
FIX: Usar imagem menor como input
FIX: Trocar para RealESRGAN (menor footprint de VRAM)
```

### Cores Mudam Após Pipeline
```
CAUSA: CodeFormer pode alterar levemente a cor
FIX: Normal e aceitável para maioria dos casos
FIX: Para cores precisas, skip face restore
FIX: Aplicar LUT final para corrigir (requer ProPost)
```

---

*KB-Pipeline-Reference - Pipeline de Pós-Processamento*
*4 estágios | 3 modelos instalados | Decision trees*
*Criado: 2026-02-06*
