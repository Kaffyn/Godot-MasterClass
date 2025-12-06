# Machi Class: Shaders & Materiais

> **Instrutor:** Machi
> **Objetivo:** Perder o medo da linguagem de Shaders (GLSL/GDShader) e entender como criar efeitos visuais (VFX) que rodam na GPU.

---

## 1. O que é um Shader?

É um pequeno programa que roda para CADA pixel (Fragment Shader) ou CADA vértice (Vertex Shader) do seu objeto, milhões de vezes por segundo.
Por rodar na GPU, é extremamente rápido.

### Tipos Principais (CanvasItem vs Spatial)

- **CanvasItem (2D):** Para Sprites, UI, ColorRects.
- **Spatial (3D):** Para Meshes 3D.
- **Particles:** Para definir comportamento de partículas (chuva, fogo).

---

## 2. A Estrutura Básica

Crie um `ShaderMaterial` num Sprite e clique em "New Shader".

```glsl
shader_type canvas_item;

// Uniforms são variáveis que você controla pelo Inspector (GDScript -> Shader)
uniform vec4 nova_cor : source_color;
uniform float intensidade : hint_range(0.0, 1.0);

void fragment() {
    // COLOR: A cor do pixel atual na tela
    // TEXTURE: A imagem original
    // UV: A coordenada (0,0 a 1,1) do pixel na imagem

    vec4 cor_original = texture(TEXTURE, UV);

    // Exemplo: Pintar tudo de vermelho mantendo o alpha
    vec4 cor_final = cor_original;
    cor_final.rgb = mix(cor_original.rgb, nova_cor.rgb, intensidade);

    COLOR = cor_final;
}
```

---

## 3. Efeitos Práticos (Receitas de Bolo)

### A. Hit Flash (Piscar Branco ao levar dano)

Clássico de jogos retrô.

```glsl
shader_type canvas_item;

uniform bool active = false;

void fragment() {
    vec4 tex_color = texture(TEXTURE, UV);

    if (active) {
        // Define a cor como Branco Puro, mantendo a transparência original
        COLOR = vec4(1.0, 1.0, 1.0, tex_color.a);
    } else {
        COLOR = tex_color;
    }
}
```

_Como usar:_ No script do inimigo, ao levar dano:
`material.set_shader_parameter("active", true)`
(Espere 0.1s)
`material.set_shader_parameter("active", false)`

### B. Sway (Vento em Grama/Árvore 2D)

Usa o Vertex Shader para mover os vértices superiores enquanto fixa os inferiores.

```glsl
shader_type canvas_item;

uniform float speed = 1.0;
uniform float strength = 5.0;

void vertex() {
    // Mover apenas se o vértice estiver no topo (UV.y < 0.5 aproximadamente)
    // VERTEX.x += sin(TIME * speed) * strength * (1.0 - UV.y);

    // Em GDShader 2D, VERTEX é local.
    VERTEX.x += sin(TIME * speed) * strength * (1.0 - UV.y);
}
```

---

## 4. Visual Shader Editor

Se você não gosta de código, a Godot tem um editor visual de nós (similar ao Blender ou Unreal).

- Crie um `VisualShader`.
- Arraste nós como `Input > UV`, `VectorOp > Add`, `Texture`.
- Conecte a saída no `Output > Color`.

**Dica:** É ótimo para prototipar, mas shaders escritos em código costumam ser mais fáceis de organizar e versionar no Git a longo prazo.

---

## 5. Screen Reading Shaders (Distorção/Blur)

Para fazer efeitos que afetam o que está "atrás" do objeto (como vidro fosco ou onda de calor), usamos `SCREEN_TEXTURE`.

```glsl
shader_type canvas_item;

uniform sampler2D screen_texture : hint_screen_texture, filter_linear_mipmap;
uniform float amount : hint_range(0.0, 5.0);

void fragment() {
    vec4 color = texture(screen_texture, SCREEN_UV, amount); // O 3º param é o nível de Blur (LOD)
    COLOR = color;
}
```

Nota: Em Godot 4, é necessário declarar o `hint_screen_texture`.

---

## 6. Performance

- **Evite `if/else` complexos** dentro de shaders se possível. (Branching na GPU pode ser caro, embora placas modernas lidem bem).
- **Math is cheap:** Senos, cossenos e multiplicações são quase grátis.
- **Texture Lookups:** Ler texturas é a parte mais "lenta". Não leia 50 texturas num shader só se não precisar.
