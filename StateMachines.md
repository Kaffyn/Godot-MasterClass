# Godot MBA: Máquinas de Estado - A Arte de Orquestrar o Comportamento

> **De:** Machi
> **Para:** Você, que busca controle e clareza no comportamento do seu jogo.
>
> Comportamentos complexos em jogos rapidamente se transformam em um pesadelo de `if/else` aninhados. Seu inimigo está `parado`, `andando`, `atacando`... E de repente, ele `esconde`, `toma dano` e `morre`. Cada nova condição gera mais uma ramificação, mais um bug. Máquinas de Estado Finitas (FSMs) são a sua salvação. Elas fornecem a estrutura e a disciplina para orquestrar o comportamento de qualquer entidade no seu jogo, do menor inimigo ao jogo inteiro.

---

## 1. O Problema do `if/else` Infinito (Spaghetti Code do Comportamento)

Quando o comportamento de uma entidade é governado por uma longa cadeia de `if/else`, a complexidade escala exponencialmente.

```gdscript
# inimigo_spaghetti.gd
func _process(delta):
    if is_alive:
        if is_idle:
            # Lógica de Idle
            if player_in_range:
                is_idle = false
                is_chasing = true
        elif is_chasing:
            # Lógica de Perseguição
            if player_too_close:
                is_chasing = false
                is_attacking = true
            elif player_escaped:
                is_chasing = false
                is_idle = true
        elif is_attacking:
            # Lógica de Ataque
            if attack_finished:
                is_attacking = false
                is_chasing = true
    else:
        # Lógica de Morte
```

Este código é difícil de ler, de depurar e de estender. Adicionar um novo comportamento (ex: "correr para cobertura") é um convite para novos bugs. A solução é estruturar esses comportamentos em **Estados**.

## 2. As Máquinas de Estado Finitas (FSMs): O Básico

Uma Máquina de Estado Finita (FSM) é um modelo matemático de computação. Ela pode estar em apenas um **estado** por vez. Pode mudar de um estado para outro (uma **transição**) em resposta a certas entradas ou condições.

- **Estados**: `IDLE`, `CHASE`, `ATTACK`, `DEAD`.
- **Transições**: `IDLE -> CHASE` (se o player for detectado), `CHASE -> ATTACK` (se o player estiver perto).

## 3. Os 3 Níveis de Máquinas de Estado em Godot

A Godot oferece flexibilidade para implementar FSMs, desde a abordagem mais simples até a mais robusta e desacoplada.

### 3.1. Nível Básico: `enum` e `match` (O Ponto de Partida)

Esta é a forma mais direta de implementar uma FSM e é um excelente ponto de partida para comportamentos simples.

- **Como Funciona**: Você define os estados usando um `enum`. No `_process` (ou `_physics_process`), você usa uma estrutura `match` para executar a lógica correspondente ao `current_state`.
- **Vantagens**: Simples de entender e implementar, rápido para protótipos e pequenos comportamentos.
- **Desvantagens**: A lógica de todos os estados vive no mesmo script, tornando-o grande e difícil de manter. As transições são implícitas nas condições `if` dentro de cada `case`.

_Exemplo (revisado de Fundamentos_Godot.md):_

```gdscript
extends CharacterBody2D

enum State { IDLE, CHASE, ATTACK }
var current_state: State = State.IDLE

func _physics_process(delta: float):
    match current_state:
        State.IDLE:
            _process_idle_state(delta)
        State.CHASE:
            _process_chase_state(delta)
        State.ATTACK:
            _process_attack_state(delta)

func _process_idle_state(delta):
    # Lógica de Idle
    if _player_detected():
        current_state = State.CHASE

func _process_chase_state(delta):
    # Lógica de Perseguição
    if _player_is_close_enough():
        current_state = State.ATTACK
    elif _player_escaped():
        current_state = State.IDLE

func _process_attack_state(delta):
    # Lógica de Ataque
    if _attack_finished():
        current_state = State.CHASE
```

### 3.2. Nível Intermediário: "State Pattern" (Classes ou Nós)

Esta abordagem implementa o padrão de design "State". Cada estado é uma **classe (ou nó) separada**, encapsulando sua própria lógica de `_enter`, `_process` e `_exit`.

- **Como Funciona**: Você tem um nó `StateMachine` central que gerencia uma lista de nós-filhos, onde cada filho é um estado. O `StateMachine` apenas delega o processamento ao estado atual.
- **Vantagens**: Clareza na separação de responsabilidades. Cada script de estado é pequeno e focado. Fácil de adicionar novos estados.
- **Desvantagens**: Pode gerar muito boilerplate (código repetitivo) para a criação de cada arquivo de estado. A lógica ainda é totalmente baseada em código, dificultando o ajuste por game designers.

_Exemplo Conceitual:_

```gdscript
# state_machine.gd (No Player ou Inimigo)
extends Node

@export var initial_state: Node # Drag-and-drop o nó do estado inicial

var current_state: Node

func _ready():
    current_state = initial_state
    current_state.enter()

func change_state(new_state_node: Node):
    current_state.exit()
    current_state = new_state_node
    current_state.enter()

func _process(delta):
    current_state.process(delta) # Delega a lógica
```

```gdscript
# idle_state.gd (Nó-filho de StateMachine)
extends Node
class_name IdleState

func enter():
    # Inicia animação de Idle
    get_parent().get_parent().anim_player.play("idle")

func exit():
    # Lógica ao sair de Idle
    pass

func process(delta):
    # Condições para transição
    if get_parent().get_parent()._player_detected():
        get_parent().change_state(get_parent().get_node("ChaseState"))
```

### 3.3. Nível Avançado (Machi Way): FSM Baseada em Resources

Esta é a abordagem mais poderosa e alinhada com a ROP (Programação Orientada a Resources) e a filosofia da SoftEngine. **Cada estado é um `Resource`**, permitindo que game designers ajustem o comportamento via Inspector, sem tocar em código.

- **Como Funciona**: Um `StateComponent` central gerencia uma biblioteca de `StateResource`s. Cada `StateResource` define dados (animação, velocidade, duração) e pode até conter pequenas funções para lógicas específicas desse estado. As transições são definidas como dados no `StateResource`.
- **Vantagens**:
  - **Data-Driven**: Game designers podem criar e ajustar estados diretamente no editor.
  - **Reusabilidade Extrema**: Um `IdleStateResource` pode ser compartilhado por dezenas de inimigos.
  - **Desacoplamento**: A lógica do estado é separada do controlador.
  - **Integração com Save**: Fácil de serializar e carregar estados de jogo.
- **Desvantagens**: Mais complexo para configurar inicialmente, exige um entendimento profundo de ROP.

_Exemplo Conceitual (similar ao `ROP.md` e `softengine_machines`):_

```gdscript
# idle_state_resource.gd
extends Resource
class_name IdleStateResource

@export var animation_name: String = "idle_anim"
@export var movement_speed_multiplier: float = 0.0 # Parado
@export var detection_range: float = 100.0

func get_transition_condition(pawn_data: Dictionary) -> String:
    if pawn_data["player_distance"] < detection_range:
        return "chase_state_id" # Retorna o ID do próximo estado
    return "" # Sem transição
```

```gdscript
# character_fsm_controller.gd (Anexado ao Player/Inimigo)
extends Node
class_name CharacterFSMController

@export var states_library: Array[StateResource] # Biblioteca de estados .tres
@export var initial_state_id: String = "idle_state_id"

var current_state_resource: StateResource
var _states_map: Dictionary = {} # Para lookup rápido

func _ready():
    for state in states_library:
        _states_map[state.id] = state
    change_state(initial_state_id)

func change_state(state_id: String):
    var new_state = _states_map.get(state_id)
    if new_state:
        current_state_resource = new_state
        # Aplica os dados do Resource (anim, speed, etc.)
        get_parent().apply_state_data(current_state_resource)
    else:
        push_error("Estado não encontrado: " + state_id)

func _process(delta):
    if current_state_resource:
        var next_state_id = current_state_resource.get_transition_condition(get_parent().get_pawn_data())
        if next_state_id != "":
            change_state(next_state_id)
```

## 4. Transições e Condições (As Pontes entre Estados)

Transições são as regras que ditam quando mudar de um estado para outro. Elas podem ser:

- **Baseadas em Tempo**: "Ataque dura 0.5 segundos". (`Timer` ou `await get_tree().create_timer(duration).timeout`)
- **Baseadas em Eventos**: "Player detectado" (sinais de `Area2D` ou `_player_detected()`).
- **Baseadas em Animação**: "Animação de ataque terminou" (`await anim_player.animation_finished`).
- **Baseadas em Input**: "Botão de pulo pressionado" (`Input.is_action_just_pressed("jump")`).

No Nível Avançado, essas condições podem ser definidas como **dados** dentro dos `StateResource`s ou como funções que os `StateResource`s chamam (delegam) do controlador.

## 5. Máquinas de Estado Aninhadas e Paralelas (Níveis Avançados)

### 5.1. FSMs Aninhadas (Hierarchical State Machines - HSM)

Um estado pode ter sua própria FSM interna.

- **Exemplo**: O estado `ATTACK` de um inimigo pode ter sub-estados: `ATTACK_MELEE`, `ATTACK_RANGED`, `ATTACK_CHARGE`. A lógica do FSM principal apenas diz "estou atacando", e o FSM aninhado dentro do estado `ATTACK` cuida dos detalhes de como ele ataca.
- **Vantagens**: Reduz a complexidade do FSM principal, organiza a lógica de sub-comportamentos.

### 5.2. FSMs Paralelas

Múltiplas FSMs independentes rodando simultaneamente na mesma entidade.

- **Exemplo**: Um jogador pode ter uma `MovementFSM` (Idle, Walk, Run, Jump) rodando em paralelo com uma `CombatFSM` (Unarmed, Sword, Bow, Spell). O estado `Sword` na `CombatFSM` não afeta o `Walk` na `MovementFSM`, exceto talvez por regras de animação ou velocidade.
- **Vantagens**: Gerencia comportamentos independentes sem criar um "super-estado" que tenta descrever todas as combinações.

## 6. Estudo de Caso: O StateEngineering (Arquitetura de Contexto)

> **Nível 4: O Fim das Transições Manuais**
>
> Se o Nível 3 (FSM baseada em Resources) é a "graduação", o **StateEngineering** é o Mestrado. Ele resolve o maior problema das máquinas de estado complexas: a **explosão combinatória de transições**.

### 6.1. A Filosofia: Filtragem > Transição

Em uma FSM tradicional, você diz: *"Se estou em Idle e aperto Espaço, vá para Jump"*.
No StateEngineering, você diz: *"O Jogador apertou Espaço. Qual é o melhor estado para agora?"*

Mudamos a pergunta de **"Para onde vou?"** para **"Quem sou eu agora?"**.

Imagine um **Motor de Busca** (como o Google) dentro do seu personagem.
Quando o contexto muda (ex: o jogador aperta Ataque enquanto cai), o sistema pergunta:
> *"Tenho uma Katana. Estou no Ar. Apertei Ataque. O que se encaixa?"*

O sistema filtra sua biblioteca de estados e encontra:
1.  `SwordGroundAttack`? ❌ Rejeitado (Requer: Chão).
2.  `KatanaAirAttack`? ✅ Aceito (Requer: Ar, Katana).

### 6.2. Os Dados: O Resource como Regra (`AttackData`)

Em vez de código, nossos ataques são definidos por **Requisitos**.
Veja como um `AttackData.tres` se parece na prática (simplificado):

```gdscript
class_name AttackData extends Resource

# --- QUEM SOU EU? ---
@export var animation_name: String = "air_slash_v1"
@export var damage: int = 15

# --- QUANDO POSSO EXISTIR? (O Filtro) ---
@export_group("Requirements")
@export var req_motion: StateMachine.Motion = StateMachine.Motion.ANY
@export var req_physics: StateMachine.Physics = StateMachine.Physics.AIR # Só funciona no ar!
@export var req_weapon: StateMachine.Weapon = StateMachine.Weapon.KATANA # Só com Katana!

# --- REGRAS DE SOBREVIVÊNCIA (Reatividade) ---
@export_group("Reaction Rules")
# Se eu tocar no chão no meio do ataque, o que acontece?
@export var on_physics_change: StateMachine.Reaction = StateMachine.Reaction.CANCEL
```

**A Análise do Machi:**
Note que **não há `if (is_on_floor())`** em lugar nenhum. O Resource declara suas necessidades. Se o requisito `req_physics` não bater com o contexto atual, esse estado nem entra na lista de candidatos.

### 6.3. O Cérebro: O Sistema de Pontuação (Score System)

E se tivermos dois estados válidos?
1.  `GenericAirAttack` (Requer: Ar)
2.  `KatanaAirAttack` (Requer: Ar + Katana)

O algoritmo `find_best_match()` no `Machine.gd` não pega o primeiro que acha. Ele dá **Pontos** por especificidade.

- **Match Genérico (ANY):** 0 pontos.
- **Match Exato:** 1 ponto.

**Resultado:** O `KatanaAirAttack` ganha porque é mais específico para a arma atual. Isso permite que você crie um "Ataque Genérico" como fallback e depois "especialize" o jogo criando Resources mais detalhados, sem nunca quebrar o código existente.

### 6.4. Reatividade Declarativa

O maior causador de bugs em jogos de ação é o cancelamento de estados.
*Exemplo: O jogador toma dano no meio de um ataque.*

No jeito antigo, todo estado precisaria checar: `if took_damage: change_state(HURT)`.
No StateEngineering, definimos isso no Resource:

```gdscript
@export var on_take_damage: StateMachine.Reaction = StateMachine.Reaction.CANCEL
```

O `MachineComponent` observa o contexto global. Se o contexto mudar para `Status: STUNNED`, ele olha para o estado atual, vê a regra `CANCEL`, e mata o estado imediatamente.

### Conclusão do Estudo

O StateEngineering remove a necessidade de escrever código de transição.
- Quer um novo ataque? Crie um `.tres`.
- Quer que o ataque pare de funcionar na água? Mude o filtro no Inspector.
- Quer um combo? Crie um estado que requer `Attack: Combo2`.

É a aplicação máxima de **Dados sobre Lógica**.

---



## 7. Conclusão: Disciplina e Flexibilidade

Máquinas de Estado são ferramentas poderosas para impor disciplina ao comportamento do seu jogo. Ao escolher o nível certo de abstração (enum, classes ou Resources), você pode criar sistemas que são fáceis de entender, de depurar e, o mais importante, de estender e balancear, seja você um programador ou um game designer.
