# KB-PROMPT-PATTERNS: Patterns, Anti-Patterns & Ingredients Library
## ComfyUI Ops Squad | Lyra Knowledge Base
### Extracted from: Production analysis of high-performing prompts vs typical AI prompts

---
type: KB
priority: P0
squad: comfyui-ops
agent: prompt-architect
created: 2026-02-06
version: 1.0.0
always_load_before: [generate-image, generate-from-reference, create-brand-asset, create-banner, create-thumbnail, batch-variations]
---

## COMO USAR ESTE DOCUMENTO

1. **Antes de escrever QUALQUER prompt:** Reler os 9 Layers (seção 1)
2. **Para evitar imagens artificiais:** Consultar Anti-Patterns (seção 3)
3. **Para ingredientes:** Usar as Variable Libraries (seção 4-8)
4. **Para testar qualidade:** Aplicar Compression Test (seção 9)

---

## META-PRINCÍPIO

```
Prompts típicos descrevem como uma imagem PARECE.
Prompts efetivos descrevem o que uma imagem É.

"Parece" = superfície → imagens tecnicamente competentes mas emocionalmente vazias
"É" = profundidade → imagens que parecem capturadas de um momento real

A diferença: instalar um MODELO SITUACIONAL COMPLETO
(física, psicologia, dinâmica social, causalidade narrativa)
e deixar o output visual EMERGIR desse modelo.
```

---

## 1. THE 9-LAYER ARCHITECTURE

Todo prompt efetivo opera em 9 camadas simultâneas. Prompts típicos operam em 1-2.

```
LAYER 0: GENRE / FORMAT
  Que categoria de imagem é essa?
  (editorial, mirror selfie, meme, documentary, paparazzi, etc.)
  Isso seta o template estético master.

LAYER 1: STORY / CONTEXT
  Por que essa imagem existe? O que aconteceu antes/depois?
  Quem tirou? Pra quem foi tirada?
  Isso gera coerência causal.

LAYER 2: SUBJECT(S)
  Não só aparência, mas AÇÃO e EXPRESSÃO.
  Expressão descrita como COMPORTAMENTO, não adjetivo.
  ("soft gaze toward phone screen" NÃO "beautiful expression")

LAYER 3: ENVIRONMENT
  Espaço físico com EVIDÊNCIA de habitação/uso.
  Objetos que contam histórias, não objetos que decoram.
  ("unmade bed, clothes on floor" NÃO "cozy bedroom")

LAYER 4: MATERIALITY
  Descrições cross-modal de textura.
  Visual + tátil + peso + temperatura + sinal social.
  Comportamento do material sob a iluminação específica.

LAYER 5: LIGHT SOURCE + BEHAVIOR
  De ONDE a luz vem (não só qualidade).
  Como ela interage com materiais na cena.
  ("harsh on-camera flash" NÃO "dramatic lighting")

LAYER 6: CAPTURE DEVICE + CONTEXT
  Que câmera/phone/device, e POR QUE esse device.
  O device implica um genoma estético inteiro.

LAYER 7: CALIBRATED IMPERFECTION
  Falhas apropriadas ao contexto.
  Não ruído aleatório, mas evidência de condições reais.

LAYER 8: VIEWER RELATIONSHIP
  Quem está olhando isso e qual é a relação
  com o sujeito? Isso molda tudo.

LAYER 9: NEGATIVE SPACE
  O que essa imagem NÃO É.
  Quais armadilhas estéticas evitar.
```

---

## 2. THE 7 PRINCIPLES

### P1: DIEGETIC PROMPTING
Tudo no prompt deve existir DENTRO do mundo da imagem. Não quebre a quarta parede com instruções de renderização ("8K", "octane render") ou julgamentos ("stunning", "beautiful"). Fique dentro da história.

### P2: CAUSAL CHAINS > ATTRIBUTE LISTS
```
FRACO:  subject=woman, hair=messy, room=messy, mood=intimate
FORTE:  ela acordou, se sentiu bonita, sentou no chão, tirou selfie
        (cabelo bagunçado, quarto bagunçado e intimidade são CONSEQUÊNCIAS)
```

### P3: CROSS-MODAL TRIANGULATION
Descreva cada elemento importante por múltiplos canais sensoriais:
```
VISUAL:  "long-pile, halo-like under light"
TÁTIL:   "soft, fuzzy"
PESO:    "heavy"
TÉRMICO: "warm"
SOCIAL:  "expensive"
```
5 pontos de dados triangulando um material > 1 ponto com especificidade máxima.

### P4: CALIBRATED IMPERFECTION
Imagens perfeitas parecem AI. Imperfeição aleatória também parece AI. Especifique o TIPO EXATO de imperfeição que existiria no contexto descrito:
- Selfie de espelho: qualidade iPhone, roupas no chão
- Meme: grain de câmera descartável, ISO alto
- Editorial fashion: flash direto na câmera (escolha estética deliberada)

### P5: VIEWER IMPLICATION
Toda foto tem um viewer implícito. Stock photos implicam nenhum viewer. Imagens boas implicam um viewer específico.
```
Mirror selfie: O viewer é a pessoa pra quem ela mandou (íntimo)
Fashion editorial: O viewer está lendo uma revista (avaliativo)
Meme: O viewer achou isso na internet (divertido/confuso)
```

### P6: DEVICE AS AESTHETIC GENOME
O device de captura não é credencial. É encoding comprimido de um sistema visual inteiro.
"iPhone mirror selfie" contém:
- Distorção wide-angle de câmera frontal
- HDR computacional
- Color science específica
- Iluminação de tela no rosto
- Reflexo de espelho
- Contexto social (casual, espontâneo, íntimo)

### P7: NEGATIVE SPACE DEFINITION
Defina o que a imagem NÃO É:
- "feels REAL not staged"
- "not directly at lens"
- "Internet Meme, NOT editorial"

---

## 3. ANTI-PATTERNS (O QUE MATA REALISMO)

### AP1: RESOLUTION FETISH ❌
```
RUIM: "8K, ultra-detailed, hyperrealistic, Unreal Engine, octane render"
```
São keywords de pipeline de renderização, não descrições de imagem. Uma selfie de iPhone DEVE ter resolução menor que um editorial medium format.

**Fix:** Substitua keywords de resolução por device/contexto que implica o nível apropriado.

### AP2: ADJECTIVE STACK ❌
```
RUIM: "stunning, beautiful, gorgeous, breathtaking, ethereal"
```
Palavras semanticamente idênticas. Ocupam tokens sem adicionar informação visual.

**Fix:** Substitua adjetivos avaliativos por descrições físicas específicas. "Stunning" não significa nada. "Cheekbones that catch light at the edge" significa algo.

### AP3: CAMERA-AS-CREDENTIAL ❌
```
RUIM: "shot on Hasselblad H6D, 85mm f/1.4, Kodak Portra 400"
```
Trata câmera/lente/film como badges de prestígio. A pessoa não sabe POR QUE escolheu Hasselblad vs Mamiya.

**Fix:** Descreva o OUTCOME visual que quer. Ou use nomes de equipamento como labels de sistemas estéticos que você realmente entende.

### AP4: THE MISSING STORY ❌
```
RUIM: "A woman standing on a cliff overlooking the ocean at sunset"
```
Sem história. Sem razão pra ela estar lá. Sem contexto social. Resultado = stock photo.

**Fix:** Responda "por que essa pessoa existe nesse espaço nesse momento?" antes de descrever o visual.

### AP5: THE FLATNESS PROBLEM ❌
```
RUIM: "beautiful woman, white dress, beach, golden hour"
```
Descrição single-layer. 4 substantivos. Sem hierarquia, profundidade, textura, imperfeição, emoção, relação com viewer, contexto social, especificidade material. Opera em 1 camada vs as 9 necessárias.

### AP6: OVER-STYLING IMPERFECTION ❌
```
RUIM: "artfully messy hair, perfectly imperfect"
```
Imperfeição genérica é tão AI quanto perfeição. Especifique o TIPO de bagunça.

**Fix:** "lots of face-framing pieces escaped, falling around face, the 'just threw it up' hair that actually looks perfect"

### AP7: EMPTY BEAUTY DESCRIPTORS ❌
```
RUIM: "gorgeous woman", "stunning model", "breathtaking beauty"
```
Zero informação visual. O modelo não sabe o que "gorgeous" PARECE.

**Fix:** Descreva FEATURES específicas: "sharp cheekbones, full lips, deep brown eyes with gold flecks visible in the iris, warm olive skin"

---

## 4. VARIABLE LIBRARY: TEXTURES & MATERIALS

### Fabrics (Cross-Modal)
| Material | Visual | Tátil | Peso | Térmico | Social |
|----------|--------|-------|------|---------|--------|
| Angora/Mohair | halo-like under light, fuzzy pile | incredibly soft, pillow-like | heavy, substantial | warm, insulating | expensive, luxurious |
| Thin ribbed cotton | ribbed lines visible, slightly see-through | soft, familiar, worn | light, barely there | cool, breathable | casual, intimate, everyday |
| Raw silk | subtle sheen, catches light in waves | smooth with slight texture grain | medium, fluid | cool touch, warming | refined, understated wealth |
| Worn leather | creased, patina visible, matte to semi-gloss | supple, broken-in | medium-heavy | cool to touch | rebellious, experienced |
| Wet fabric | clings, translucent, darker shade | cold, uncomfortable, sticky | heavier than dry | cold | vulnerable, dramatic |
| Chunky knit | visible stitch pattern, cable texture | cozy, substantial, chunky | heavy | very warm | homey, approachable |
| Satin/Silk charmeuse | high sheen, liquid drape, catches every light | slippery, cool, weightless | very light | cool | sensual, formal, evening |
| Denim (worn) | faded at stress points, whiskers at hips | stiff but familiar | medium | neutral | everyday, American |

### Skin Textures
| Context | Description |
|---------|-------------|
| Close-up beauty | visible pores on nose/cheekbones, fine peach fuzz catching sidelight, natural variation in skin tone across face |
| Golden hour outdoor | warm natural glow, subtle sheen from humidity, micro-freckles visible on nose bridge |
| Flash photography | glossy highlight on forehead/cheekbones, pores visible in harsh light, skin appears more textured |
| Moody/dark | luminous where light hits, skin almost disappearing into shadow, veins visible on hands/wrists |
| Beach/water | salt crystals on skin catching light, water droplets on ankles, sand grains on feet |
| Post-workout | natural sheen across forehead and collarbones, flushed cheeks, slightly reddened nose tip |

### Hair Textures
| Type | Specific Description |
|------|---------------------|
| Messy bun | "loosely pulled back, casual, lots of face-framing pieces escaped, falling around face, the 'just threw it up' hair that actually looks perfect" |
| Wet-look sleek | "pulled back in sleek wet-look style, exposing bone structure, every strand controlled" |
| Wind-blown | "strands crossing face, catching light individually, not styled chaos but real wind movement" |
| Slept-on | "flat on one side, volumized on the other, pillow creases still visible, genuinely messy not styled messy" |
| Beach waves | "salt-textured, expanded volume, individual curls separating, the texture you get after ocean water dries" |

---

## 5. VARIABLE LIBRARY: ENVIRONMENTS (Authenticity Markers)

### Bedrooms
| Element | Authentic | Stock/AI |
|---------|-----------|----------|
| Bed | unmade, sheets bunched, one pillow on floor | perfectly made, decorative pillows arranged |
| Floor | clothes scattered, charger cable visible | spotless, styled rug |
| Walls | fairy lights, posters taped unevenly | blank or "artfully decorated" |
| Mirror | leaning against wall, fingerprints | mounted perfectly |
| Nightstand | water glass, phone charger, random items | single candle, styled book |

### Workspaces
| Element | Authentic | Stock/AI |
|---------|-----------|----------|
| Desk | coffee ring stains, sticky notes, cables | clean surface, one monitor |
| Monitor | browser tabs open, Slack notifications | single centered window |
| Chair | jacket draped over back | empty, pristine |
| Floor | bag dropped by feet, shoes kicked off | spotless |
| Lighting | uneven — desk lamp + monitor glow + overhead | even, professional |

### Cafes
| Element | Authentic | Stock/AI |
|---------|-----------|----------|
| Table | crumbs, used napkin, condensation ring | spotless surface |
| Cup | half-drunk, lipstick mark on rim | perfect latte art, untouched |
| Background | people blurred, one person on phone, staff moving | empty or uniformly blurred |
| Sound cues | earbuds in/out, phone face-down | nothing |

---

## 6. VARIABLE LIBRARY: LIGHTING SOURCES (Not Quality)

### Real Light Sources
| Source | Visual Signature | When to Use |
|--------|-----------------|-------------|
| iPhone screen on face | cool white with slight blue, illuminates chin-up | nighttime selfies, intimate |
| Laptop screen | blueish, creates undereye shadows | late night coding, moody |
| Edison bulb desk lamp | warm amber, soft, side-directional | home office, cozy, intellectual |
| Ring light | even, flat, shadow-killing, slight catchlight circles in eyes | influencer content, beauty |
| On-camera flash | harsh, flat, hard shadows behind subject, skin appears glossy | party photos, paparazzi, editorial |
| Neon signs reflected | colored patches on skin, wet surfaces multiply reflections | nightlife, urban, cyberpunk |
| Car dashboard at night | dim green/blue from instruments, warm from street lights outside | driving scenes, transitions |
| Candle | warm, flickering, very low, creates deep shadows | intimate dinner, atmospheric |
| Window at golden hour | warm directional, long shadows, dust particles visible in beams | morning routines, lifestyle |
| Overcast through window | soft, flat, even, no shadows | "clean girl" aesthetic, minimal |

---

## 7. VARIABLE LIBRARY: CAPTURE DEVICES (Aesthetic Genomes)

### Phones
| Device | Aesthetic Genome |
|--------|-----------------|
| iPhone front camera (selfie) | slight wide-angle distortion, computational HDR, slightly warm, smoothing on skin, 0.5-1m focus distance |
| iPhone rear camera | sharper, more natural colors, slight depth effect, 2-5m sweet spot |
| Android (Samsung) | oversaturated colors, aggressive HDR, slightly cooler whites |
| Older iPhone (pre-12) | softer, less computational, more organic noise in low light |

### Professional
| Device | Aesthetic Genome |
|--------|-----------------|
| Direct flash (any camera) | harsh highlight, hard shadow directly behind, skin appears glossy/shiny, red-eye risk, paparazzi/party aesthetic |
| Disposable camera | heavy grain, color shift, light leaks, vignetting, limited dynamic range, nostalgic |
| Polaroid | soft colors, narrow dynamic range, white border implicit, vintage warmth |
| DSLR on auto | slightly over-processed, pop-up flash, amateur family photo energy |
| Medium format (Hasselblad/Phase One) | insane detail, smooth tonal gradients, shallow DOF even at moderate apertures, luxury/editorial |
| Film (Portra 400) | warm skin tones, soft highlights, grain structure, organic, timeless |
| Film (Cinestill 800T) | tungsten color shift, halation glow around bright sources, neon-friendly, night-cinematic |

---

## 8. VARIABLE LIBRARY: VIEWER RELATIONSHIPS

| Relationship | Characteristics | Example |
|-------------|-----------------|---------|
| **Intimate recipient** | Direct but soft gaze, vulnerability, private setting, "sent this to you" energy | Mirror selfie for partner |
| **Magazine reader** | Subject performs for camera, styled, aspirational, evaluative distance | Fashion editorial |
| **Social media follower** | Curated casualness, "caught in the moment" but not really, hashtag-ready | Instagram lifestyle |
| **Paparazzi subject** | Caught off-guard, flash, motion blur, cropped awkwardly, voyeuristic | Celeb candid |
| **Friend with phone** | Genuine candid, unflattering angle possible, real laughter, not posed | Party photos |
| **Professional photographer** | Subject aware of camera, collaborative, best angle, directed pose | Portraits, headshots |
| **Surveillance/found footage** | High angle, grainy, timestamp, no awareness of camera | Meme, creepy, documentary |
| **Self (mirror/reflection)** | Examining self, not performing, private, contemplative | Getting ready scenes |

---

## 9. COMPRESSION TEST

Prompt efetivo pode ser comprimido sem perder função.

### Stock Prompt (Low Information Density: ~0.3 bits/token)
```
A stunning woman in a white dress on a beach, golden hour,
shot on Hasselblad, 85mm f/1.4, Kodak Portra 400,
photorealistic, detailed skin, 8K, ultra-detailed
```
Tokens desperdiçados em redundância (stunning/photorealistic/ultra-detailed) e credenciais sem entendimento.

### Architectural Prompt (High Information Density: ~2.8 bits/token)
```
Mirror selfie, sat on bedroom floor. iPhone front camera.
Hair in a messy bun that took zero effort but looks right.
Unmade bed behind her, clothes on floor. Soft eyes looking
at the phone screen, not the lens. The kind of photo she'd
text to someone at 2am with no caption.
```
Cada frase encoda história, materialidade, contexto social, relação com viewer e estética simultaneamente.

---

## 10. PROMPT RECIPE FORMAT

Use esta estrutura para construir prompts:

```yaml
genre: [mirror selfie | editorial | paparazzi | documentary | meme | ...]
story: [por que essa imagem existe? 1 frase]
viewer: [quem está olhando e por quê?]
device: [que device capturou e por quê esse?]

subject:
  who: [descrição física ESPECÍFICA, não adjetivos avaliativos]
  doing: [ação concreta, não pose genérica]
  expression: [comportamento dos olhos/boca, não adjetivo]
  wearing:
    item: [tipo + cor + fit]
    material: [visual + tátil + peso + temperatura + social signal]

environment:
  where: [local específico]
  evidence: [objetos que provam alguém vive/usa esse espaço]
  mess: [imperfeição calibrada ao contexto]

light:
  source: [de ONDE vem a luz — objeto específico]
  behavior: [como interage com materiais da cena]

imperfection: [o que está deliberadamente "errado" nessa imagem]
not_this: [o que essa imagem NÃO é]
```

---

## 11. EXAMPLES: BEFORE → AFTER

### Example 1: Beach Portrait

**BEFORE (typical):**
```
A gorgeous woman on a beach at sunset, flowing white dress,
golden hour light, shot on Hasselblad, 85mm f/1.4, Kodak Portra 400,
photorealistic, hyper-detailed skin
```

**AFTER (architectural):**
```
She walked to the water's edge just as the last light hit the horizon.
A tall woman with deep brown skin and short natural curls, barefoot in
the shallow surf, a white linen wrap dress pressed against her body by
the onshore wind. She's not posing — she's watching a ship far out,
one hand holding the fabric at her hip. The dying sun outlines her in
gold fire. Water droplets on her ankles catch individual rays.
Phase One, 110mm, the compression flattens the ocean into a wall
of warm color behind her. Fuji Pro 400H greens in the shadows.
```

### Example 2: Tech Portrait

**BEFORE (typical):**
```
A man coding at a desk with multiple monitors, dark room, blue light,
futuristic, Matrix vibes, shot on ARRI Alexa, 35mm, Cinestill 800T,
photorealistic
```

**AFTER (architectural):**
```
3AM and he's still at it. Three monitors cast the only light in the room
— green terminal text reflected in his glasses, purple IDE syntax on his
cheekbones. He leans forward, typing something he just figured out, the
kind of lean that means the coffee went cold two hours ago. The mechanical
keyboard is the loudest thing in the apartment. A monitor light bar casts
a thin line of warm white across the desk, catching steam from a forgotten
mug. This is what 'in the zone' looks like from the outside.
ARRI Alexa Mini, 35mm Zeiss at f/2, Cinestill 800T halation bleeding
from the bright screen edges.
```

### Example 3: Intimate Selfie

**BEFORE (typical):**
```
A beautiful young woman taking a mirror selfie in her bedroom,
blonde hair, white top, casual, natural light, iPhone photo,
photorealistic
```

**AFTER (architectural):**
```
Mirror selfie from the bedroom floor. iPhone front camera,
you can tell because the text on her phone case is reversed.
Platinum blonde hair in a bun that took zero effort — face-framing
pieces falling out everywhere. White ribbed tank cropped above
the belly button, grey cotton shorts that are basically underwear.
One leg bent up, leaning back on one arm. Looking at her phone
screen, not the lens. Unmade bed behind her in the reflection,
clothes on the floor, fairy lights on the wall. The kind of photo
you'd text someone at 2am with just a sun emoji.
```

---

## CHECKLIST RÁPIDO (Antes de Cada Prompt)

```
[ ] Tem HISTÓRIA? (por que essa imagem existe?)
[ ] Tem VIEWER? (quem está olhando?)
[ ] Sujeito descrito por AÇÕES e FEATURES, não adjetivos?
[ ] Materiais descritos cross-modal? (visual + tátil + peso + social)
[ ] Luz descrita pela FONTE, não qualidade genérica?
[ ] Device implica um genoma estético?
[ ] Tem IMPERFEIÇÃO CALIBRADA ao contexto?
[ ] Define o que NÃO É?
[ ] Zero adjective stacks? (stunning, beautiful, gorgeous)
[ ] Zero resolution fetish? (8K, ultra-detailed, masterpiece)
[ ] Opera em 5+ layers dos 9?
```

---

*KB-Prompt-Patterns v1.0.0 | Lyra Knowledge Base*
*Extracted from: Production analysis + Neural Architecture deep-dive*
*Criado: 2026-02-06*
