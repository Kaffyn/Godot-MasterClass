# Godot Architecture: O Modelo Mental do Engenheiro

Muitos acham que o diferencial da Godot é só o GDScript ou ser Open Source, mas a verdade está na sua arquitetura de objetos.

Para dominar a Godot, você precisa parar de pensar em "Scripts" e começar a pensar em **Classes e Objetos**.

A Godot é inteiramente baseada em **Herança** e **Composição**.

- **Nodes** = Herança de Comportamento (Lógica)
- **Resources** = Herança de Dados (Configuração)
- **Signals** = O Padrão Observer (Comunicação Desacoplada)
- **Scenes** = Composição (Blueprints de Árvores)
- **SceneTree** = O Gerenciador de Ciclo de Vida (O Mundo)
- **Inspector** = A Interface de Injeção de Dependência

---

## 0. Herança: O DNA do seu Código

Antes de falarmos de Godot, precisamos falar de Biologia (ou quase isso).

**Herança** é a capacidade de uma Classe (Filho) receber todas as características e comportamentos de outra Classe (Pai).

Quando dizemos que `Cachorro` herda de `Animal`:

1. **Variáveis (O que ele tem):** Se `Animal` tem `vida` e `peso`, o `Cachorro` automaticamente tem `vida` e `peso`.
2. **Funções (O que ele faz):** Se `Animal` tem `comer()` e `dormir()`, o `Cachorro` sabe `comer()` e `dormir()`.
3. **Extensão:** O `Cachorro` pode adicionar coisas novas, como `latir()`.

Na Godot, isso é definido pela palavra-chave `extends`.

> **Conceito Chave:** Herança cria uma relação **"É UM"**.
>
> - O Player **É UM** CharacterBody2D.
> - O Inimigo **É UM** CharacterBody2D.
> - A Parede **É UM** StaticBody2D.

Se você herda de algo, você ganha todos os poderes daquilo de graça. Mas cuidado: herança em excesso cria rigidez. Use com sabedoria.

--

## 1. Nodes: A Cadeia de Especialização

Já entendemos que herança é "receber poderes". Na Godot, os Nodes são organizados em uma **Cadeia de Especialização**.

Não olhe para um `CharacterBody2D` como uma peça única. Olhe para a árvore genealógica dele:

1. **Object:** A base de tudo (gerencia memória).
2. **Node:** Entra na SceneTree, tem nome e parentesco.
3. **Node2D:** Ganha `position`, `rotation`, `scale`.
4. **CollisionObject2D:** Ganha formas de colisão.
5. **CharacterBody2D:** Ganha `move_and_slide()`.

Quando você faz `extends CharacterBody2D`, seu script é o próximo passo dessa evolução.

**Por que isso importa?**
Se você precisa apenas de uma posição no mundo, use `Node2D` (leve). Se precisa de física, use `CharacterBody2D` (pesado). Escolher o nó certo é escolher a "bagagem" certa de herança para carregar.

---

## 2. Resources: Polimorfismo e Dados

Resources são dados, mas também são classes. Isso permite o **Polimorfismo**: a capacidade de substituir um objeto por outro mais complexo, sem que o sistema perceba.

### O Caso do AudioStream

Um `AudioStreamPlayer` precisa de um `AudioStream` para tocar.
Normalmente, você arrasta um arquivo `.wav` (que é um `AudioStreamWAV`).

Mas a Godot tem um recurso nativo chamado **AudioStreamRandomizer**.
Ele **herda** de `AudioStream`.

Isso significa que você pode colocar um `AudioStreamRandomizer` (que contém 10 sons diferentes e varia o pitch) no lugar onde iria um simples `.wav`.

O `AudioStreamPlayer` não sabe a diferença. Ele só sabe "tocar o stream".
Isso é arquitetura robusta: você aumentou a complexidade do som (Game Feel) sem mudar uma linha de código do player.

> **Machi Way:** Use Resources para definir _comportamentos intercambiáveis_. Stats, Tabelas de Loot, e até IAs podem ser Resources.

---

## 3. Signals: O Sistema Nervoso do Jogo

Sinais são a implementação do **Observer Pattern** na Godot. Eles permitem que objetos "gritem" que algo aconteceu, sem se importar com quem está ouvindo.

### Por que usar Sinais? (Desacoplamento)

Imagine que seu `Player` precisa avisar a `HealthBar` que tomou dano.

**Jeito Errado (Acoplado):**

```gdscript
# player.gd
func take_damage(amount):
    hp -= amount
    # O Player precisa saber o caminho exato da UI. Se você mudar a UI, o jogo quebra.
    get_parent().get_node("CanvasLayer/HealthBar").update_health(hp)
```

**Jeito Certo (Sinais):**

```gdscript
# player.gd
signal health_changed(new_hp, max_hp) # Define o "grito"

func take_damage(amount):
    hp -= amount
    health_changed.emit(hp, max_hp) # Emite o sinal. O Player não sabe quem ouve.
```

### Conectando Sinais

Você pode conectar pelo Editor (aba Node) ou via Código (mais seguro e limpo).

```gdscript
# level_manager.gd
func _ready():
    var player = $Player
    var hud = $CanvasLayer/HUD

    # Conecta o sinal do player à função da HUD
    player.health_changed.connect(hud._on_player_health_changed)
```

### Await: Esperando Sinais

Sinais também servem para pausar a execução até que algo aconteça, usando `await`.

```gdscript
func attack():
    play_animation("slash")
    await animation_player.animation_finished # Espera a animação acabar
    deal_damage()
```

Isso transforma código assíncrono complexo em uma leitura linear e limpa.

## 4. Scenes: A Matrioska de Objetos

Aqui reside a maior diferença entre Godot e engines como GameMaker.

Muitos comparam Godot e GameMaker apenas pela linguagem (GDScript vs GML), concluindo erroneamente que são iguais. **Não são.**

- **GameMaker (Modelo Plano):** Você tem uma `Room` (Sala) e joga `Objects` dentro dela. Um objeto não costuma ter outros objetos dentro dele de forma estrutural.
- **Godot (Modelo Recursivo):** Tudo é um Node. Uma Cena é apenas um Node que contém outros Nodes.

Isso significa que na Godot, a composição é **Infinita e Recursiva**.

**Exemplo Prático:**
No GameMaker, seu `Player` é um objeto com um sprite desenhado nele.
Na Godot, seu `Player.tscn` é uma árvore:

- `CharacterBody2D` (O Player em si)
  - `Sprite2D` (A aparência)
  - `CollisionShape2D` (A física)
  - `AnimationPlayer` (O animador)
  - `Sword.tscn` (Uma outra cena inteira dentro do player!)
    - `Sprite2D` (A aparência da espada)
    - `Area2D` (A hitbox da espada)

Você não apenas "tem uma espada". Você tem uma **instância completa** da cena da espada dentro da cena do player.

Isso permite que você construa sistemas complexos combinando peças menores (Lego), em vez de escrever códigos gigantescos em um único objeto monolítico.

---

## 5. SceneTree: O Gerenciador de Mundos

A `SceneTree` não é apenas uma lista de objetos. Ela é a estrutura que mantém seu jogo vivo.

Muitos iniciantes acham que a "Cena Principal" é o topo de tudo. **Errado.**
Existe um nó acima de tudo, chamado **Root** (que é um `Window/Viewport`).

A estrutura real do seu jogo rodando é assim:

```text
root (Window)
├── Autoload_GlobalAudio (AudioStreamPlayer)
├── Autoload_GameManager (Node)
└── Current_Scene (Level_01)
    ├── Player
    └── Enemies
```

### O Segredo dos Autoloads (Singletons)

Autoloads não são "mágica". Eles são apenas **Nodes normais** que a Godot injeta na árvore _antes_ da sua cena atual.

- **Autoloads:** São irmãos da sua fase. Eles nunca morrem (a menos que você os mate).
- **Current Scene:** É o nó que muda quando você chama `change_scene_to_file()`.

Quando você troca de fase, a Godot deleta o nó `Current_Scene` e coloca o novo no lugar. Os Autoloads continuam lá, assistindo a tudo. É por isso que usamos Autoloads para tocar música contínua ou guardar inventário entre fases.

### Ciclo de Vida (Lifecycle)

Entender a árvore é entender a ordem dos eventos:

1. `_init()`: O objeto nasce na memória (ainda fora da árvore).
2. `_enter_tree()`: O objeto foi plugado no pai.
3. `_ready()`: O objeto e **todos os seus filhos** estão prontos. (Aqui começa o jogo).
4. `_exit_tree()`: O objeto foi desconectado (mas ainda existe na RAM).

**Dica de Performance:** Adicionar e remover nós da SceneTree é custoso. Para projéteis e efeitos, use **Object Pooling** (esconder e reutilizar em vez de criar e destruir).

---

## 6. Inspector: A Interface do Designer

O Inspector não é apenas um visualizador de variáveis. Ele é a ferramenta onde você constrói a **Interface de Uso** dos seus scripts para os Game Designers (ou para você mesmo no futuro).

Um bom engenheiro cria scripts que são fáceis de configurar no Inspector.

### O Poder do @export

Use anotações para transformar variáveis simples em controles ricos:

```gdscript
# Agrupa variáveis no Inspector para organização
@export_category("Combat Stats")
@export var max_health: int = 100
@export_range(0.0, 10.0, 0.1) var attack_speed: float = 1.5 # Slider de 0 a 10

@export_group("Dependencies")
@export var weapon_data: WeaponResource # Arraste um .tres aqui
@export var hit_particles: PackedScene # Arraste um .tscn aqui
```

### Injeção de Dependência

Em vez de usar `$Nó/Filho` (que quebra se você renomear o nó), exporte a dependência:

```gdscript
@export var animation_player: AnimationPlayer
```

Agora você arrasta o nó para o slot no Inspector. Se você mover ou renomear o nó na cena, a Godot atualiza a referência automaticamente. **Zero quebras.**

> **Machi Way:** Se um script precisa de algo para funcionar, esse "algo" deve ser uma variável `@export`. Nunca assuma que um nó existe em um caminho fixo.

---

## 7. Graph: Criando suas Próprias Ferramentas

A Godot é uma engine feita com a própria Godot. Isso significa que os nós usados para criar o Visual Shader ou o Visual Script estão disponíveis para você.

Se você quer criar uma **Skill Tree**, um **Sistema de Diálogo** ou um **Behavior Tree Visual**, você não precisa desenhar retângulos e linhas na mão.

Use a família de nós de Grafo (Graph):

1. **GraphEdit:** O "canvas" infinito com grid, zoom e snap.
2. **GraphElement:** A classe base para tudo que vai dentro do grafo.
3. **GraphNode:** Os "nós" com slots de conexão (portas de entrada e saída).
4. **GraphFrame (Novo no 4.3):** Um container redimensionável para agrupar nós visualmente (como os comentários no Unreal/Blender).

### Por que isso é Arquitetura?

Porque permite que você crie **Ferramentas para sua Equipe**.
Em vez de pedir para o Game Designer editar um JSON gigante para os diálogos, você cria uma ferramenta visual usando `GraphEdit` onde ele conecta caixinhas de "Fala" e "Escolha".

---

## 8. Glossário de Nós Essenciais

Para navegar na Godot, você precisa conhecer as 4 Famílias Reais. Quase tudo herda de um desses quatro:

### 1. Node (A Alma)

Não tem posição, não tem visual. Serve para lógica pura e gerenciamento.

- **Timer:** Conta tempo e emite sinal `timeout`.
- **AnimationPlayer:** O maestro que anima propriedades de outros nós.
- **HTTPRequest:** Para falar com APIs e servidores.
- **AudioStreamPlayer:** Toca sons (sem posição 3D/2D).

### 2. Node2D (O Corpo 2D)

Tem `position` (x, y), `rotation` e `scale`. Usa pixels.

- **Sprite2D:** Mostra uma imagem.
- **CharacterBody2D:** Física controlável (Player/Inimigos).
- **Area2D:** Detecta quando algo entra nela (Gatilhos, Hitboxes).
- **Camera2D:** Define o que o jogador vê.
- **Marker2D:** Um ponto invisível para referência (ex: boca do canhão).

### 3. Node3D (O Corpo 3D)

Tem `position` (x, y, z) e `basis` (rotação complexa). Usa metros.

- **MeshInstance3D:** Mostra um modelo 3D.
- **CharacterBody3D:** Física controlável em 3D.
- **OmniLight3D / DirectionalLight3D:** Iluminação.
- **Camera3D:** O olho do mundo 3D.

### 4. Control (A Interface / UI)

Tem `anchor` e `margin`. Usa o sistema de retângulos para UI responsiva.

- **Label:** Texto.
- **Button:** Botão clicável.
- **TextureRect:** Imagem na UI.
- **ColorRect:** Retângulo de cor sólida (fundo).
- **VBoxContainer / HBoxContainer:** Organizadores automáticos de layout (Vertical/Horizontal).
