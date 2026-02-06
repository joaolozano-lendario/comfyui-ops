# KB-PROMPTING-HEURISTICS: Como Escrever Prompts State-of-Art
## ComfyUI Ops Squad | Knowledge Base
### Fonte: Google DevBlog + Community Research + Production Testing

---
type: KB
priority: P0
squad: comfyui-ops
created: 2026-02-06
version: 1.0.0
source: Google internal testing data + 30+ community guides + API docs
always_load_before: [generate-image, create-brand-asset, create-banner, create-thumbnail, create-video]
---

## COMO USAR ESTE DOCUMENTO

1. **Antes de escrever QUALQUER prompt:** Reler a Regra de Ouro (seção 1)
2. **Para fotorealismo:** Usar o Master Template (seção 2) + Vocabulário (seções 3-5)
3. **Para evitar artefatos:** Consultar seção de Negative Semântico (seção 7)
4. **Para refinamento:** Consultar seção de Edição Iterativa (seção 8)
5. **Prompts prontos:** Seção 9 tem 10 gold standard examples

---

## 1. A REGRA DE OURO

```
NARRATIVO > KEYWORDS

Google internal testing:
- Prompts narrativos: 94% de coerência
- Listas de keywords: 61% de coerência

DIFERENÇA: 33 pontos percentuais.
```

### O Que Isso Significa na Prática

| RUIM (keywords) | BOM (narrativo) |
|-----------------|-----------------|
| "old man, wrinkles, fishing boat, sunset, 8k, realistic, masterpiece" | "A weathered fisherman in his 70s inspects his catch at golden hour. Deep wrinkles tell stories of decades at sea. Shot on 85mm f/1.8, shallow depth of field with harbor boats as creamy bokeh." |
| "woman, red dress, city, night, neon, professional" | "A confident woman in a flowing red dress strides through rain-slicked Tokyo streets. Neon reflections create a kaleidoscope on the wet pavement. Captured with a 35mm lens at f/2.8, emphasizing the environmental context." |

### Por Que Funciona

O Gemini é um modelo de linguagem nativo. Entende contexto, relações, física, narrativa. Keywords ativam features isoladas. Narrativas ativam compreensão integrada.

---

## 2. O MASTER TEMPLATE

```
A photorealistic [SHOT TYPE] of [SUBJECT], [ACTION/EXPRESSION],
set in [ENVIRONMENT]. The scene is illuminated by [LIGHTING],
creating a [MOOD] atmosphere. Captured with a [CAMERA/LENS],
emphasizing [TEXTURES/DETAILS]. [ASPECT RATIO] format.
```

### Variáveis Decodificadas

| Variável | O Que Controla | Impacto |
|----------|---------------|---------|
| **SHOT TYPE** | Enquadramento e composição | Define o "frame" visual |
| **SUBJECT** | Sujeito central (quanto mais específico, melhor) | +34% accuracy com detalhes |
| **ACTION** | O que o sujeito está fazendo | Dá vida e contexto |
| **ENVIRONMENT** | Onde (camadas: geral → específico) | Constrói o mundo |
| **LIGHTING** | Setup técnico de iluminação | **ELEMENTO MAIS IMPACTANTE** |
| **MOOD** | Emoção e atmosfera | Tom global |
| **CAMERA/LENS** | Focal length + abertura | Controla a "física" da imagem |
| **TEXTURES** | O que deve estar nítido/detalhado | Ancora realismo |

---

## 3. VOCABULÁRIO DE CÂMERA E LENTES

### Lentes de Retrato

| Lente | Efeito | Quando Usar |
|-------|--------|-------------|
| `85mm f/1.4` | Compressão facial linda, bokeh cremoso, máxima separação | Headshots close-up |
| `85mm f/1.8` | Similar ao f/1.4, um pouco mais DOF | **RECOMENDADO PADRÃO** pra retratos |
| `50mm f/2.0` | Perspectiva natural (olho humano) | Retratos ambientais, geral |
| `35mm f/2.8` | Cinematic, contexto ambiental | Documentário, street, editorial |

### Lentes Wide Angle

| Lente | Efeito | Quando Usar |
|-------|--------|-------------|
| `24mm wide angle` | Contexto amplo, leve distorção nas bordas | Arquitetura, interiores |
| `16mm ultra-wide` | Distorção dramática, cenas expansivas | Real estate, paisagens dramáticas |

### Lentes Telephoto

| Lente | Efeito | Quando Usar |
|-------|--------|-------------|
| `200mm telephoto` | Compressão pesada de fundo, isolamento extremo | Esportes, wildlife |
| `135mm` | Compressão suave, sem extremos | Fashion, eventos |

### Lentes Especiais

| Lente | Efeito | Quando Usar |
|-------|--------|-------------|
| `macro lens, f/4` | Close-up extremo, DOF finíssimo | Produtos, natureza, comida |
| `tilt-shift` | Efeito miniatura, planos de foco seletivos | Cityscapes, arquitetura |

### Abertura: O Que Cada f/ Faz

| Range | Efeito Visual |
|-------|---------------|
| `f/1.4 - f/2.0` | DOF raso, bokeh pesado, sujeito "salta" do fundo |
| `f/2.8 - f/4` | DOF moderado, algum contexto de fundo visível |
| `f/5.6 - f/8` | Maioria da imagem nítida, pouco bokeh |
| `f/11 - f/16` | Deep focus, tudo nítido (paisagens) |

---

## 4. VOCABULÁRIO DE ILUMINAÇÃO

### REGRA: Iluminação é o elemento técnico MAIS impactante pra fotorealismo.

### Iluminação Natural

| Termo | Efeito | Quando Usar |
|-------|--------|-------------|
| `golden hour light` | Quente, direcional, sombras longas | Retratos outdoor, lifestyle |
| `soft diffused window light` | Uniforme, favorecedor, sem sombras duras | Retratos indoor |
| `overcast sky` | Softbox natural, sem sombras | Fashion, editorial |
| `harsh midday sun` | Sombras fortes, alto contraste | Propositalmente unflattering/dramático |
| `blue hour light` | Cool, ethereal, pós-pôr-do-sol | Cityscapes, moody |

### Iluminação de Estúdio

| Termo | Efeito | Quando Usar |
|-------|--------|-------------|
| `Rembrandt lighting, 2:1 ratio` | Triângulo de luz na bochecha da sombra | **CLÁSSICO** retrato |
| `three-point lighting setup` | Key + fill + back light, balanceado | Profissional, corporativo |
| `butterfly lighting` | Key light direto acima da câmera | Glamour, beauty |
| `split lighting` | Metade do rosto iluminado, metade escuro | Dramático, moody |
| `rim lighting / backlight` | Halo de luz nas bordas do sujeito | Separação do fundo |
| `softbox studio lighting` | Broad, uniforme, profissional | Produtos, headshots |

### Iluminação Artística/Cinematic

| Termo | Efeito | Quando Usar |
|-------|--------|-------------|
| `chiaroscuro` | Contraste extremo luz/escuro | Fine art, Caravaggio |
| `neon reflections on wet pavement` | Cyberpunk, noturno urbano | Sci-fi, urban night |
| `practicals only` | Luz de lâmpadas, telas, velas na cena | Naturalista, interiores |
| `volumetric light / god rays` | Raios visíveis através de poeira/névoa | Atmosférico, dramático |

---

## 5. VOCABULÁRIO DE PÓS-PROCESSAMENTO E FILM STOCK

### Film Stocks (Referência no Prompt)

| Termo | Efeito | Melhor Para |
|-------|--------|-------------|
| `Kodak Portra 400 film stock` | Tons de pele quentes, grain leve | **RETRATOS** (padrão ouro) |
| `Fuji Velvia 50` | Cores super saturadas, alto contraste | Paisagens, natureza |
| `Fuji Pro 400H` | Verdes cool, cores muted | Fashion, editorial |
| `Kodak Ektar 100` | Vívido, saturado, contraste alto | Produtos, landscape |
| `Cinestill 800T` | Tungstênio, halation glow | Noturno, cinematic |
| `Ilford HP5` | P&B clássico, grain rico | Documentário, artístico |

### Estilos de Color Grading

| Termo | Efeito |
|-------|--------|
| `teal-and-orange color grading` | Look de blockbuster Hollywood |
| `muted color palette` | Editorial, fashion, understated |
| `high dynamic range` | Tons ricos em shadows e highlights |
| `desaturated with lifted blacks` | Look editorial moderno |
| `cross-processed` | Cores propositalmente shifted, experimental |

---

## 6. MODIFICADORES DE QUALIDADE — USAR COM MODERAÇÃO

### REGRA: NÃO empilhar modificadores. Um ou dois é suficiente.

| FUNCIONA | NÃO FUNCIONA |
|----------|-------------|
| `hyperdetailed skin texture` (1 modificador específico) | `8k, best quality, masterpiece, ultra HD, photorealistic` (5 empilhados) |
| `commercial photography quality` | `trending on ArtStation, award-winning, professional` |
| `visible skin imperfections, realistic detail` (+6% success rate) | Repetir "photorealistic" 3 vezes |

### Modificadores Efetivos (Pick 1-2)

| Modificador | Quando Usar |
|-------------|-------------|
| `hyperdetailed skin texture` | Close-up de rosto |
| `visible skin pores, natural imperfections` | Fotorealismo extremo |
| `commercial photography quality` | Produtos, advertising |
| `editorial fashion photography` | Fashion editorial |
| `natural hair texture, realistic eye reflections` | Âncoras de textura |
| `4K resolution` | Quando quer enfatizar detalhe |

---

## 7. NEGATIVE PROMPTING SEMÂNTICO

### O Gemini NÃO tem parâmetro `negative_prompt`. Use estas técnicas:

### Técnica 1: Reframing Afirmativo (Principal)

| Em Vez Disso | Escreva Isto |
|-------------|-------------|
| "no extra fingers" | "anatomically correct hands with five fingers each" |
| "no blurry background" | "tack-sharp background with visible architectural detail" |
| "no watermarks" | "clean, unobstructed image without overlays" |
| "no cartoonish" | "documentary photography style, photojournalistic" |
| "no artificial lighting" | "illuminated exclusively by natural ambient light" |
| "no cars" | "an empty, deserted street with no signs of traffic" |

### Técnica 2: Lista de Exclusão Breve (Para Edições Complexas)

No final de um prompt longo, adicionar:
```
Avoid: extra fingers, double pupils, heavy motion blur, duplicated facial features.
```

### Técnica 3: Correção Iterativa (Via Chat Mode)

Se artefato aparecer, NÃO regenerar. Usar:
```
"Keep everything identical but correct the hand position to holding the cup naturally"
```
→ Resolve 89% dos problemas em 2-3 turnos

---

## 8. EDIÇÃO ITERATIVA (Latent Memory)

### Como o Gemini Edita

1. **Prompt 1** → Cria um "blueprint" latente da imagem
2. **Prompt 2** → Aplica como camada transformativa sobre o latente existente
3. **Cross-referencing** → Garante continuidade entre edições
4. **Drift prevention** → Verificação cruzada com o latente original

### Heurísticas Quantificadas

| Prática | Consistência |
|---------|-------------|
| UMA mudança por turno | **94%** |
| Múltiplas mudanças simultâneas | **67%** |
| Recovery com fix específico | **89%** em 2-3 ciclos |
| Regeneração completa | **89%** em 5-7 ciclos |

### O Template de Edição

```
"Keep everything identical except [MUDANÇA ESPECÍFICA]"
```

**Exemplos:**
- "Keep everything identical except change the dress to deep blue"
- "Keep everything identical except make the shadows deeper"
- "Keep everything identical except add subtle rain drops on shoulders"

### O Que EVITAR em Edição

| NÃO | POR QUE |
|-----|---------|
| Mudar 3 coisas ao mesmo tempo | Cai de 94% pra 67% consistência |
| Reescrever o prompt inteiro | Perde o latente acumulado |
| Mais de 15-20 turnos | Contexto degrada |
| "Make it better" (vago) | Modelo não sabe O QUE melhorar |

---

## 9. 10 PROMPTS GOLD STANDARD

### 1. Retrato Clássico (Padrão Ouro)

```
A photorealistic close-up portrait of an elderly Japanese ceramicist
with deep, sun-etched wrinkles and a warm, knowing smile. He is
carefully inspecting a freshly glazed tea bowl. The setting is his
rustic, sun-drenched workshop. The scene is illuminated by soft,
golden hour light streaming through a window, highlighting the fine
texture of the clay. Captured with an 85mm portrait lens at f/1.8,
resulting in a soft, blurred background with creamy bokeh.
Hyperdetailed skin texture with visible pores. The overall mood is
serene and masterful. 3:4 portrait orientation.
```

### 2. Retrato Cinematic

```
Dramatic portrait of a person in their 30s, moody cinematic lighting
with strong key light from 45 degrees above left, subtle rim light
on hair edge. Natural skin texture with visible pores and fine lines.
Shot on Arri Alexa with 50mm lens at f/2, shallow depth of field
isolating subject from dark gradient background. Film grain texture,
commercial photography quality. 4:5 format.
```

### 3. Product Hero Shot

```
A high-resolution, studio-lit product photograph of sleek wireless
earbuds on a matte black surface. Three-point softbox lighting to
highlight textures and material finish. 45-degree camera angle to
showcase the charging port and logo embossing. Ultra-realistic,
tack-sharp focus on the logo. Subtle reflections on the surface.
1:1 square aspect ratio.
```

### 4. Sci-Fi Cinematic

```
An astronaut floating in zero gravity inside a damaged spacecraft
corridor. Emergency lights cast harsh red and white light, creating
dramatic alternating shadows across the corridor walls. Floating
debris and water droplets catch the light. 35mm lens equivalent,
f/2.8 for shallow depth of field. Teal-and-orange color grading.
16:9 cinematic widescreen.
```

### 5. Street Photography

```
Candid street photograph of a jazz musician playing saxophone on a
rainy New Orleans sidewalk at dusk. Warm glow from a nearby bar
window illuminates his weathered hands. Reflections in puddles create
a mirror world below. Shot on 35mm f/2.8 with slight motion blur on
passing pedestrians. Kodak Portra 400 film stock tones. 3:2 landscape.
```

### 6. Food Photography

```
Overhead flat lay of an artisanal sourdough bread, freshly sliced,
on a rustic wooden cutting board. Steam still rising from the warm
interior. Scattered flour dust, a vintage bread knife, and a small
pot of butter. Soft, diffused natural window light from the upper
left. Macro-level detail on the crust texture and crumb structure.
Shot with macro lens at f/4. 1:1 square format.
```

### 7. Fashion Editorial

```
High fashion editorial photograph of a model in an oversized
structured blazer walking through an empty brutalist concrete
corridor. Overcast skylight from above creates even, shadowless
illumination. Desaturated muted color palette with subtle teal
undertones. Shot on medium format camera equivalent, 80mm lens
at f/4. The model's expression is distant, contemplative.
Fuji Pro 400H film tones. 9:16 vertical.
```

### 8. Landscape Épica

```
Panoramic landscape of Patagonian mountains at sunrise. Jagged peaks
pierce low-hanging clouds, with the first golden rays illuminating
snow-capped summits. A crystal-clear lake in the foreground creates
a perfect mirror reflection. Shot on 24mm wide angle at f/11 for
deep focus across the entire frame. Fuji Velvia 50 saturation with
vibrant blues and warm golds. High dynamic range. 16:9 widescreen.
```

### 9. Ambiente Interior

```
Cozy Scandinavian living room at twilight. A person reads by the
warm glow of a single floor lamp, casting a pool of golden light.
Rain streaks down floor-to-ceiling windows showing a moody cityscape
beyond. Hygge atmosphere with wool blankets, steaming mug, and soft
textiles. Practicals-only lighting. 50mm f/2.0 capturing the
intimate scene. Warm color temperature. 16:9 format.
```

### 10. Nostalgia Digital (Compact Camera)

```
A candid photo of friends laughing at a birthday party, simulating
a compact digital camera from 2005. Lens equivalent to 28-35mm.
Aperture f/2.8. ISO 400. Shutter speed 1/60 with harsh direct flash
creating sharp shadows and slightly washed-out highlights on faces.
Auto white balance with flash color cast. Slight vignetting at
corners. Minor JPEG compression artifacts. Red-eye visible on one
person. Timestamp in bottom-right corner reading "2005.03.14". 4:3.
```

---

## 10. HEURÍSTICAS POR CASO DE USO

### Example Use Cases

| Asset | Template Recomendado | Aspect | Related Task |
|-------|---------------------|--------|-------------|
| Hero image (website) | #2 Cinematic + product | 16:9 | create-brand-asset |
| Thumbnail YouTube | #2 Cinematic + texto | 16:9 | create-thumbnail |
| Banner ads | Produto + contexto | 16:9 ou 1:1 | create-banner |
| Email header | Mood + conceito | 16:9 | create-brand-asset |
| Instagram post | #3 Product ou #7 Fashion | 1:1 ou 4:5 | create-brand-asset |
| Stories | Vertical dramatic | 9:16 | create-brand-asset |
| Avatar/foto perfil | #1 Retrato | 1:1 | generate-image |

### Por Estágio de Consciência

| Awareness | Estilo Visual | Mood | Referência |
|-----------|-------------|------|-----------|
| Unaware | Provocativo, disruptivo | Urgência | #4 Sci-Fi |
| Problem Aware | Emocional, identificável | Dor → esperança | #5 Street |
| Solution Aware | Profissional, aspiracional | Confiança | #2 Cinematic |
| Product Aware | Clean, demonstrativo | Clareza | #3 Product |
| Most Aware | Premium, exclusivo | Status | #7 Fashion |

---

## 11. TEMPERATURA: QUANDO MUDAR

| Caso | Temperatura | Por Que |
|------|-----------|---------|
| Geração de imagem | **1.0** | Google recomenda, máxima criatividade |
| Exploração de variações | **1.0** | Máxima diversidade |
| Reproduzir resultado anterior | **0.3-0.5** | Menor variação |
| Análise de imagem | **0.2-0.4** | Factual, consistente |
| JSON structured output | **0.0-0.3** | Preciso, sem criatividade |

---

## 12. ORDEM DOS ELEMENTOS NO PROMPT

Pesquisa indica que esta ordenação produz resultados mais consistentes:

```
1. Declaração de estilo/medium    → "A photorealistic photograph"
2. Shot type                      → "close-up portrait"
3. Sujeito (geral → específico)   → "elderly Japanese ceramicist with..."
4. Ação/expressão                 → "carefully inspecting a tea bowl"
5. Ambiente (geral → específico)  → "in his rustic workshop"
6. Iluminação (termos técnicos)   → "golden hour light through window"
7. Câmera/lente                   → "85mm f/1.8"
8. Âncoras de textura             → "visible skin pores, fabric weave"
9. Mood/atmosfera                 → "serene and masterful"
10. Aspect ratio                  → "3:4 portrait orientation"
```

**Regra:** Spatial/composição primeiro, estilo/textura depois.

---

## CHECKLIST RÁPIDO (Antes de Cada Prompt)

```
[ ] Formato narrativo (não keywords)?
[ ] Shot type definido?
[ ] Sujeito com 3+ detalhes específicos?
[ ] Ação/expressão descrita?
[ ] Ambiente layered (geral → específico)?
[ ] Iluminação com termos técnicos?
[ ] Lente + abertura especificados?
[ ] Pelo menos 1 âncora de textura?
[ ] Aspect ratio setado (nunca "none")?
[ ] Máximo 1-2 quality modifiers?
[ ] Sem keywords empilhadas?
```

---

*KB-Prompting-Heuristics v1.0.0 | ComfyUI Ops Squad*
*Fonte: Google DevBlog + 30+ community guides + production testing*
*Criado: 2026-02-06*
