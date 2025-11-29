Preciso criar um novo StateEngineering.md - ele não deve ser um adr nem apenas codigos, a ideia dele é explicar por completo a essencia do plugin (sem vincular / mencionar softengine) ou kaffyn, a ideia é estruturar bem as coisas, tanto de forma teorica quanto prática. então precisamos realmente de uma abertura explicando os pontos, como que funciona essa logica de filtrar os resources e tudo mais, pra no final ter apenas os codigos, pós agora o usuario ja tem o conhecimento nescessario para usalos

# State Engineering

> Definição do domínio Machines: state machines de personagem e game machine (fluxo de jogo, transições de fase, autosave, eventos).

## Contexto

- Godot não oferece um padrão de **state machine** nem de **game flow**:
  - Cada projeto cria seu próprio sistema de estados (ou acumula `if/else` em scripts).
  - Transições de fases, menus, autosave e continue são tratados ad hoc.
- A SoftEngine precisa de um domínio dedicado a:
  - Orquestrar **estados de personagem** (movimento, combate, IA, etc.).
  - Orquestrar **estados de jogo** (menus, loading, gameplay, cutscenes, créditos).
  - Servir de ponto de integração entre Behavior, World, FX, UI e Core/SaveMachine.

## Decisão

1. **Criar o domínio Machines para estados e fluxo**

   - Machines será responsável por:
     - State machines de personagem:
       - Estados representados por `Resources` de comportamento (Behaviors do domínio Behavior).
       - Suporte a camadas/domínios paralelos (movimento, combate, IA, etc.).
       - Transições declarativas entre estados, com sinais e prioridades.
     - Game machine de alto nível:
       - Estados globais do jogo (menu principal, seleção, gameplay, pause, loading, créditos).
       - Transições entre cenas/fases, coordenando carregamento com World.
       - Pontos de autosave/continue integrados ao `SaveMachine` (Core).
     - Threads de interação:
       - Fluxos de diálogo, cutscenes, interações contextuais.
     - Sistema de eventos de jogo:
       - Definir eventos e reações que conectem Machines, Behavior, World, FX e UI.

2. **Estabelecer fronteiras claras de Machines com outros domínios**
   - Machines **não** define:
     - Como é o inventário, progressão ou stats (isso é Behavior).
     - Como o mundo é estruturado ou gerado (isso é World).
     - Como o feedback sensorial é renderizado (isso é FX).
     - Como a UI é desenhada (isso é UI).
   - Machines consome seus contratos:
     - Lê Behaviors e dados de Behavior para tomar decisões de estado.
     - Pede a World que carregue/instancie fases e spawns.
     - Emite eventos que FX e UI usam para reagir.
     - Chama `SaveMachine` para salvar/carregar em momentos definidos.

## Consequências

- **Positivas**

  - Consolida a lógica de fluxo (personagem + jogo) em um único domínio com boas práticas claras.
  - Facilita a criação de ferramentas de editor para visualizar e depurar estados e transições.
  - Reduz a duplicação de soluções de state machine entre projetos.

- **Negativas / trade-offs**
  - Jogos que adotarem Machines precisarão seguir o modelo de state machine da SoftEngine.
  - A game machine passa a ser um ponto crítico: bugs ou decisões ruins aqui impactam todo o fluxo do jogo.

## Próximos passos

- Criar ADR específico para a **game machine** (estrutura de estados globais, integração com SaveMachine e World).
- Criar ADR para o modelo de state machine de personagem (estrutura de Behaviors, transições, camadas).
- Elaborar Spec em `Specs/machines.md` detalhando APIs, tipos de Resources e contratos com Behavior, World, FX e UI.

esse era o plano antigo blz, ele ainda ta de pe mas com algumas mudanças, primeiro, esquece os nodes resources singletons e autoloads antigos, abaixo ta as definições do plano mais recente, tudo que não tiver explicitamente diferente, é pq se mantem igual ao plano aterior

Estamos fazendo um statemachine extremamente tipado e opinado / regrado, ou seja, oferecemos a solução direta para vc, não apenas uma base, não tems um StateResource generico, oq temos são Game, Move, Attack, Iterative, e varios outros estados pre definidos / montados, e seus datas / .tres irão herdar diretamente do estado que irão usar, o state engineering entende a nescessidade de também oferecer itens padrões para que ele funcione sozinho, então em adição temos Weapon e Armor, ou seja, o estado de um player pode ser composto pelo movimento, arma, armadura, ataque e interação, oq é maravilhoso, por isso temos o compose, pós precisamos aglomerar / definir estados por entidade, junto a isso eu tenho o MachineComponent, ele é que vai orquestar todos os estados, inclusive, seguindo o formato de soma de estados, além disso teremos um MachineBlueprint, tendo blueprint de player2d, inimigo2d, npc2d, e varios outros, e o mesmo para 3d, oq eles fazem? preconfiguram o MachineComponent

Pense no MachineComponent como um filtro, ou seja

States Base Atual: Motion : Idle, Jump : None, Attack : Fast, Physics : Air, Effect : None, Weapon : Katana

Final State: Katana Air Attack

pq / como chegamos nesse estado final? por filtros

Requisitos para o Estado Final: Motion : Any, Jump : Any, Attack : Any ! None (qualquer ataque que não seja none), Physics : Air, Effect : None, Weapon : Katana

ou seja, ao filtrar os estados o unico encontrado foi o Katana Air Attack, por isso ele foi escolhido

desempate: digamos que tenho outro que também passe pelos filtros

Katana Air Attack Fast - Requisitos para o Estado: Motion : Any, Jump : Any, Attack : Fast ! None (qualquer ataque que não seja none), Physics : Air, Effect : None, Weapon : Katana

ou seja, esse tb passa no filtro, mas ele tem uma correspondencia exata a mais do que o outro, que é o attack fast, nesse caso, ele é o escolhido

ou seja, o sistema vai primeiro filtrar, e depois ele vai escolher o que teve a melhor / maior correspondencia

isso sim é programação orientada a resources, pós se o jogo começar só com o katana air attack e em lançamentos futuros add o katana attack fast, eu só precisarei adicionar o resource no compose, sem editar nada do MachineComponent nem do player

para ajudar teremos no StateMachine (autolaod) todos os enuns la, isso garante / facilita a escabilidade dos filtros

```gdscript
extends Node

Enun Motion { IDLE, WALK, RUN, Dash }
Enun Jump { NONE, LOW, HIGH, FALL }
Enun Attack { NONE, FAST, NORMAL, CHANGED, SPECIAL }
Enun Physics { GROUND, AIR, WATER }
Enun Effect { NONE, FIRE, ICE, POISON, ELECTRIC }
Enun Weapon { NONE, SWORD, KATANA, GUN, BOW }
Enun Armor { NONE, IRON, STEEL, GOLD, DIAMOND }
Enum Stance { STAND, CROUCH, BLOCK, CLIMB, COVER }
Enum Status { NORMAL, STUNNED, INVULNERABLE, SUPER_ARMOR, DEAD }
Enum InputSource { PLAYER, AI, CINEMATIC, FORCE }
Enum Environment { OPEN, TIGHT_CORRIDOR, LEDGE, WATER_SURFACE }
Enum Tier { BASE, UPGRADED, MASTER, CORRUPTED }
enum Reaction {
    IGNORE,     # Não faz nada, mantém o estado atual rodando (ex: mudar cor do cabelo não para o ataque)
    CANCEL,     # Interrompe o estado imediatamente (ex: trocar de arma cancela o ataque)
    ADAPT,      # Tenta achar um estado equivalente no novo contexto (ex: AirAttack -> GroundAttack)
    FINISH      # Permite terminar a animação atual, mas impede loops ou combos
}
```

conteudo de um AttackData:

```gdscript
class_name AttackData
extends Resource

@export var name : String

@export_group("Visuals And FX")
@export var texture : Texture2D
@export var hframes : int = 10
@export var vframes : int = 1
@export var animation_res : Animation = Animation.new()
@export var loop : bool = false
@export var speed : float = 1.0
@export var sound : AudioStream

@export_group("Filters")
@export var motion : StateMachine.Motion
@export var jump : StateMachine.Jump
@export var attack : StateMachine.Attack
@export var weapon : StateMachine.Weapon
@export var physics : StateMachine.Physics
@export var effect : StateMachine.Effect

@export_group("Combat")
@export var damage : int = 10
@export var cooldown : float = 0.5
@export var area_pivot : Vector2 = Vector2(0,20)
@export var area_size : Vector2 = Vector2(20,20)

@export_group("Timing & Windows")
# Tempo mínimo para permitir cancelar este estado (ex: não pode pular no início do ataque)
@export var cancel_min_time: float = 0.2

# Permite Input Buffering? (se apertar ataque antes de acabar, já agendar o próximo)
@export var enable_buffering: bool = true
@export var buffer_window_start: float = 0.5 # A partir de 50% da animação, aceita buffer

@export_group("Costs")
enum CostType { NONE, STAMINA, MANA, HEALTH, AMMO }
@export var cost_type: CostType = CostType.NONE
@export var cost_amount: int = 0

# Regra: O que fazer se não tiver recurso?
# IGNORE_COMMAND: Não entra no estado.
# EXECUTE_WEAK: Entra numa versão "cansada" do estado (outro filtro).
enum LowResourceRule { IGNORE_COMMAND, EXECUTE_WEAK, CONSUME_HEALTH }
@export var on_insufficient_resource: LowResourceRule = LowResourceRule.IGNORE_COMMAND

@export_category("Reaction Rules")
@export_group("Context Triggers")
# O que fazer se a Física mudar (ex: cair no chão no meio do ataque)?
@export var on_physics_change: StateMachine.Reaction = StateMachine.Reaction.CANCEL

# O que fazer se a Arma mudar (ex: player trocou hotbar)?
@export var on_weapon_change: StateMachine.Reaction = StateMachine.Reaction.CANCEL

# O que fazer se o Movimento mudar (ex: começou a correr no meio do ataque)?
@export var on_motion_change: StateMachine.Reaction = StateMachine.Reaction.IGNORE

# O que fazer se tomar dano (Hit)?
@export var on_take_damage: StateMachine.Reaction = StateMachine.Reaction.CANCEL

func _newbase(new_state : int): # mais precisamente um enun, e como enun é tratado como inteiro, podemos tratar assim

  if new_state is StateMachine.Attack: #Ja sabemos que o que mudou foi o state de ataque
    return Void # não teve mudança

  if new_state is StateMachine.Weapon #Sabemos que trocou de arma
    return attack = StateMachine.Attack.("NONE") #Cancelamos o ataque, ou seja, o player so tinha mudado o state da arma, mas ao fazer isso, o ataque foi cancelado junto
```

ainda tenho que definir as funcs reais, mas a ideia é que, se eu tiver trocado de estado, primeiro eu avisarei ao estado atual o nome do novo estado, isso pq, ele pode ter alguma integração com o novo estado / uma transição

troco do estado de air para groud, mas eu estava no meio de um Katana Air Attack, oq vai acontecer? vai cancelar o ataque? vai finalizar ele no chão seguindo o mesmo movimento, ou eu irei transitar para outro ataque dando continuidade ao anterior mas agora no chão?

ou seja, eu consigo definir que o estado de "attack" deve ser mantido, e que o proximo ataque deve continuar no mid dele, ou seja, Kattana Attack, não mais kattana air attack

agora, e se eu tivesse um estado mais especifico, um de "pousar" com o ataque? daria pra ir nele

note, o estado atual não sabe nem define qual vai ser o proximo estado, ele apenas define a regra de conclusão do proprio estado, no caso um estado de ataque com espada, se ta no meio dele e troca para katana, ele vai manter o estado de ataque? não, logo se antes eu tava andando com a espada e atacando, agora que eu mudei, eu estou apenas andando com a katana, não mais atacando

porem a forma que eu defini a func foi fixa, e eu preciso de uma forma que seja exportado em variaveis (as regras tb devem ser variaveis, para que esas func interna use essas variaveis)

## Core

### 1. Autoload: `core/StateMachine.gd`

> **Dicionário de Tipos Global**

```gdscript
@tool
extends Node

# Dicionário de Tipos Global
# Adicionar um item aqui atualiza automaticamente todos os Resources do jogo.

# Constante Curinga para filtros
const ANY = -1

enum Motion { ANY = -1, IDLE, WALK, RUN, DASH }
enum Jump { ANY = -1, NONE, LOW, HIGH, FALL }
enum Attack { ANY = -1, NONE, FAST, NORMAL, CHARGED, SPECIAL }
enum Physics { ANY = -1, GROUND, AIR, WATER }
enum Effect { ANY = -1, NONE, FIRE, ICE, POISON, ELECTRIC }
enum Weapon { ANY = -1, NONE, SWORD, KATANA, GUN, BOW }
enum Armor { ANY = -1, NONE, IRON, STEEL, GOLD, DIAMOND }
enum Stance { ANY = -1, STAND, CROUCH, BLOCK, CLIMB, COVER }
enum Tier { ANY = -1, BASE, UPGRADED, MASTER, CORRUPTED }
enum GameState { ANY = -1, PLAYING, PAUSED, CUTSCENE, MENU }

# Regras de Reação
enum Reaction {
	IGNORE,     # Mantém o estado atual
	CANCEL,     # Interrompe o estado
	ADAPT,      # Tenta adaptar para o novo contexto
	FINISH      # Impede loop/combo
}

enum CostType { NONE, STAMINA, MANA, HEALTH, AMMO }
enum LowResourceRule { IGNORE_COMMAND, EXECUTE_WEAK, CONSUME_HEALTH }
```

### 2. Container Genérico: `core/StateCompose.gd`

> **Apenas uma lista de Resources. Use para criar `AttackCompose.tres`, `Moves.tres`, etc.**

```gdscript
@tool
class_name StateCompose
extends Resource

@export var states: Array[Resource]

# Cache para busca rápida por nome (útil para Cutscenes ou Debug)
var _states_map: Dictionary = {}
var _initialized: bool = false

func initialize() -> void:
	if _initialized: return
	_states_map.clear()

	for res in states:
		if res and "name" in res:
			_states_map[res.name] = res

	_initialized = true

func get_state_by_name(name: String) -> Resource:
	if not _initialized: initialize()
	return _states_map.get(name)

func get_all() -> Array[Resource]:
	return states
```

### 3. A Base (Cérebro): `core/Machine.gd`

> **Contém a lógica pesada de filtro e gerenciamento de estado.**

```gdscript
@tool
class_name Machine
extends Node

signal state_changed(new_res)
signal context_changed(category, value)

# Estado Atual
var current_state_res: Resource
var time_in_state: float = 0.0

# Contexto (Memória do Componente)
var context: Dictionary = {}

func _process(delta: float) -> void:
	if current_state_res:
		time_in_state += delta

# --- Context API ---

func set_context(category: String, value: int) -> void:
	# Otimização: Se o valor é o mesmo, ignora
	if context.get(category) == value: return

	context[category] = value
	emit_signal("context_changed", category, value)
	_on_context_updated(category, value)

# Virtual: Filhos (PlayerMachine, EnemyMachine) implementam isso
func _on_context_updated(_category: String, _value: int) -> void:
	pass

# --- State API ---

func change_state(new_res: Resource, preserve_time: bool = false) -> void:
	if current_state_res == new_res and not preserve_time: return

	current_state_res = new_res
	if not preserve_time:
		time_in_state = 0.0

	emit_signal("state_changed", new_res)

	if new_res:
		print("[%s] State: %s" % [name, new_res.get("name")])
	else:
		print("[%s] State: Stopped" % name)

# --- Algoritmo de Filtro Centralizado ---
# Roda a lógica de comparação entre o Contexto Atual e os Requisitos dos Resources
func find_best_match(candidates: Array[Resource]) -> Resource:
	var best_candidate: Resource = null
	var best_score: int = -1

	for res in candidates:
		if not res: continue

		var score = 0
		var possible = true

		# Itera sobre todas as categorias presentes no contexto atual
		for category in context.keys():
			var ctx_val = context[category]

			# Monta o nome da propriedade esperada no resource (ex: req_weapon)
			var req_prop = "req_" + category.to_lower()

			# Duck Typing: Verifica se o resource tem essa propriedade
			var req_val = res.get(req_prop)

			if req_val != null:
				# 1. Filtro Rígido (Must be EQUAL or ANY)
				if req_val != StateMachine.ANY and req_val != ctx_val:
					possible = false
					break

				# 2. Score (Match exato ganha ponto)
				if req_val == ctx_val:
					score += 1

		if not possible: continue

		# Prioridade Manual definida no Resource
		var priority = res.get("priority_override")
		if priority: score += priority

		if score > best_score:
			best_score = score
			best_candidate = res

	return best_candidate
```

## Game

### 1. Data: `data/resources/GameData.gd`

Define as regras de um estado global (ex: "Pausado", "Gameplay").

```gdscript
class_name GameData
extends Resource

@export_category("Identity")
@export var name: String
@export var priority_override: int = 0

@export_category("Requirements (Filter)")
# Define quando este estado deve ser ativado
@export var req_game_state: StateMachine.GameState = StateMachine.GameState.ANY

@export_category("Settings")
# O que acontece quando este estado entra?
@export var time_scale: float = 1.0 # 0.0 para pause
@export var mouse_mode: Input.MouseMode = Input.MOUSE_MODE_CAPTURED
@export var block_player_input: bool = false

# Opcional: Caminho para uma cena de UI (ex: PauseMenu.tscn)
@export var ui_overlay_path: String = ""
```

### 2. Machine: `machines/GameMachine.gd`

O orquestrador global. Geralmente colocado no nó raiz da cena ou num Singleton/Autoload dedicado a gerenciamento.

```gdscript
class_name GameMachine
extends Machine

@export var game_states_compose: StateCompose
@export var default_state: StateMachine.GameState = StateMachine.GameState.MENU

func _ready() -> void:
	if game_states_compose: game_states_compose.initialize()

	# Inicia o jogo no estado padrão
	set_context("GameState", default_state)

func _on_context_updated(category: String, value: int) -> void:
	if category == "GameState":
		_apply_game_state()

func _apply_game_state() -> void:
	if not game_states_compose: return

	# Usa o algoritmo da base Machine para achar o recurso (PauseData, GameplayData, etc)
	var list = game_states_compose.get_all()
	var best = find_best_match(list)

	if best and best is GameData:
		change_state(best)
		_execute_state_logic(best)

func _execute_state_logic(data: GameData) -> void:
	# Aplica configurações globais
	Engine.time_scale = data.time_scale
	Input.mouse_mode = data.mouse_mode

	# Lógica de Bloqueio de Input (pode ser lida pelo PlayerMachine depois)
	# Ex: Global.player_input_blocked = data.block_player_input

	# Exemplo de UI: Sinaliza para um UIManager o que mostrar
	# SignalBus.emit_signal("request_ui_overlay", data.ui_overlay_path)
```

## Player

### 1. Data: `data/resources/AttackData.gd`

> **O Resource que vai no arquivo .tres.**

```gdscript
@tool
class_name AttackData
extends Resource

@export_category("Identity")
@export var name: String
@export var priority_override: int = 0

@export_group("Visuals")
@export var animation_res: Animation
@export var loop: bool = false

@export_category("Filters (Requirements)")
# Nomes das vars devem bater com as chaves de Contexto (ex: "req_physics" -> Context "Physics")
@export var req_motion: StateMachine.Motion = StateMachine.Motion.ANY
@export var req_jump: StateMachine.Jump = StateMachine.Jump.ANY
@export var req_attack: StateMachine.Attack = StateMachine.Attack.ANY
@export var req_weapon: StateMachine.Weapon = StateMachine.Weapon.ANY
@export var req_physics: StateMachine.Physics = StateMachine.Physics.ANY
@export var req_stance: StateMachine.Stance = StateMachine.Stance.ANY
@export var req_tier: StateMachine.Tier = StateMachine.Tier.BASE

@export_category("Reaction Rules")
# Nomes das vars devem bater com a lógica de _get_reaction (ex: "on_physics_change")
@export var on_physics_change: StateMachine.Reaction = StateMachine.Reaction.CANCEL
@export var on_weapon_change: StateMachine.Reaction = StateMachine.Reaction.CANCEL
@export var on_motion_change: StateMachine.Reaction = StateMachine.Reaction.IGNORE
@export var on_stance_change: StateMachine.Reaction = StateMachine.Reaction.ADAPT
```

### 1. Machine: `machines/PlayerMachine.gd`

> **O script que vai no Node do Player.**

```gdscript
@tool
class_name PlayerMachine
extends Machine

@export_category("Data")
@export var attack_compose: StateCompose
# @export var move_compose: StateCompose

@export_category("Settings")
@export var default_weapon: StateMachine.Weapon = StateMachine.Weapon.KATANA

func _ready() -> void:
	if attack_compose: attack_compose.initialize()

	# Configura estado inicial
	set_context("Weapon", default_weapon)
	set_context("Physics", StateMachine.Physics.GROUND)
	set_context("Motion", StateMachine.Motion.IDLE)
	set_context("Attack", StateMachine.Attack.NONE)

func _on_context_updated(category: String, value: int) -> void:
	# 1. Input de Ataque: Procura na lista
	if category == "Attack" and value != StateMachine.Attack.NONE:
		_try_engage_attack()
		return

	# 2. Reação Passiva (ex: cair no chão, trocar arma)
	if current_state_res:
		var reaction = _get_reaction(current_state_res, category)
		_handle_reaction(reaction)

func _try_engage_attack(is_adapting: bool = false) -> void:
	if not attack_compose: return

	# Pega a lista bruta do Compose e joga pro algoritmo da Machine
	var list = attack_compose.get_all()
	var best = find_best_match(list)

	if best:
		var preserve = false
		if is_adapting and current_state_res and best.name == current_state_res.name:
			preserve = true
		change_state(best, preserve)
	elif is_adapting:
		change_state(null) # Falhou a adaptação

func _handle_reaction(reaction: int) -> void:
	match reaction:
		StateMachine.Reaction.CANCEL: change_state(null)
		StateMachine.Reaction.ADAPT: _try_engage_attack(true)
		StateMachine.Reaction.FINISH:
			if current_state_res: current_state_res.loop = false

# Helper para ler propriedades de forma segura
func _get_reaction(res: Resource, category: String) -> int:
	# Mapeia o nome da categoria para a propriedade de reação
	var prop = "on_" + category.to_lower() + "_change"
	var val = res.get(prop)
	if val != null: return val
	return StateMachine.Reaction.IGNORE
```

## Enemy

### 1. Data: `data/resources/MoveData.gd`

(Este recurso pode ser usado tanto pelo Player quanto pelo Inimigo para andar/correr).

```gdscript
class_name MoveData
extends Resource

@export_category("Identity")
@export var name: String
@export var priority_override: int = 0

@export_category("Visuals")
@export var animation_res: Animation
@export var speed: float = 100.0
@export var acceleration: float = 10.0

@export_category("Requirements (Filter)")
@export var req_motion: StateMachine.Motion = StateMachine.Motion.ANY
@export var req_physics: StateMachine.Physics = StateMachine.Physics.ANY
@export var req_stance: StateMachine.Stance = StateMachine.Stance.ANY
@export var req_tier: StateMachine.Tier = StateMachine.Tier.BASE

@export_category("Reaction Rules")
@export var on_physics_change: StateMachine.Reaction = StateMachine.Reaction.CANCEL
@export var on_damage: StateMachine.Reaction = StateMachine.Reaction.IGNORE
```

### 2. Machine: `machines/EnemyMachine.gd`

Diferente do Player (que reage a botões), o Inimigo reage a comandos da IA (Distância, Visão).

```gdscript
class_name EnemyMachine
extends Machine

@export_category("Data")
@export var move_compose: StateCompose   # Lista de MoveData (Idle, Run, Patrol)
@export var attack_compose: StateCompose # Lista de AttackData (Bite, Slash)

@export_category("Settings")
@export var default_weapon: StateMachine.Weapon = StateMachine.Weapon.NONE
@export var attack_range: float = 50.0

func _ready() -> void:
	if move_compose: move_compose.initialize()
	if attack_compose: attack_compose.initialize()

	set_context("Weapon", default_weapon)
	set_context("Physics", StateMachine.Physics.GROUND)
	set_context("Motion", StateMachine.Motion.IDLE)
	set_context("Attack", StateMachine.Attack.NONE)

# --- API para o Controlador de IA ---
# O script do CharacterBody (EnemyController) chama isso a cada frame ou via Timer
func update_ai_decision(target_distance: float, has_line_of_sight: bool, health_low: bool) -> void:

	# 1. Decisão de Movimento
	if target_distance <= attack_range and has_line_of_sight:
		# Perto o suficiente -> Parar e tentar atacar
		set_context("Motion", StateMachine.Motion.IDLE)
		set_context("Attack", StateMachine.Attack.NORMAL)
	elif has_line_of_sight:
		# Vendo mas longe -> Perseguir
		set_context("Motion", StateMachine.Motion.RUN)
		set_context("Attack", StateMachine.Attack.NONE)
	else:
		# Perdido -> Idle ou Patrulha
		set_context("Motion", StateMachine.Motion.IDLE) # Ou WALK para patrulha
		set_context("Attack", StateMachine.Attack.NONE)

# --- Reatividade ---

func _on_context_updated(category: String, value: int) -> void:
	# Prioridade 1: Ataque
	if category == "Attack" and value != StateMachine.Attack.NONE:
		_try_engage_attack()
		return

	# Prioridade 2: Reação de Dano/Física no estado atual
	if current_state_res:
		var reaction = _check_reaction(current_state_res, category)
		if reaction == StateMachine.Reaction.CANCEL:
			change_state(null)
			return

	# Prioridade 3: Movimento (Se não estiver atacando)
	if category == "Motion" and (current_state_res == null or current_state_res is MoveData):
		_try_engage_movement()

func _try_engage_attack() -> void:
	if not attack_compose: return
	var best = find_best_match(attack_compose.get_all())
	if best: change_state(best)

func _try_engage_movement() -> void:
	if not move_compose: return
	var best = find_best_match(move_compose.get_all())
	# Só troca se for diferente, para não resetar animação de Run toda hora
	if best: change_state(best, true)

# Helper genérico para ler reações de múltiplos tipos de dados
func _check_reaction(res: Resource, category: String) -> int:
	# Verifica se existe a propriedade 'on_X_change'
	var prop = "on_" + category.to_lower() + "_change"
	var val = res.get(prop)
	if val != null: return val
	return StateMachine.Reaction.IGNORE
```
