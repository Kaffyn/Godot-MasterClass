# Fundamentos do Godot: POO, GDScript, Sinais, Nós e SceneTree

> **De:** Machi
> **Para:** Você, que está no início da jornada para se tornar um arquiteto.
>
> Não se constrói um arranha-céu sobre areia. Antes de falarmos de arquiteturas complexas, ROP, e sistemas de plugins, precisamos garantir que seu alicerce é rocha pura. Este documento é a sua fundação. Domine estes conceitos e você terá a base para construir qualquer sistema, não importa quão complexo.

---

## 1. GDScript: A Linguagem do Motor

GDScript é a linguagem nativa da Godot, projetada para ser concisa e integrada. Sua sintaxe é similar a Python, onde a indentação (o `Tab` no seu teclado) define os blocos de código.

### 1.1. Variáveis e Tipagem Estática

No Machi Class, a **Tipagem Estática não é opcional, é lei**. Ela nos dá segurança, performance e um autocomplete que funciona, economizando horas de depuração.

```gdscript
# Ruim (Dinâmico - pode ser qualquer coisa)
var vida = 100

# Bom (Estático - garantido ser um inteiro)
var vida: int = 100
var velocidade: float = 250.5
var nome: String = "Machi"
var is_vivo: bool = true
```

### 1.2. Coleções: Arrays e Dictionaries

**Arrays (Listas Ordenadas):** Para quando a ordem dos itens importa.

```gdscript
# Use Arrays tipados sempre que possível
var inventario: Array[String] = ["Espada", "Poção", "Mapa"]

func exemplo_array():
    inventario.append("Chave")
    print(inventario[0]) # Imprime "Espada"
    inventario.erase("Poção")

    # Loop (Iteração)
    for item in inventario:
        print("Tenho: " + item)
```

**Dictionaries (Chave-Valor):** Para quando você precisa de acesso rápido a um dado por um identificador único.

```gdscript
var dados_player: Dictionary = {
    "nome": "Heroi",
    "nivel": 10,
    "xp": 500
}

func exemplo_dict():
    print(dados_player["nome"])
    dados_player["xp"] += 50
```

### 1.3. Funções (Métodos)

Funções são os "verbos" do seu código. Elas devem ter um contrato claro, definindo o que recebem (parâmetros) e o que retornam.

```gdscript
# Uma função que não retorna nada
func pular() -> void:
    velocity.y = -400

# Uma função que retorna um inteiro
func somar(a: int, b: int) -> int:
    return a + b
```

**Funções Estáticas (Utilitários):**
São funções que pertencem à classe, mas não precisam de uma instância para serem chamadas. Perfeitas para lógica de cálculo pura.

```gdscript
# Em um script "MathUtils.gd"
static func calcular_dano_critico(base_damage: int) -> int:
    return base_damage * 2
```

_Chamada de outro script: `var dano = MathUtils.calcular_dano_critico(50)`_

---

## 2. O Paradigma Godot: Nós, Cenas e a Árvore

Muitos confundem estes termos. Vamos definir a hierarquia da existência na Godot.

### 2.1. Nodes (Nós)

O bloco de construção fundamental. Um nó é uma **Classe** que herda de `Node`. Ele tem:

- Nome.
- Propriedades (ex: `position`, `rotation`, `scale`).
- Callbacks de ciclo de vida (`_ready`, `_process`).
- A capacidade de ter filhos, formando uma hierarquia.

### 2.2. Scenes (Cenas)

Uma Cena (`.tscn`) **não é** um objeto. É um **Blueprint** (uma planta baixa) que descreve como uma árvore de nós e seus componentes devem ser construídos.

- `Player.tscn` é o arquivo de definição no seu disco.
- Quando você chama `minha_cena.instantiate()`, a Godot lê o blueprint e cria os objetos (Nós) na memória RAM.

### 2.3. SceneTree (A Árvore de Cena)

É o "Mundo Ativo". O gerenciador principal que executa o loop do jogo e organiza todos os nós ativos.

**Hierarquia Visual:**

```
SceneTree (O Jogo Rodando)
├── root (A Janela Principal)
│   ├── Global (Autoload)
│   ├── SoundManager (Autoload)
│   └── Level01 (Cena Atual)
│       ├── Player
│       │   └── Sword
│       └── Enemies
│           ├── Goblin1
│           └── Orc2
```

---

## 3. POO: Pensando em Objetos na Godot

### 3.1. O Mito do "Script"

Muitos iniciantes dizem: _"Vou colocar um script nesse objeto"_.
A mentalidade correta é: _"Vou transformar esse objeto em uma **instância da minha Classe**"_.

Quando você anexa um arquivo `player.gd` a um `Node`, você está **estendendo** a funcionalidade base daquele nó.
`extends CharacterBody2D` significa que a sua classe `player.gd` **É UM** `CharacterBody2D`, herdando todos os seus poderes (`move_and_slide()`, `velocity`, etc.) e adicionando novos.

### 3.2. Os 4 Pilares da POO

- **Abstração:** Esconder a complexidade. Seu `Player` não precisa saber como a `Gun` calcula a balística. Ele apenas chama `$Gun.shoot()`.
- **Encapsulamento:** Proteger os dados. Em vez de permitir `player.hp = -999`, você força o uso de uma função `player.take_damage(amount)` que contém a lógica de validação e morte.
- **Herança:** Criar especializações. Útil, mas use com moderação.
  - `Enemy` (Base) -> `Goblin` (Filho, rápido) -> `Orc` (Filho, forte).
- **Polimorfismo:** Tratar objetos diferentes da mesma maneira. Uma explosão não se importa se o que está no seu raio de alcance é um `Goblin` ou um `Orc`. Se ambos herdam de `Enemy` e têm a função `take_damage()`, a explosão pode simplesmente chamar `objeto.take_damage()` em todos.

---

## 4. Padrões de Arquitetura em Godot

### 4.1. Composição sobre Herança (O Jeito Godot)

Embora Herança seja útil, **Composição** é o superpoder da Godot. Em vez de criar uma árvore genealógica complexa (`Inimigo -> Voador -> Atirador`), nós montamos o objeto com **Componentes**.

**A Regra:** "Tem um" é melhor que "É um".

- O Player **TEM UM** `HealthComponent`.
- O Player **TEM UM** `InventoryComponent`.

Na SceneTree, isso se parece com:

```
Player (CharacterBody2D)
├── HealthComponent (Node)
├── HitboxComponent (Area2D)
└── Gun (Node2D)
```

A vantagem? Se você quiser que uma `Caixa` tenha vida, você apenas arrasta o `HealthComponent.tscn` para dentro da cena da `Caixa`, sem herança complexa.

### 4.2. `class_name`: Elevando Scripts a Tipos Globais

Use `class_name` para dar um nome global ao seu script, transformando-o em um tipo que a Godot reconhece em todo o projeto. Isso permite checagem de tipo (`is MinhaClasse`) e melhora o autocomplete.

```gdscript
# interactable.gd
class_name Interactable extends Area2D

func interact(user: Node2D):
    print(user.name + " interagiu comigo!")
```

### 4.3. Sinais: A Arte da Comunicação Desacoplada

Como os nós conversam sem criar dependências rígidas?
**Regra de Ouro:** _"O Pai manda no Filho. O Filho avisa o Pai."_

- **Call Down (Pai chama Filho):** O Pai sabe quem é o filho (`$Filho`) e pode chamar funções dele diretamente.
- **Signal Up (Filho avisa Pai):** O Filho **NUNCA** deve saber quem é seu pai (`get_parent()`). Ele emite um sinal ("grida" um evento), e quem estiver interessado (o pai, um Autoload, etc.) se conecta a esse sinal para reagir. O filho não se importa quem ouviu.

**Exemplo Prático: Barra de Vida (HUD)**

1. **O Player (Emissor):** Ele não sabe se existe uma UI. Ele apenas avisa que sua vida mudou.

   ```gdscript
   # player.gd
   signal health_changed(new_value, max_value)

   var hp: int = 100

   func take_damage(amount: int):
       hp -= amount
       health_changed.emit(hp, 100)
   ```

2. **A UI (Receptor):** A UI se conecta ao sinal do Player para se atualizar.

   ````gdscript # health_bar.gd
   func \_ready():
   var player = get_tree().get_first_node_in_group("Player")
   if player:
   player.health_changed.connect(update_bar)

       func update_bar(new_hp, max_hp):
           value = new_hp
           max_value = max_hp
       ```

   Este padrão de Sinais é a base para código desacoplado e fácil de manter.
   ````

---

## 5. Aplicando os Fundamentos: Gameplay Básico

Aqui estão exemplos que unem sintaxe, POO e os padrões da Godot.

### 5.1. Movimento Básico com `CharacterBody2D`

```gdscript
extends CharacterBody2D

const SPEED = 300.0
const JUMP_VELOCITY = -400.0
var gravity = ProjectSettings.get_setting("physics/2d/default_gravity")

func _physics_process(delta: float) -> void:
    if not is_on_floor():
        velocity.y += gravity * delta

    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        velocity.y = JUMP_VELOCITY

    var direction = Input.get_axis("ui_left", "ui_right")
    velocity.x = direction * SPEED if direction else move_toward(velocity.x, 0, SPEED)

    move_and_slide()
```

### 5.2. Máquina de Estados Simples com `enum` e `match`

Para dar um cérebro a um inimigo, usamos uma máquina de estados. A forma mais simples é com `enum` para os estados e `match` para a lógica.

```gdscript
extends CharacterBody2D

enum State { IDLE, CHASE, ATTACK }
var current_state: State = State.IDLE

@export var speed: float = 100.0
@export var attack_range: float = 50.0
@export var detection_range: float = 300.0
var player: Node2D

func _ready():
    # Exemplo simples de encontrar o player
    player = get_tree().get_first_node_in_group("Player")

func _physics_process(delta: float):
    # O cérebro que decide qual lógica rodar
    match current_state:
        State.IDLE:
            _process_idle()
            # Lógica de transição
            if _can_see_player():
                current_state = State.CHASE
        State.CHASE:
            _process_chase()
            # Lógica de transição
            var dist_to_player = global_position.distance_to(player.global_position)
            if dist_to_player <= attack_range:
                current_state = State.ATTACK
            elif dist_to_player > detection_range:
                current_state = State.IDLE
        State.ATTACK:
            _process_attack()
            # Lógica de transição
            if global_position.distance_to(player.global_position) > attack_range:
                current_state = State.CHASE

# Funções de cada estado
func _process_idle():
    velocity = velocity.move_toward(Vector2.ZERO, speed)
    move_and_slide()

func _process_chase():
    var direction = (player.global_position - global_position).normalized()
    velocity = direction * speed
    move_and_slide()

func _process_attack():
    velocity = Vector2.ZERO
    print("Atacando!")

# Função helper
func _can_see_player() -> bool:
    if player:
        return global_position.distance_to(player.global_position) < detection_range
    return false
```

Esta estrutura `Enum` + `Match` é o primeiro passo para criar IAs mais complexas, um tópico que aprofundaremos no documento de Máquinas de Estado.

---

## 6. Sistemas de Spawn e Fábricas

> **Objetivo:** Criar inimigos, balas e itens dinamicamente. Entender `instantiate()`, `add_child` e como evitar problemas de hierarquia.

---

### 6.1. O Básico: Instanciando Cenas

Para criar algo que não existe no editor (ex: uma bala ao apertar o gatilho), precisamos de uma referência ao seu arquivo `.tscn` (PackedScene).

```gdscript
# weapon.gd
@export var bullet_scene: PackedScene # Arraste bullet.tscn aqui

func shoot():
    if not bullet_scene: return

    # 1. Cria a instância na memória (ainda não visível)
    var bullet = bullet_scene.instantiate()

    # 2. Configura posição e rotação
    bullet.global_position = global_position
    bullet.rotation = rotation

    # 3. CRÍTICO: Onde adicionar?
    # Errado: add_child(bullet) -> A bala vai ser filha da Arma. Se a arma girar, a bala gira junto!

    # Certo: Adicionar na "Raiz do Mundo"
    get_tree().current_scene.add_child(bullet)
```

### 6.2. O Problema do `owner`

Quando você instancia algo via código, o `owner` (propriedade usada para salvar cenas) vem nulo.
Isso é normal em runtime. Só se preocupe com `owner` se estiver criando ferramentas de editor (`@tool`).

---

### 6.3. Spawners Inteligentes (Factory Pattern)

Não deixe a lógica de spawn espalhada. Crie nós dedicados: `EnemySpawner`.

#### Exemplo: Spawner com ROP

Imagine um Spawner que pode criar Goblins ou Orcs dependendo de um Resource de configuração.

```gdscript
# enemy_spawner.gd
extends Marker2D

@export var spawn_list: Array[PackedScene]
@export var spawn_interval: float = 2.0
@export var limit: int = 5

var _active_count: int = 0

func _ready():
    $Timer.wait_time = spawn_interval
    $Timer.timeout.connect(_on_timer_timeout)
    $Timer.start()

func _on_timer_timeout():
    if _active_count >= limit: return

    spawn_random()

func spawn_random():
    var scene = spawn_list.pick_random()
    var enemy = scene.instantiate()

    enemy.global_position = global_position

    # Conectar sinal de morte para liberar o slot
    enemy.tree_exited.connect(func(): _active_count -= 1)

    get_parent().add_child(enemy)
    _active_count += 1
```

---

### 6.4. Pooling (Otimização)

Instanciar (`.new()`) e Destruir (`queue_free()`) custa caro para a CPU.
Se você tem um jogo "Bullet Hell" com 1000 tiros por segundo, o jogo vai travar (Garbage Collection).

**Solução: Object Pooling**
Em vez de destruir a bala, você a esconde e desliga a física.
Quando precisar atirar de novo, você pega aquela bala escondida, move para a arma e liga de novo.

**Implementação (Simplificada):**

```gdscript
# bullet_pool.gd
var _pool: Array[Node2D] = []
var _scene: PackedScene

func get_bullet() -> Node2D:
    if _pool.is_empty():
        return _scene.instantiate() # Cria nova se faltar
    else:
        var b = _pool.pop_back()
        b.visible = true
        b.set_physics_process(true)
        return b

func return_bullet(b: Node2D):
    b.visible = false
    b.set_physics_process(false)
    _pool.append(b)
```

_Nota: Em Godot 4, a instanciação ficou muito mais rápida, então Pooling só é obrigatório em casos extremos (Mobile ou Bullet Hells massivos)._

---

### 6.5. Spawning de Itens (Loot Tables)

Ao matar um inimigo, ele deve dropar algo?
Use **Resources** para definir tabelas de loot.

```gdscript
# loot_table.gd
class_name LootTable extends Resource

@export var items: Array[LootEntry] # Array de outro Resource customizado

func roll_item() -> PackedScene:
    var roll = randf() # 0.0 a 1.0
    var current = 0.0

    for entry in items:
        current += entry.chance
        if roll <= current:
            return entry.item_scene

    return null # Nada dropou
```

O Inimigo tem um `export var loot: LootTable`. Ao morrer, chama `loot.roll_item()` e instancia o resultado.
