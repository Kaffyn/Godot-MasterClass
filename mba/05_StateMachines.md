# Machi Class: Máquinas de Estado - A Arte de Orquestrar o Comportamento

> **De:** Machi
> **Para:** Você, que busca controle e clareza no comportamento do seu jogo.
>
> Comportamentos complexos em jogos rapidamente se transformam em um pesadelo de `if/else` aninhados. Seu inimigo está `parado`, `andando`, `atacando`... E de repente, ele `esconde`, `toma dano` e `morre`. Cada nova condição gera mais uma ramificação, mais um bug. Máquinas de Estado Finitas (FSMs) são a sua salvação. Elas fornecem a estrutura e a disciplina para orquestrar o comportamento de qualquer entidade no seu jogo, do menor inimigo ao jogo inteiro.

---

## 1. O Problema do `if/else` Infinito (Complexidade Ciclomática do Comportamento)

Quando o comportamento de uma entidade é governado por uma longa cadeia de `if/else`, a complexidade escala exponencialmente.

```gdscript
# inimigo_monolitico.gd
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

## 6. Estudo de Caso: O Behavior Engineering (Arquitetura de Comportamento Unificado)

> **Nível 4: A Unificação Total**
>
> Se o Nível 3 (FSM baseada em Resources) é a "graduação", o **Behavior Engineering** é o Mestrado definitivo. Ele unifica _o que_ um personagem é (seus atributos) e _o que_ ele faz (suas ações/estados) em um único domínio. Removemos a fragmentação e resolvemos o problema clássico da **explosão combinatória de transições** de uma vez por todas.

### 6.1. A Filosofia: Filtragem por Contexto e Comportamento

Em uma FSM tradicional, você diz: _"Se estou em Idle e aperto Espaço, vá para Jump"_.
No Behavior Engineering, a pergunta é: _"O Jogador apertou Espaço. Qual é a melhor **Ação** para agora, considerando meu contexto (atributos, equipamento, estado físico)?"_

Mudamos a pergunta de **"Para onde vou?"** para **"Qual comportamento é o mais adequado?"**.

Imagine um **Motor de Busca de Comportamentos** dentro do seu personagem.
Quando o contexto muda (ex: o jogador aperta Ataque enquanto cai e está com pouca vida), o sistema pergunta:

> _"Tenho uma Katana. Estou no Ar. Apertei Ataque. Estou com Pouca Vida. Qual **Ação** (comportamento) se encaixa melhor?"_

O sistema vasculha sua biblioteca de `ActionData` Resources e encontra:

1. `SwordGroundAttack`? ❌ Rejeitado (Requer: Chão).
2. `KatanaAirAttack`? ✅ Aceito (Requer: Ar, Katana).

### 6.2. Os Dados: O Resource como Regra (_ActionData_)

Nossas ações são definidas por `ActionData` Resources, que agora podem incluir requisitos de atributos.
Veja como um `ActionData.tres` se parece na prática (simplificado, combinando State e Behavior):

```gdscript
class_name ActionData extends Resource

# --- QUEM SOU EU? (Identidade do Comportamento) ---
@export var animation_name: String = "air_slash_v1"
@export var base_damage_effect: Effect # O efeito que esta ação causa

# --- QUANDO POSSO EXISTIR? (Requisitos de Contexto e Atributos) ---
@export_group("Requirements")
@export var req_motion_tag: BehaviorTags.Motion = BehaviorTags.Motion.ANY
@export var req_physics_tag: BehaviorTags.Physics = BehaviorTags.Physics.AIR # Só funciona no ar!
@export var req_weapon_tag: BehaviorTags.Weapon = BehaviorTags.Weapon.KATANA # Só com Katana!
@export var req_min_stamina: float = 10.0 # Requisito de Atributo!

# --- REGRAS DE SOBREVIVÊNCIA (Reatividade Declarativa) ---
@export_group("Reaction Rules")
# O que acontece se eu tocar no chão no meio deste ataque aéreo?
@export var on_physics_change_reaction: BehaviorTags.Reaction = BehaviorTags.Reaction.ADAPT
# O que acontece se minha estamina acabar no meio da ação?
@export var on_stamina_low_reaction: BehaviorTags.Reaction = BehaviorTags.Reaction.CANCEL
```

**A Análise do Machi:**
Note que _não há_ `if (get_attribute_value("stamina") < 10)` no seu código. O `ActionData` declara suas necessidades de contexto _e_ de atributos. Se o requisito `req_physics_tag` não bater, ou `req_min_stamina` não for atendido, essa ação nem entra na lista de candidatos.

### 6.3. O Cérebro: Hash Map O(1) e Scoring System

Como o sistema encontra a ação certa sem testar todas as 500 ações da lista?

1. **Indexação (Hash Map O(1)):**
    Ao iniciar, o sistema organiza as ações em "baldes" baseados em suas tags principais (ex: todas as ações de `AIR` vão para o balde `AIR`).
    Quando o contexto muda para `AIR`, o sistema olha _apenas_ para esse balde. O custo de busca é constante, não importa quantas ações existam.

2. **Sistema de Pontuação (Scoring):**
    Dentro do balde, se houver múltiplos candidatos, o sistema roda um concurso de especificidade:

    - **Match Genérico (ANY):** 0 pontos.
    - **Match Exato (Valor Igual):** +1 ponto por cada tag/atributo que corresponde exatamente.

    **Exemplo:**

    - `GenericAirAttack` (Requer: Ar) -> Score 0 (Base)
    - `KatanaAirAttack` (Requer: Ar + Katana) -> Score 1 (Ganhou ponto pela Katana)
    - `DesperateAirStrike` (Requer: Ar + Katana + Vida Baixa) -> Score 2 (Ganhou ponto pela Vida Baixa)

    **Resultado:** O `DesperateAirStrike` vence automaticamente se todas as condições forem atendidas. Isso permite "especialização progressiva" sem quebrar o código base.

### 6.4. Reatividade Declarativa e Processamento de Efeitos

O maior causador de bugs em jogos de ação é o cancelamento de ações ou o processamento de efeitos.
_Exemplo: O jogador toma dano no meio de um ataque._

No jeito antigo, todo estado precisaria checar: `if took_damage: change_state(HURT)`.
No Behavior Engineering, definimos isso declarativamente no `ActionData`:

```gdscript
@export var on_take_damage_reaction: BehaviorTags.Reaction = BehaviorTags.Reaction.CANCEL
```

O `BehaviorController` observa o contexto global (incluindo mudanças em atributos). Se o contexto mudar para `Status: STUNNED` (ou se o HP cair para zero), ele olha para a ação atual, vê a regra `CANCEL`, e mata a ação imediatamente.

Além disso, a `ActionData` agora contém referências a `Effect` Resources, que são processados diretamente pelo `BehaviorController` do alvo.

### Conclusão do Estudo

O Behavior Engineering remove a necessidade de escrever código de transição e de gerenciamento manual de atributos.

- Quer um novo ataque que também aplique um veneno? Crie um `ActionData.tres` e injete um `PoisonEffect.tres`.
- Quer que o ataque pare de funcionar se a estamina acabar? Adicione `req_min_stamina` e uma `on_stamina_low_reaction`.

É a aplicação máxima de **Dados sobre Lógica** para o comportamento completo do jogo.

---

## 7. Conclusão: Disciplina e Flexibilidade

Máquinas de Estado são ferramentas poderosas para impor disciplina ao comportamento do seu jogo. Ao escolher o nível certo de abstração (enum, classes ou Resources), você pode criar sistemas que são fáceis de entender, de depurar e, o mais importante, de estender e balancear, seja você um programador ou um game designer.
