# Machi Class: Game Feel & Juice

> **Instrutor:** Machi
> **Objetivo:** Unificar os conceitos de movimento, som e interface para criar uma experiência de usuário polida, responsiva e satisfatória. O "Juice" não é um extra, é um requisito.

---

## Animação & Motion

> **Objetivo:** Dominar as ferramentas de movimento da Godot. Entender a diferença entre `AnimationPlayer` e `Tweens` e quando usar cada um.

---

### 1. AnimationPlayer vs Tweens

| Ferramenta          | Melhor Uso                                                                                  | Exemplo                                                                              |
| :------------------ | :------------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| **AnimationPlayer** | Animações complexas, visuais, com keyframes fixos, sincronia de áudio e chamadas de função. | Walk cycles, Ataques, Cutscenes, UI complexa de entrada.                             |
| **Tweens**          | Animações procedurais, interpolações numéricas, movimentos simples "Fire-and-forget".       | Fade in/out de um item coletado, Tremer a câmera, Barra de vida descendo suavemente. |

---

### 2. O Poder do AnimationPlayer

O `AnimationPlayer` é o editor de vídeo da Godot. Ele anima QUALQUER propriedade exportada.

#### Call Method Track (A arma secreta)

Além de animar Posição e Rotação, você pode animar Funções.

- **Cenário:** Um ataque de espada.
- **Problema:** O dano deve ocorrer exatamente no frame 12, quando a espada atinge o ápice.
- **Solução:** Adicione uma "Call Method Track" no AnimationPlayer, aponte para o script do Player e insira uma chave no tempo 0.4s chamando a função `apply_damage()`.

#### AnimationTree (Estados de Animação)

Para personagens, não controle o AnimationPlayer via código (`play("run")`). Use uma `AnimationTree`.

- **StateMachine:** Cria um grafo visual (Idle <-> Run <-> Jump).
- **BlendSpace1D/2D:** Mistura animações baseado em vetores (Ex: Idle mistura com Walk que mistura com Run dependendo da velocidade 0 a 100).

---

### 3. Tweens (Scripted Animation)

Tweens são leves e poderosos para "Game Feel".
Em Godot 4, Tweens não são mais Nodes. São objetos criados on-the-fly.

```gdscript
func collect_item():
    # Animação de "pulo" do item antes de sumir
    var tween = create_tween()

    # Sobe e Escala
    tween.tween_property(self, "position:y", position.y - 50, 0.3).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_OUT)
    tween.parallel().tween_property(self, "scale", Vector2(0, 0), 0.3).set_delay(0.2)

    # Callback ao terminar
    tween.tween_callback(queue_free)
```

#### Easing e Transições

O segredo do polimento ("Juice") está nas curvas.

- **Linear:** Chato, robótico.
- **Sine/Cubic:** Suave, natural.
- **Elastic/Bounce:** Cartunesco, divertido.

Use `set_trans()` e `set_ease()` em todo Tween.

---

### 4. Screen Shake (A Arte de Tremer)

Screen Shake não deve mover a Câmera diretamente, ou você perde o controle do "offset" do jogador.
Use um `Camera2D` com um nó filho ou propriedade dedicada para "Shake Offset".

```gdscript
# camera_shaker.gd
extends Camera2D

var shake_strength: float = 0.0
@export var fade_speed: float = 5.0

func apply_shake(strength: float):
    shake_strength = strength

func _process(delta):
    if shake_strength > 0:
        # Reduz a força com o tempo
        shake_strength = lerpf(shake_strength, 0, fade_speed * delta)

        # Aplica offset aleatório
        offset = Vector2(
            randf_range(-shake_strength, shake_strength),
            randf_range(-shake_strength, shake_strength)
        )
```

---

### 5. Transições de Cena

Nunca troque de cena seco (`change_scene`). Use um singleton `TransitionLayer`.

1. Crie uma cena com um `ColorRect` (preto) cobrindo a tela.
2. Use `AnimationPlayer` para animar a propriedade `modulate:a` de 0 a 1 (Fade Out) e 1 a 0 (Fade In).
3. Use `await` no seu SceneLoader.

```gdscript
func change_scene(path):
    await TransitionLayer.fade_out()
    get_tree().change_scene_to_file(path)
    await TransitionLayer.fade_in()
```

---

## Áudio Dinâmico e Imersivo

> **Objetivo:** Parar de usar sons repetitivos e estáticos. Aprender a usar o `AudioStreamRandomizer` e `AudioBuses` para criar uma paisagem sonora profissional.

---

### 1. O Problema do "Efeito Metralhadora"

Em jogos de tiro ou com passos rápidos, tocar o EXATO mesmo arquivo `.wav` repetidamente cria um efeito artificial e irritante chamado "Machine Gun Effect".
O cérebro humano detecta padrões rapidamente.

**A Solução Antiga:**
Criar um script que sorteia um pitch aleatório antes de dar play.

**A Solução Godot 4:**
Usar `AudioStreamRandomizer`.

---

### 2. AudioStreamRandomizer

Este é um **Resource** especial que embrulha seus sons. Ele permite variações automáticas de Pitch (tom) e Volume sem escrever uma linha de código.

#### Como Criar:

1. No FileSystem, crie um novo Resource -> `AudioStreamRandomizer`.
2. Salve como `footstep_grass.tres`.
3. No Inspector do Resource:
   - **Streams:** Adicione seus arquivos de áudio (pode ser 1 ou vários variações de passos).
   - **Random Pitch:** Defina a escala (ex: `1.1`). Isso significa que o som pode tocar 10% mais grave ou mais agudo a cada play.
   - **Random Volume:** Defina uma variação (ex: `2.0` dB).

#### Como Usar:

No seu `AudioStreamPlayer`, em vez de arrastar o `.wav`, arraste o `footstep_grass.tres`. Pronto.

---

### 3. Sistema de Áudio Global (SoundManager)

Não coloque `AudioStreamPlayer` dentro de balas ou objetos que morrem rápido. Se o objeto for deletado (`queue_free`), o som corta.

Crie um Autoload `SoundManager`.

```gdscript
# sound_manager.gd
extends Node

# Pool de players para sons 2D (espaciais) e UI (globais)
func play_sfx(stream: AudioStream, pitch_scale: float = 1.0):
    var player = AudioStreamPlayer.new()
    player.stream = stream
    player.pitch_scale = pitch_scale
    player.bus = "SFX"
    add_child(player)
    player.play()
    # Auto-destruição após tocar
    player.finished.connect(player.queue_free)

func play_sfx_2d(stream: AudioStream, position: Vector2):
    var player = AudioStreamPlayer2D.new()
    player.stream = stream
    player.global_position = position
    player.bus = "SFX"
    # Configurar atenuação (distância máxima)
    player.max_distance = 500
    get_tree().current_scene.add_child(player)
    player.play()
    player.finished.connect(player.queue_free)
```

---

### 4. Audio Buses (Mixagem)

Nunca deixe todos os sons no Master.
Vá na aba **Audio** (parte inferior do editor) e crie Buses:

- **Master**
  - **Music** (Para trilha sonora)
  - **SFX** (Efeitos gerais)
    - **Ambience** (Vento, chuva)
    - **UI** (Cliques)
  - **Voice** (Diálogos)

**Por que?**

1. **Menu de Opções:** Permite que o jogador baixe só a música.
2. **Ducking (Sidechain):** Você pode adicionar um efeito "Compressor" no Bus da Música e fazer ele baixar o volume automaticamente quando o Bus de Voz tiver sinal. (Estilo rádio locutor).
3. **Reverb Local:** Crie um Bus "ReverbCave". Quando o player entrar na caverna, redirecione os sons dele para esse Bus via script (`player_audio.bus = "ReverbCave"`).

---

### 5. Música Dinâmica (Interativa)

Para trocar de música (Exploração -> Combate) sem cortes bruscos, use `AnimationPlayer` ou `Tweens` para controlar o volume dos Buses ou dos Players.

```gdscript
func enter_combat_mode():
    var tween = create_tween()
    # Fade out exploração
    tween.tween_property($MusicExploration, "volume_db", -80.0, 2.0)
    # Fade in combate
    tween.parallel().tween_property($MusicCombat, "volume_db", 0.0, 2.0)
```

Ou use o recurso **Interactive Music** se estiver usando ferramentas de middleware como FMOD/Wwise (avançado), mas para Godot puro, Crossfading é a chave.

---

## UI Profissional (Themes & Containers)

> **Objetivo:** Criar interfaces que se adaptam a qualquer resolução de tela e são fáceis de reestilizar globalmente. Proibido posicionar botões "na mão".

---

### 1. A Regra de Ouro: Containers

Nunca use `Position` (x, y) para elementos de UI. Use **Containers**.
A Godot tem um sistema de layout poderoso (similar ao Flexbox da Web).

#### Os Três Mosqueteiros:

1. **`HBoxContainer`:** Alinha filhos horizontalmente.
2. **`VBoxContainer`:** Alinha filhos verticalmente.
3. **`GridContainer`:** Alinha em grade (colunas fixas).

#### Containers de Ajuste:

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

### 2. Size Flags e Anchors

Dentro de um Container, o filho tem propriedades de Layout:

- **Expand:** Ocupar todo o espaço vazio disponível.
- **Fill:** Esticar o conteúdo para preencher o espaço.
- **Shrink Center:** Ficar no meio.

Use **Anchors** (Âncoras) apenas para o nó RAIZ da sua UI (ex: o `Control` principal do HUD).

- **Full Rect:** Ancora nos 4 cantos da tela.

---

### 3. O Poder dos Themes (`.theme`)

Imagine mudar a fonte de **todos** os botões do jogo de uma vez.
Isso é feito com `Theme`.

#### O Fluxo de Trabalho:

1. Crie um recurso `main_theme.tres`.
2. Vá nas Project Settings -> GUI -> Theme -> Custom e defina ele como padrão.
3. Abra o editor de Theme.
4. Adicione o tipo `Button`.
5. Mude a `Font`, `Font Color`, e os `StyleBox` (Normal, Hover, Pressed).

**StyleBox:**
É o que define a "cara" de um painel ou botão.

- **StyleBoxFlat:** Cor sólida, bordas arredondadas, sombra simples. (Ótimo para protótipos e UI moderna/flat).
- **StyleBoxTexture:** Usa uma imagem (9-patch slice) para bordas complexas (Pixel Art, Medieval).

#### Variações de Tipo (Type Variations)

E se eu quiser um botão "Perigo" que é vermelho, mas o padrão é azul?
Não mude a cor no nó!

1. No seu Theme, adicione um novo tipo (clique no `+`).
2. Nomeie como `DangerButton` (Base Type: Button).
3. Configure o `StyleBoxNormal` para vermelho.
4. No Inspector do seu botão, em `Theme Type Variation`, escreva `DangerButton`.

Agora você tem um sistema semântico de estilos.

---

### 4. Fonts (LabelSettings)

Em Godot 4, `Label` tem uma propriedade especial `LabelSettings` (Resource).
Isso permite criar presets de tipografia: `Header1.tres`, `BodyText.tres`, `Tooltip.tres`.

- Crie esses resources e reutilize. Não configure tamanho de fonte em cada Label individualmente.

---

### 5. Control Nodes Essenciais

- **`TextureRect`:** Para imagens puras (Logos, Ícones). Tem controle de escala melhor que Sprite2D para UI.
- **`NinePatchRect`:** Para molduras que esticam sem distorcer as bordas.
- **`RichTextLabel`:** Texto com formatação BBCode (`[b]Negrito[/b]`, `[rainbow]Arco-íris[/rainbow]`, `[img]icone.png[/img]`).
- **`ProgressBar` / `TextureProgressBar`:** Barras de vida/loading.

---

### 6. Inputs de UI

Lembre-se que `_input()` pega tudo, mas `_gui_input()` pega eventos específicos daquele controle.
Para botões, sempre use os sinais:

- `pressed()`
- `mouse_entered()` / `mouse_exited()` (para Tooltips ou sons de hover).
