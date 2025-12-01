# Godot MBA: State Engineering

> **De:** Machi
> **Nível:** Arquiteto (Level 4)
> **Objetivo:** Abandonar as transições manuais de estado. Dominar a arquitetura de **Filtro Contextual** e **Score System** para criar sistemas de combate e movimento que escalam infinitamente sem `if/else`.

---

## 1. O Fim do "Caos de Transições"

Você já escreveu uma State Machine tradicional. Você sabe como funciona:
_"Se estou Idle e aperto Espaço, vou para Jump"_.
_"Se estou Jump e toco no chão, vou para Idle"_.

Isso funciona bem até você ter 50 estados.
De repente, você tem que conectar `Idle`, `Run`, `Walk`, `Crouch` para `Jump`. E todos eles precisam saber que `Jump` existe. Se você adicionar um `DoubleJump`, precisa editar 4 arquivos diferentes.

O **State Engineering** propõe uma inversão completa de controle.

**A Nova Filosofia:**
Não diga para onde ir. Diga **"Quem sou eu agora?"**.

Em vez de transições rígidas (`Idle -> Attack`), nós usamos um **Motor de Busca**.
Quando o jogador aperta "Ataque", o sistema pergunta:

> "Dentre as 200 animações de ataque que tenho no disco, qual delas serve para um personagem que está **Segurando uma Katana**, está **No Ar** e tem **Stamina > 10**?"

Se só existir uma, é essa que roda. Se existirem duas, a mais específica ganha.

---

## 2. A Arquitetura da Solução

O sistema é composto por três pilares. Entenda isso e você entenderá tudo.

### A. Machine (O Cérebro)

É o componente (`Node`) que vive no Player. Ele não sabe as regras do jogo. Ele só sabe **filtrar**.
Ele mantém o **Contexto Atual** (um dicionário de fatos):

- `Weapon: SWORD`
- `Physics: AIR`
- `Motion: MOVING`

### B. Data (A Regra)

São seus `Resources` (`.tres`). Cada ataque, cada movimento, é um arquivo.
Eles não têm código de lógica. Eles têm **Metadados de Requisito**.

- `AirSlash.tres` diz: "Eu preciso de `Physics: AIR` e `Weapon: SWORD`".

### C. Compose (O Banco de Dados)

É apenas uma lista (`Resource`) que agrupa todos os `Data` que um personagem conhece.
O Player tem um `PlayerAttacks.tres` que lista todos os ataques que ele aprendeu.

---

## 3. O Algoritmo de "Score System"

Como o `Machine` escolhe o estado certo? Ele roda um concurso.

Imagine que o contexto é: **Arma = Espada**, **Física = Chão**. O jogador aperta Ataque.

| Candidato         | Requisitos           | Resultado   | Pontos              |
| :---------------- | :------------------- | :---------- | :------------------ |
| `Soco Básico`     | `Arma: Any`          | ✅ Passou    | 0                   |
| `Corte de Espada` | `Arma: Espada`       | ✅ Passou    | **1** (Match Exato) |
| `Tiro de Pistola` | `Arma: Arma de Fogo` | ❌ Rejeitado | -                   |

**Vencedor:** `Corte de Espada`.

**Por que isso é genial?**
Porque permite **Polimorfismo de Dados**.
Você pode começar o jogo apenas com o `Soco Básico`. Quando o jogador equipa a espada, o contexto muda. O `Machine` automaticamente começa a selecionar o `Corte de Espada` sem você mudar uma linha de código na máquina de estados.

---

## 4. Implementação de Referência

Aqui está como implementamos isso na Kaffyn. Use isso como base para sua própria engine.

### 4.1. Core: O Vocabulário (`StateMachine.gd`)

Primeiro, definimos a linguagem que o jogo fala. Isso deve ser um Autoload ou uma classe estática.

```gdscript
# core/state_machine.gd
@tool
extends Node

# O coringa. Significa "Não me importo com esse valor".
const ANY = -1

# Categorias de Contexto
enum Motion { ANY = -1, IDLE, WALK, RUN, DASH }
enum Attack { ANY = -1, NONE, FAST, HEAVY, SPECIAL }
enum Physics { ANY = -1, GROUND, AIR, WATER }
enum Weapon { ANY = -1, NONE, SWORD, BOW, MAGIC }

# Regras de Reação (O que fazer se o contexto mudar?)
enum Reaction {
    IGNORE,     # Continue rodando o estado atual
    CANCEL,     # Pare agora (ex: tomou dano)
    ADAPT,      # Tente achar um estado novo que sirva (ex: pulou com a espada -> ataque aéreo)
}
```

### 4.2. Data: A Definição do Ataque (`AttackData.gd`)

Este é o script que vai nos seus `.tres`. Note como ele é puramente declarativo.

```gdscript
# data/resources/attack_data.gd
@tool
class_name AttackData extends Resource

@export_group("Identidade")
@export var animation_name: String
@export var damage: int = 10

@export_group("Filtros (Requisitos)")
# Se o contexto não bater com isso, este ataque é invisível.
@export var req_weapon: StateMachine.Weapon = StateMachine.Weapon.ANY
@export var req_physics: StateMachine.Physics = StateMachine.Physics.ANY
@export var req_motion: StateMachine.Motion = StateMachine.Motion.ANY

@export_group("Reatividade")
# Se eu cair no chão enquanto este ataque aéreo roda, o que acontece?
@export var on_physics_change: StateMachine.Reaction = StateMachine.Reaction.ADAPT

@export_group("Cooldowns")
# Impede que certas ações (ex: Dash) sejam spammadas
@export var context_cooldown_filter: StateMachine.ContextFilter = StateMachine.ContextFilter.NONE
@export var context_cooldown_time: float = 0.0
```

### 4.3. Compose: O Agrupador (`StateCompose.gd`)

Este é o "Deck de Cartas" da entidade. Em vez de arrastar 50 ataques para o Player, você arrasta **um** arquivo `PlayerAttacks.tres` que contém a lista. Isso permite que inimigos diferentes compartilhem o mesmo "Move Set" básico, apenas trocando o Resource de Compose.

```gdscript
# core/state_compose.gd
@tool
class_name StateCompose extends Resource

@export var states: Array[Resource]

# Cache para acesso rápido (Opcional)
func get_all() -> Array[Resource]:
    return states
```

### 4.4. Machine: O Motor de Busca (`Machine.gd`)

A lógica pesada fica aqui. O `Machine` consome um `Compose` para encontrar a melhor resposta.

```gdscript
# core/machine.gd
class_name Machine extends Node

# Contexto atual (O "Estado do Mundo" para esta entidade)
var context: Dictionary = {}
var current_state: Resource

func set_context(category: String, value: int) -> void:
    if context.get(category) == value: return
    context[category] = value
    _on_context_updated(category, value)

# O Algoritmo de Ouro
func find_best_match(compose: StateCompose) -> Resource:
    if not compose: return null

    var best_res: Resource = null
    var best_score: int = -1

    # Itera sobre o "Deck" de estados fornecido
    for res in compose.get_all():
        var score = 0
        var possible = true

        # Verifica cada requisito do Resource contra o Contexto Atual
        if "req_weapon" in res:
            if res.req_weapon != StateMachine.ANY:
                if res.req_weapon != context.get("Weapon"):
                    possible = false # Falhou no requisito rígido
                else:
                    score += 1 # Ganhou ponto por especificidade!

        if "req_physics" in res:
            if res.req_physics != StateMachine.ANY:
                if res.req_physics != context.get("Physics"):
                    possible = false
                else:
                    score += 1

        if possible and score > best_score:
            best_score = score
            best_res = res

    return best_res
```

---

## 5. Aplicando na Prática: O Player

Como conectamos isso no personagem? Note como o código fica limpo.

```gdscript
# machines/player_machine.gd
class_name PlayerMachine extends Machine

# AQUI ESTÁ A MÁGICA:
# Você não arrasta ataques individuais. Você arrasta um "Pacote de Ataques".
# Se quiser criar um "Dark Player", basta duplicar o .tres do compose,
# trocar 2 ataques e arrastar para o novo inimigo.
@export var attack_library: StateCompose

func _ready():
    # Estado inicial
    set_context("Weapon", StateMachine.Weapon.SWORD)
    set_context("Physics", StateMachine.Physics.GROUND)

func _unhandled_input(event):
    if event.is_action_pressed("attack"):
        _try_attack()

func _try_attack():
    # O Player pede: "Dentro da minha biblioteca, o que serve para agora?"
    var best = find_best_match(attack_library)
    if best:
        change_state(best)
    else:
        print("Nenhum ataque disponível para este contexto.")

func _on_physics_process(delta):
    # Atualiza o contexto de física automaticamente
    var is_grounded = owner.is_on_floor()
    set_context("Physics",
        StateMachine.Physics.GROUND if is_grounded else StateMachine.Physics.AIR
    )
```

## 6. Conclusão do Módulo

O **State Engineering** transforma o caos de `if/else` em uma lista organizada de **Regras**.

- O Game Designer pode criar um novo ataque de "Espada de Fogo que só funciona no Ar" criando um arquivo `.tres` e configurando `req_weapon: FIRE_SWORD` e `req_physics: AIR`.
- O Programador nunca mais precisa abrir o script do Player para "conectar" esse novo ataque. O sistema o encontra automaticamente.

Isso é arquitetura de verdade.
