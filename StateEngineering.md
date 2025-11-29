# State Engineering

> Arquitetura de Máquina de Estados Orientada a Resources e Contexto.

## 1. Conceito e Filosofia

O State Engineering propõe uma mudança de paradigma em relação às State Machines tradicionais (FSM). Em vez de definir transições rígidas (`Idle -> Walk -> Run`), definimos **Estados** como recursos independentes e usamos um **Contexto** para filtrar qual recurso deve estar ativo a cada momento.

Este sistema é **fortemente tipado e opinado**. Não oferecemos apenas uma base vazia; oferecemos uma estrutura padronizada para lidar com Movimento, Combate, Fluxo de Jogo e IA, garantindo escalabilidade imediata.

### O Problema

Em engines como Godot, a falta de um padrão nativo leva a:

- Scripts monolíticos cheios de `if/else`.
- Dificuldade em reutilizar lógica (ex: o ataque de um inimigo não serve para o player).
- Acoplamento rígido (o estado de "Pulo" precisa saber que existe um estado de "Ataque Aéreo" para transitar para ele).

### A Solução: Filtragem Contextual

Em vez de o código ditar a transição, o **Ambiente (Contexto)** dita a possibilidade.

Pense no componente `Machine` não como um gerenciador de estados simples, mas como um **Motor de Busca e Filtragem**.

#### O Algoritmo de "Score System"

Imagine o seguinte cenário:

- **Contexto Atual:** O Player está segurando uma **Katana** e está no **Ar** (Caindo).
- **Input:** O jogador aperta o botão de **Ataque**.

O sistema não pergunta "qual é o próximo estado?". Ele pergunta: **"Dentre todos os ataques disponíveis, qual se encaixa melhor nessa situação?"**

1. **Fase 1: Requisitos (Filter)**

   - **Ataque Genérico de Espada?** -> Rejeitado (Requer Arma: Sword).
   - **Ataque de Chão da Katana?** -> Rejeitado (Requer Física: Ground).
   - **Ataque Aéreo da Katana?** -> **Aceito** (Requer Arma: Katana, Física: Air).

2. **Fase 2: Pontuação (Score)**
   Se houver múltiplos candidatos aceitos, quem ganha?

   - **Candidato A:** Requer `Physics: Any`. (Pontos: 0)
   - **Candidato B:** Requer `Physics: Air`. (Pontos: +1 por especificidade).
   - **Vencedor:** Candidato B.

Isso permite que você adicione um novo ataque (ex: "Katana Fast Air Attack") apenas criando um arquivo `.tres`, sem tocar em uma linha de código. O sistema automaticamente o escolherá se o contexto for `Attack: Fast`.

---

## 2. Arquitetura de Dados

O sistema é composto por três pilares fundamentais:

### A. Machine (O Cérebro)

É um `Node` que vive na cena (Player, Inimigo, Game). Ele mantém a "memória" do que está acontecendo agora (Contexto) e executa o algoritmo de busca.

- **Responsabilidade:** Gerenciar inputs, atualizar contexto e trocar o recurso atual.

### B. Data (A Regra)

São `Resources` (`.tres`). Eles não contêm lógica de execução, apenas dados e regras de reação.

- **Exemplo:** `KatanaSlash.tres`. Define dano, animação, cooldown e **Requisitos** (ex: `req_weapon: KATANA`).
- **Reatividade:** Define o que fazer se o contexto mudar (ex: "Se eu tocar no chão, Cancele este ataque").

### C. Compose (A Coleção)

É um contêiner (`Resource`) que agrupa múltiplos `Data`.

- **Função:** Serve como o "Banco de Dados" local da entidade. O Player tem um `AttackCompose` contendo todos os seus ataques possíveis.

---

## 3. Implementação Prática

Abaixo estão os scripts essenciais para o funcionamento do ecossistema.

### 3.1. Core

**A base do sistema, agnóstica a tipos de entidades.**

#### `core/StateMachine.gd` (Autoload)

Define o vocabulário do jogo.

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

#### `core/StateCompose.gd`

Container genérico para listas de estados.

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

#### `core/Machine.gd`

A lógica central de processamento e filtragem.

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

---

### 3.2. Game Flow

**Gerenciamento de estados globais (Pause, Menu, Play).**

#### `data/resources/GameData.gd`

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

#### `machines/GameMachine.gd`

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

  # Lógica de Bloqueio de Input e UI seriam implementadas aqui
```

---

### 3.3. Player System

**Sistema reativo baseado em Inputs e Combate.**

#### `data/resources/AttackData.gd`

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

#### `machines/PlayerMachine.gd`

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

---

### 3.4. Enemy System

**Sistema reativo baseado em Decisões de IA.**

#### `data/resources/MoveData.gd`

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

#### `machines/EnemyMachine.gd`

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
