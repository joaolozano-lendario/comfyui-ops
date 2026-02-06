# SVG to Image - Layout Programatico para Diffusion

Cria SVG programaticamente (texto, formas, layout) e aplica diffusion para estilizar.

---

## WORKFLOW

### 1. Briefing
- Que criar: logo concept, layout de banner, diagrama, texto estilizado, pattern
- Elementos: texto, formas geometricas, icones, composicao
- Estilo final: realista, artistico, neon, metalico, etc.
- Dimensoes do output

### 2. Gerenciar Fontes (se necessario)
Listar fontes disponiveis:
```
mcp__comfyui__list_fonts
```

Baixar fonte do Google Fonts:
```
mcp__comfyui__download_font (source: { type: "google", family: "Montserrat", weight: 700 })
```

Fontes recomendadas por uso:
| Uso | Fonte | Weight |
|---|---|---|
| Headlines bold | Montserrat | 700-900 |
| Elegante/serif | Playfair Display | 400-700 |
| Tech/moderno | Inter | 400-700 |
| Handwritten | Caveat | 400 |
| Fantasy/maps | Cinzel | 400-700 |

### 3. Criar SVG
Construir SVG com elementos precisos:
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1024 1024">
  <!-- Fundo -->
  <rect width="1024" height="1024" fill="#0A0A0A"/>
  <!-- Elementos -->
  <text x="512" y="512" text-anchor="middle"
        font-family="Montserrat" font-size="72"
        font-weight="bold" fill="#FFD44A">
    TEXTO AQUI
  </text>
</svg>
```

Dicas SVG:
- Usar `viewBox` para controlar proporcoes
- `text-anchor="middle"` para centralizar texto
- Cores em hex
- Formas: `rect`, `circle`, `ellipse`, `path`, `polygon`

### 4. Renderizar SVG
```
mcp__comfyui__render_svg (
  svg: "<svg>...</svg>",
  width: 1024,
  height: 1024,
  background: "#0A0A0A",
  fonts: [{ name: "Montserrat", family: "Montserrat" }]
)
```
Retorna filename no input folder do ComfyUI.

### 5. Buscar Workflow img2img
```
mcp__comfyui__search_templates (taskType: "img2img")
```
Ou:
```
mcp__comfyui__get_example_workflow (name: "Image-to-Image (img2img)")
```

### 6. Configurar Denoise
| Objetivo | Denoise | Efeito |
|---|---|---|
| Preservar layout (adicionar textura) | 0.3-0.4 | SVG reconhecivel, ganha estilo |
| Transformacao moderada | 0.5-0.6 | Elementos originais guiam mas resultado e criativo |
| Interpretacao livre | 0.7-0.9 | SVG como inspiracao, resultado bem diferente |

### 7. Craftar Prompt
Descrever o estilo visual desejado SOBRE o layout do SVG:
- "metallic gold text with reflections, dark luxurious background, cinematic lighting"
- "neon glowing text on wet city street at night, cyberpunk aesthetic"
- "hand-painted watercolor style, soft organic textures, artistic"

### 8. Validar e Executar
```
mcp__comfyui__validate_workflow (workflow: JSON)
mcp__comfyui__run_workflow (
  workflow: JSON com imagem de input = SVG renderizado,
  name: "svg_descritivo",
  sync: true,
  imageFormat: "jpeg",
  imageQuality: 90
)
```

### 9. Iterar
- SVG nao aparece: reduzir denoise para 0.3
- Muito rigido: aumentar denoise para 0.6+
- Fonte errada: verificar nome da fonte no render_svg
- Texto ilegivel: usar Flux (melhor para texto) ou manter denoise baixo

---

## PARAMETROS

| Parametro | Default | Descricao |
|---|---|---|
| svg | (obrigatorio) | Conteudo SVG completo |
| width | 1024 | Largura do render |
| height | 1024 | Altura do render |
| background | transparent | Cor de fundo do SVG |
| denoise | 0.4 | Quanto transformar o SVG |
| style_prompt | (obrigatorio) | Estilo visual a aplicar |

---

## TROUBLESHOOTING

| Problema | Solucao |
|---|---|
| Fonte nao renderiza | Verificar `list_fonts`, baixar se faltando |
| SVG vazio/branco | Verificar viewBox e coordenadas dos elementos |
| Texto cortado | Ajustar font-size ou viewBox |
| Resultado perde layout | Reduzir denoise para 0.3-0.4 |
| Cores erradas | Verificar fill/stroke no SVG |
