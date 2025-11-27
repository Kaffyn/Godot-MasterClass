# Curso Kaffyn: GDScript Essentials (Zero to Hero)

> **Instrutor:** Machi
> **Objetivo:** Ensinar a linguagem GDScript pura, lógica de programação e implementação de gameplay básica sem depender de arquiteturas complexas (ROP). Foco em "fazer funcionar bem".

---

## Módulo 1: A Sintaxe da Linguagem

GDScript é similar a Python. Indentação (TAB) define blocos de código.

### Variáveis e Tipagem
Na Kaffyn, usamos **Tipagem Estática**. Isso evita bugs e melhora o autocomplete.

```gdscript
# Ruim (Dinâmico - pode ser qualquer coisa)
var vida = 100

# Bom (Estático - garantido ser inteiro)
var vida: int = 100
var velocidade: float = 250.5
var nome: String = "Machi"
var is_vivo: bool = true
```

### Coleções (Arrays e Dictionaries)

**Arrays (Listas Ordenadas):**
```gdscript
var inventario: Array[String] = ["Espada", "Poção", "Mapa"]

func exemplo_array():
    inventario.append("Chave")
    print(inventario[0]) # Imprime "Espada"
    inventario.erase("Poção")
    
    # Loop (Iteração)
    for item in inventario:
        print("Tenho: " + item)
```

**Dictionaries (Chave-Valor):**
Ótimo para dados estruturados simples.
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

### Funções (Métodos)

```gdscript
# void: Não retorna nada
func pular() -> void:
    velocity.y = -400

# Retorna int
func somar(a: int, b: int) -> int:
    return a + b
```

**Static Funcs (Utilitários):**
Funções que não precisam de uma instância do objeto para rodar. Útil para matemática ou ajudantes.
```gdscript
static func calcular_dano(base: int, forca: int) -> int:
    return base + (forca * 2)
```

---

## Módulo 2: Movimento do Player (CharacterBody2D)

O "Hello World" dos jogos.

```gdscript
extends CharacterBody2D

const SPEED = 300.0
const JUMP_VELOCITY = -400.0

# Get the gravity from the project settings to be synced with RigidBody nodes.
var gravity = ProjectSettings.get_setting("physics/2d/default_gravity")

func _physics_process(delta: float) -> void:
    # 1. Aplicar Gravidade
    if not is_on_floor():
        velocity.y += gravity * delta

    # 2. Pulo
    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        velocity.y = JUMP_VELOCITY

    # 3. Movimento Horizontal
    # Retorna -1 (esquerda), 1 (direita) ou 0 (parado)
    var direction = Input.get_axis("ui_left", "ui_right")
    
    if direction:
        velocity.x = direction * SPEED
    else:
        # Desaceleração suave
        velocity.x = move_toward(velocity.x, 0, SPEED)

    # 4. A Mágica da Godot (Move baseado na velocity e trata colisões)
    move_and_slide()
```

---

## Módulo 3: Inimigo Simples (Follow & Attack)

Como fazer um inimigo perseguir o player?

```gdscript
extends CharacterBody2D

var speed: float = 150.0
var player_ref: CharacterBody2D = null

func _ready():
    # Jeito "sujo" mas funcional para protótipos: buscar o player no grupo
    var players = get_tree().get_nodes_in_group("Player")
    if players.size() > 0:
        player_ref = players[0]

func _physics_process(delta):
    if player_ref:
        # Direção = Destino - Origem (Normalizado para ter tamanho 1)
        var direction = (player_ref.global_position - global_position).normalized()
        
        velocity = direction * speed
        move_and_slide()
        
        # Olhar para o player
        if direction.x > 0:
            $Sprite2D.flip_h = false
        else:
            $Sprite2D.flip_h = true
```

---

## Módulo 4: Sistema de Dano (Causar e Receber)

A comunicação básica entre objetos.

**No Inimigo (Receber Dano):**
```gdscript
var hp: int = 3

func take_damage(amount: int) -> void:
    hp -= amount
    $AnimationPlayer.play("hurt")
    
    if hp <= 0:
        die()

func die():
    queue_free() # Remove o nó da cena
```

**No Player (Causar Dano - Ataque Corpo a Corpo):**
Geralmente usamos uma `Area2D` chamada "Hitbox" que ativamos durante a animação de ataque.

```gdscript
# player.gd

func attack():
    $AnimationPlayer.play("attack")
    # A animação deve ligar/desligar o monitoring da Area2D

# Sinal da Area2D (Hitbox)
func _on_hitbox_body_entered(body):
    if body.has_method("take_damage"):
        body.take_damage(1)
```

---

## Módulo 5: Inteligência Artificial Básica (Enum State Machine)

Como fazer o inimigo não ser um zumbi que só anda em linha reta? Ele precisa de **Estados**.
Sem usar ROP ou classes complexas, usamos `enum` e `match`.

```gdscript
extends CharacterBody2D

# 1. Definir os estados possíveis
enum State { IDLE, CHASE, ATTACK }

# 2. Variável para guardar o estado atual
var current_state: State = State.IDLE

# Configurações
var speed = 100.0
var attack_range = 50.0
var detect_range = 300.0
var player: Node2D = null

func _physics_process(delta):
    # 3. A Máquina de Estados (Cérebro)
    match current_state:
        State.IDLE:
            _process_idle(delta)
        State.CHASE:
            _process_chase(delta)
        State.ATTACK:
            _process_attack(delta)

# --- Lógica de cada Estado ---

func _process_idle(delta):
    velocity = Vector2.ZERO
    $AnimationPlayer.play("idle")
    
    # Transição: Se ver o player, persiga
    if _can_see_player():
        current_state = State.CHASE

func _process_chase(delta):
    if not player: return
    
    var direction = (player.global_position - global_position).normalized()
    velocity = direction * speed
    move_and_slide()
    $AnimationPlayer.play("run")
    
    var dist = global_position.distance_to(player.global_position)
    
    # Transições
    if dist <= attack_range:
        current_state = State.ATTACK
    elif dist > detect_range:
        current_state = State.IDLE

func _process_attack(delta):
    velocity = Vector2.ZERO
    # Toca animação de ataque e espera ela acabar
    # (Nota: Em código real, usariamos sinais de animação, aqui é simplificado)
    $AnimationPlayer.play("attack") 
    
    # Transição de volta (simulada)
    # Num jogo real, aguardariamos 'animation_finished'
    if not $AnimationPlayer.is_playing():
        current_state = State.CHASE

# --- Helpers ---
func _can_see_player() -> bool:
    # Lógica simples de distância
    if not player:
        var p = get_tree().get_nodes_in_group("Player")
        if p: player = p[0]
    
    if player:
        return global_position.distance_to(player.global_position) < detect_range
    return false
```

Esta estrutura `Enum` + `Match` é a base de qualquer IA de jogo. Simples, legível e funcional para comportamentos básicos.
