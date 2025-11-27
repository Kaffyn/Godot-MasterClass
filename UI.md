# Godot MBA: UI Profissional (Themes & Containers)

> **Instrutor:** Machi
> **Objetivo:** Criar interfaces que se adaptam a qualquer resolução de tela e são fáceis de reestilizar globalmente. Proibido posicionar botões "na mão".

---

## 1. A Regra de Ouro: Containers

Nunca use `Position` (x, y) para elementos de UI. Use **Containers**.
A Godot tem um sistema de layout poderoso (similar ao Flexbox da Web).

### Os Três Mosqueteiros:

1. **`HBoxContainer`:** Alinha filhos horizontalmente.
2. **`VBoxContainer`:** Alinha filhos verticalmente.
3. **`GridContainer`:** Alinha em grade (colunas fixas).

### Containers de Ajuste:

- **`MarginContainer`:** Adiciona "respiro" (padding) em volta do conteúdo. Essencial para não colar texto nas bordas da tela.
- **`CenterContainer`:** Centraliza o filho.
- **`PanelContainer`:** Adiciona um fundo (Background) automático ao redor dos filhos.

**Exemplo: Menu Principal**

```
CanvasLayer
└── MarginContainer (Padding 50px)
    └── VBoxContainer (Alinhamento Vertical)
        ├── Label (Título "Super Game")
        ├── Button ("Jogar")
        ├── Button ("Opções")
        └── Button ("Sair")
```

Se você mudar a resolução de 720p para 4K, esse menu continua centralizado e legível automaticamente.

---

## 2. Size Flags e Anchors

Dentro de um Container, o filho tem propriedades de Layout:

- **Expand:** Ocupar todo o espaço vazio disponível.
- **Fill:** Esticar o conteúdo para preencher o espaço.
- **Shrink Center:** Ficar no meio.

Use **Anchors** (Âncoras) apenas para o nó RAIZ da sua UI (ex: o `Control` principal do HUD).

- **Full Rect:** Ancora nos 4 cantos da tela.

---

## 3. O Poder dos Themes (`.theme`)

Imagine mudar a fonte de **todos** os botões do jogo de uma vez.
Isso é feito com `Theme`.

### O Fluxo de Trabalho:

1. Crie um recurso `main_theme.tres`.
2. Vá nas Project Settings -> GUI -> Theme -> Custom e defina ele como padrão.
3. Abra o editor de Theme.
4. Adicione o tipo `Button`.
5. Mude a `Font`, `Font Color`, e os `StyleBox` (Normal, Hover, Pressed).

**StyleBox:**
É o que define a "cara" de um painel ou botão.

- **StyleBoxFlat:** Cor sólida, bordas arredondadas, sombra simples. (Ótimo para protótipos e UI moderna/flat).
- **StyleBoxTexture:** Usa uma imagem (9-patch slice) para bordas complexas (Pixel Art, Medieval).

### Variações de Tipo (Type Variations)

E se eu quiser um botão "Perigo" que é vermelho, mas o padrão é azul?
Não mude a cor no nó!

1. No seu Theme, adicione um novo tipo (clique no `+`).
2. Nomeie como `DangerButton` (Base Type: Button).
3. Configure o `StyleBoxNormal` para vermelho.
4. No Inspector do seu botão, em `Theme Type Variation`, escreva `DangerButton`.

Agora você tem um sistema semântico de estilos.

---

## 4. Fonts (LabelSettings)

Em Godot 4, `Label` tem uma propriedade especial `LabelSettings` (Resource).
Isso permite criar presets de tipografia: `Header1.tres`, `BodyText.tres`, `Tooltip.tres`.

- Crie esses resources e reutilize. Não configure tamanho de fonte em cada Label individualmente.

---

## 5. Control Nodes Essenciais

- **`TextureRect`:** Para imagens puras (Logos, Ícones). Tem controle de escala melhor que Sprite2D para UI.
- **`NinePatchRect`:** Para molduras que esticam sem distorcer as bordas.
- **`RichTextLabel`:** Texto com formatação BBCode (`[b]Negrito[/b]`, `[rainbow]Arco-íris[/rainbow]`, `[img]icone.png[/img]`).
- **`ProgressBar` / `TextureProgressBar`:** Barras de vida/loading.

---

## 6. Inputs de UI

Lembre-se que `_input()` pega tudo, mas `_gui_input()` pega eventos específicos daquele controle.
Para botões, sempre use os sinais:

- `pressed()`
- `mouse_entered()` / `mouse_exited()` (para Tooltips ou sons de hover).
