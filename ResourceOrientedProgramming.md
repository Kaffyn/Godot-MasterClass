# Godot MBA: Programação Orientada a Resources (ROP)

Bem-vindo ao guia definitivo sobre **Resource-Oriented Programming (ROP)** na Kaffyn. Este documento foi estruturado como um mini-curso para transformar sua mentalidade de "Programador de Scripts" para "Arquiteto de Sistemas Godot".

---

## Módulo 1: A Realidade Oculta

### Você já usa Resources (mesmo sem saber)

Muitos iniciantes acham que Resources são algo avançado ou exótico. A verdade é que **90% do desenvolvimento na Godot é manipulação de Resources**.

Quando você arrasta uma imagem para um `Sprite2D`, você está preenchendo uma propriedade com um `Texture2D`. Quando coloca um som num `AudioStreamPlayer`, está usando um `AudioStream`.

- **`Texture2D` (.png, .jpg):** Dados de pixel.
- **`AudioStream` (.wav, .ogg):** Dados de onda sonora.
- **`Animation`:** Dados de keyframes.
- **`TileSet`:** Definições de tiles e colisões.

**A Grande Sacada:**
Se a engine usa Resources para gerenciar dados complexos (como texturas e áudios) de forma eficiente e compartilhada, por que você está usando `Dictionaries` ou variáveis soltas (`var damage = 10`) para gerenciar os dados do seu jogo?

**O Poder da Referência:**
Se 500 inimigos usam a mesma textura `goblin.png`, a Godot carrega essa imagem na memória **apenas uma vez**. Se você criar seus próprios Resources, ganhará essa mesma otimização de memória de graça.

---

## Módulo 2: Definindo seus Próprios Resources

### A Sintaxe Básica

Criar um Resource customizado é criar uma "Ficha de Dados" ou um "Blueprint".

Para começar, crie um script `.gd` que herde de `Resource`.

```gdscript
# character_stats.gd
extends Resource
class_name CharacterStats

# O 'class_name' é CRUCIAL.
# Ele registra esse script no editor da Godot, permitindo que você
# clique com botão direito -> Create New -> CharacterStats.

@export_group("Atributos Base")
@export var max_hp: int = 100
@export var speed: float = 250.0
@export var display_name: String = "Hero"

@export_group("Visual")
@export var portrait: Texture2D
```

### A Prática: Criando Arquivos `.tres`

1. Salve o script acima.
2. No FileSystem, clique com botão direito -> **Create New...**
3. Busque por **CharacterStats**.
4. Crie arquivos como `goblin_stats.tres`, `hero_stats.tres`, `boss_stats.tres`.

Agora você tem arquivos físicos no seu projeto que representam o balanceamento do jogo. **Nenhum código de lógica foi escrito ainda, apenas definições de dados.**

---

## Módulo 3: A Magia dos Dados com Comportamento

### Resources com Funções (Helper Functions)

Resources não são apenas "structs" burras de dados. Eles podem ter métodos! O segredo é que esses métodos devem operar **apenas sobre os dados do próprio Resource** (funções puras ou autocontidas).

Isso permite encapsular regras de negócio dentro do dado.

**Exemplo: Um Item de Cura**

```gdscript
# healing_item.gd
extends Resource
class_name HealingItem

@export var base_heal: int = 10
@export var multiplier: float = 1.5
@export var sound_on_use: AudioStream

# O Resource sabe calcular quanto ele cura baseado em um parâmetro externo
func calculate_healing(user_intelligence: int) -> int:
	return base_heal + (user_intelligence * multiplier)

# O Resource pode retornar dados formatados para UI
func get_tooltip_text() -> String:
	return "Cura base de %d + (INT * %.1f)" % [base_heal, multiplier]
```

Note que o Resource **não cura o player**. Ele não acessa `Player.hp`. Ele apenas fornece a informação e os cálculos necessários. Quem aplica a cura é o Node.

---

## Módulo 4: Integração com Nodes (O Padrão)

### Injeção de Dependência via Inspector

O padrão Machi dita que **Nodes são Agentes** e **Resources são Configurações**. O Node sabe "como atirar", o Resource define "o que está sendo atirado".

**Exemplo no Player:**

```gdscript
# player.gd
extends CharacterBody2D

# Aqui está a mágica. Não hardcode valores. Exponha o "contrato".
@export var stats: CharacterStats

@onready var sprite = $Sprite2D

func _ready():
	# Validação de segurança
	if not stats:
		push_error("Player sem Stats atribuído!")
		return

	# Configuração inicial usando os dados do Resource
	sprite.texture = stats.portrait
	print("Player %s inicializado com %d HP" % [stats.display_name, stats.max_hp])

func take_damage(amount: int):
	# Em um cenário real, usariamos uma variavel current_hp local,
	# mas baseada no max_hp do resource.
	pass
```

Agora, para mudar o Player de "Guerreiro" para "Mago", você não muda o script. Você apenas arrasta um novo `.tres` para o slot `Stats` no Inspector.

---

## Módulo 5: Aplicações Avançadas (Estudos de Caso)

Agora que dominamos a teoria, vamos implementar sistemas complexos e reais usados em produção na Kaffyn.

### Caso A: Máquina de Estados (Resource-Based FSM)

Uma evolução natural da ROP é substituir `enums` (`State.IDLE`, `State.RUN`) por `Resources`. Isso permite que cada estado carregue seus próprios dados (animação, colisão, sons) e que a lógica do controlador apenas "aplique" o Resource atual, sem switches gigantes.

#### 1. Definindo os Dados (Os Resources)

Primeiro, criamos a definição ("blueprint") do que compõe um movimento e um ataque.

```gdscript
# move_resource.gd
class_name MoveResource extends Resource

@export var id: String = "idle" # Identificador único para lógica (ex: "run", "jump")
@export var animation: String = "idle_anim" # Nome da animação no AnimationPlayer
@export var collision_shape: Vector2 = Vector2(20, 40) # Tamanho da hitbox
@export var sound_fx: String = "" # Nome do som no SoundManager
```

```gdscript
# attack_resource.gd
class_name AttackResource extends Resource

@export var id: String = "slash_1"
@export var damage: int = 10
@export var animation: String = "slash_1_anim"
@export var cooldown: float = 0.5
@export var collision_config: Vector2 = Vector2(40, 40) # Tamanho da hitbox do ataque
```

#### 2. O Controlador (O Cérebro)

O script do personagem (`CharacterBody2D`) gerencia os Resources. Note como usamos Dicionários (`_moves_map`) para evitar loops (`for`) a cada frame, garantindo performance O(1).

```gdscript
extends CharacterBody2D

# --- Componentes ---
@onready var anim_player: AnimationPlayer = $AnimationPlayer
@onready var collision_shape: CollisionShape2D = $CollisionShape2D
@onready var audio_player: AudioStreamPlayer2D = $AudioStreamPlayer2D

# --- Configuração de Resources (Inspector) ---
@export_group("State Configuration")
@export var move_library: Array[MoveResource] # Arraste seus .tres aqui
@export var attack_library: Array[AttackResource] # Arraste seus .tres aqui

# --- Estado Atual ---
var current_move_res: MoveResource
var current_attack_res: AttackResource

# --- Cache para Performance (O(1) Lookup) ---
var _moves_map: Dictionary = {}
var _attacks_map: Dictionary = {}

# --- Variaveis de Gameplay ---
var SPEED = 300.0
const JUMP_VELOCITY = -400.0
var _is_attacking: bool = false

func _ready() -> void:
	# 1. Indexar Resources para acesso rápido
	for move in move_library:
		_moves_map[move.id] = move
	for attack in attack_library:
		_attacks_map[attack.id] = attack

	# 2. Estado Inicial
	if move_library.size() > 0:
		_apply_move_state(move_library[0].id)

func _physics_process(delta: float) -> void:
	# Se estiver atacando e o ataque travar movimento, retorna (opcional)
	# if _is_attacking: return

	# --- Lógica de Física ---
	if not is_on_floor():
		velocity += get_gravity() * delta

	if Input.is_action_just_pressed("ui_accept") and is_on_floor():
		velocity.y = JUMP_VELOCITY

	var direction := Input.get_axis("ui_left", "ui_right")
	if direction:
		velocity.x = direction * SPEED
	else:
		velocity.x = move_toward(velocity.x, 0, SPEED)

	move_and_slide()

	# --- Resolver Estados ---
	_resolve_move_state()
	_resolve_attack_state()

# Decide qual estado de movimento DEVERÍAMOS estar
func _resolve_move_state() -> void:
	var desired_id = "idle"

	if not is_on_floor():
		desired_id = "jump"
	elif velocity.x != 0:
		desired_id = "run"

	# Só aplica se mudou (evita reiniciar animações)
	if current_move_res == null or current_move_res.id != desired_id:
		_apply_move_state(desired_id)

# Aplica os dados do Resource de Movimento ao personagem
func _apply_move_state(id: String) -> void:
	if not _moves_map.has(id):
		push_warning("MoveState não encontrado: " + id)
		return

	current_move_res = _moves_map[id]

	# Aplica Animação
	if anim_player:
		anim_player.play(current_move_res.animation)

	# Aplica Colisão (Exemplo com RectangleShape2D)
	if collision_shape and collision_shape.shape is RectangleShape2D:
		collision_shape.shape.size = current_move_res.collision_shape

	# Toca som se houver (Lógica simplificada)
	if current_move_res.sound_fx != "":
		print("Playing sound: ", current_move_res.sound_fx)

# Decide se um ataque foi inputado
func _resolve_attack_state() -> void:
	if _is_attacking: return # Já estamos atacando

	if Input.is_action_just_pressed("ui_attack"):
		_perform_attack("slash_1") # Poderia ser lógica de combo aqui

# Executa a lógica de Ataque (Transiente)
func _perform_attack(id: String) -> void:
	if not _attacks_map.has(id): return

	_is_attacking = true
	current_attack_res = _attacks_map[id]

	# Aplica dados do Ataque
	if anim_player:
		anim_player.play(current_attack_res.animation)
		# Aguarda fim da animação para liberar estado
		await anim_player.animation_finished

	_is_attacking = false
	current_attack_res = null

	# Retorna ao estado de movimento correto
	if anim_player and current_move_res:
		anim_player.play(current_move_res.animation)
```

**Vantagens:**

1. **Extensibilidade:** Para adicionar um novo ataque "Heavy Slash", você cria um arquivo `.tres`, configura o dano/animação e arrasta para o array `attack_library`. Zero código alterado.
2. **Visualização:** O Inspector mostra exatamente quais estados o personagem tem.
3. **Separação:** O programador cuida do `_physics_process`, o Game Designer cuida de ajustar hitboxes e tempos de animação nos arquivos `.tres`.

> **Nota do Arquiteto:** Esta abordagem (Resource-Based FSM) é considerada o **Nível 3** de maturidade em Godot. Se você quer ver o **Nível 4**, onde removemos até mesmo os `if/else` de transição em favor de um sistema de "Score e Filtragem Contextual", consulte o documento de **[State Engineering](StateEngineering.md)**.

---

### Caso B: Sistema de Inventário

O Inventário é o exemplo clássico de ROP. Aqui, a separação entre **Definição** (O que é o item?) e **Estado** (Quantos eu tenho?) é crítica.

#### 1. A Definição (`ItemData`)

Este Resource representa o item de forma estática. Ele não sabe "quem" o possui ou "quantos" existem.

```gdscript
# item_data.gd
class_name ItemData extends Resource

@export var id: String = "item_unique_id"
@export var display_name: String = "New Item"
@export_multiline var description: String = ""
@export var icon: Texture2D
@export var max_stack: int = 99
@export var weight: float = 1.0
```

#### 2. O Estado (`SlotData`)

Para ter um inventário, precisamos de uma forma de contar os itens. Usamos um **Resource container** chamado `SlotData`. Ele serve como a "célula" do inventário.

```gdscript
# slot_data.gd
class_name SlotData extends Resource

@export var item_data: ItemData
@export_range(0, 99) var quantity: int = 0:
	set(value):
		quantity = value
		# Se o slot estiver vazio, limpa a referência do item
		if quantity <= 0:
			item_data = null
			quantity = 0

func can_add_item(new_item: ItemData) -> bool:
	return item_data == new_item or item_data == null

func add_item(new_item: ItemData, amount: int = 1) -> void:
	if item_data == null:
		item_data = new_item

	quantity += amount
```

#### 3. O Componente (`InventoryComponent`)

Finalmente, a lógica de gerenciar os slots fica em um `Node` (Componente), que pode ser anexado ao Player, a um Baú ou a um NPC.

```gdscript
# inventory_component.gd
class_name InventoryComponent extends Node

# O inventário é uma coleção de Slots.
# Exportamos Array[SlotData] para poder configurar slots iniciais no editor.
@export var slots: Array[SlotData] = []

signal inventory_updated

func add_item(item: ItemData, amount: int = 1) -> bool:
	# 1. Tentar empilhar em slots existentes
	for slot in slots:
		if slot.item_data == item and slot.quantity < item.max_stack:
			var space_left = item.max_stack - slot.quantity
			var add_amount = min(amount, space_left)

			slot.quantity += add_amount
			amount -= add_amount

			if amount == 0:
				inventory_updated.emit()
				return true

	# 2. Se sobrar, buscar slot vazio
	for slot in slots:
		if slot.item_data == null:
			slot.add_item(item, amount)
			inventory_updated.emit()
			return true

	# Falha: inventário cheio
	return false
```

**Vantagem:** O mesmo script `InventoryComponent` serve para o inventário do Jogador, para o Baú da dungeon e para a loja do NPC. Tudo configurável via Inspector.

---

## Módulo 6: Benefícios e Conclusão

Por que este MBA exige ROP?

1. **Separação de Responsabilidades:** Programadores codam a lógica (`.gd`), Game Designers ajustam o balanceamento (`.tres`).
2. **Versionamento (Git):** Resources são arquivos de texto. Se um designer muda o HP do Boss, o Git mostra exatamente essa linha alterada, sem conflito com a lógica do script.
3. **Reusabilidade:** Um `EffectResource` de "Explosão de Fogo" pode ser usado por uma Trap, por uma Magia do Player e por um Inimigo, sem duplicação de código.
4. **Testabilidade:** É fácil escrever testes unitários para Resources, pois eles não dependem da SceneTree, gráficos ou input.

**Próximos Passos:**

1. Pare de usar `export var speed = 100`. Crie um `MoveStats`.
2. Pare de usar Strings para identificar coisas. Use Resources como chaves de dicionário.
3. Explore o uso de `@export_resource` e arrays tipados.

Bem-vindo à arquitetura profissional em Godot.
