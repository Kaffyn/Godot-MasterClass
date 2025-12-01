# Godot MBA: Behavior Engineering

> **De:** Machi
> **Nível:** Arquiteto (Level 4)
> **Objetivo:** Dominar a engenharia de entidades vivas. O Behavior Engineering é o sistema central que define **quem** um personagem é (Atributos, Itens, Progressão) e **o que** ele faz (Ações, Movimento, Combate).

---

## 1. O Domínio do Comportamento

O **Behavior Engineering** é o plugin responsável por dar vida e regras a qualquer personagem do seu jogo (Player, Inimigos, NPCs).

Ele não é apenas uma "State Machine". É um motor completo de RPG e Lógica de Entidade.

### O Que Ele Faz (Escopo)

- **Atributos (Stats):** Vida, Força, Velocidade, Mana. Com suporte a modificadores e histórico.
- **Ações (Actions):** O antigo conceito de "States". Movimento, Pulo, Ataque, Dash. Tudo decidido por contexto.
- **Itens e Inventário (Dados):** A definição do que são os itens e o container lógico que os guarda.
- **Efeitos (Effects):** As consequências de ações (Dano, Cura, Status Effect).
- **Progressão:** Níveis, XP, Árvores de Habilidade.

### O Que Ele NÃO Faz (Fronteiras)

- **NÃO tem UI:** Ele não desenha barras de vida nem slots de inventário. Ele apenas fornece os dados para a UI desenhar.
- **NÃO tem World Data:** Ele não sabe o que é uma "Fase" ou onde estão os Spawns.
- **NÃO tem IA Avançada:** Ele toma decisões reativas ("Devo atacar agora?"), mas não planeja estratégias complexas de navegação (isso é domínio de IA/World).
- **NÃO tem Física:** Ele diz "Mova-se", mas a física real (`move_and_slide`) é delegada ao corpo do personagem.

---

## 2. Arquitetura de Dados (ROP)

Tudo no Behavior Engineering começa com **Resources**.

### 2.1. CharacterProfile (A Ficha)

O Resource mestre que define um personagem.

- **Base Attributes:** Vida: 100, Força: 10.
- **Action Library (Compose):** Lista de ações que ele sabe fazer.
- **Inventory Profile:** Tamanho e regras do inventário inicial.

### 2.2. Atributos e Modificadores

Números soltos (`var damage = 10`) são proibidos.
Usamos um sistema de **Atributos** que suporta **Modificadores**.

- **Attribute:** Objeto que calcula `(Base + Flat) * Multiplicadores`.
- **StatModifier:** Resource que altera um atributo (ex: "Espada de Ferro" dá `+5 Flat` em Força).

### 2.3. Actions (O Novo "State")

Esqueça transições manuais (`Idle -> Run`).
Uma **Action** é um Resource que define um comportamento isolado e seus requisitos.

```gdscript
class_name ActionData extends Resource

@export_group("Requisitos (Filtros)")
# Só posso rodar se estiver no chão e com espada
@export var req_physics: BehaviorTags.Physics = BehaviorTags.Physics.GROUND
@export var req_weapon: BehaviorTags.Weapon = BehaviorTags.Weapon.SWORD

@export_group("Reatividade")
# Se eu sair do chão (cair), tente adaptar (ex: Ataque Aéreo)
@export var on_physics_change: BehaviorTags.Reaction = BehaviorTags.Reaction.ADAPT
```

---

## 3. O Cérebro: BehaviorController

Cada personagem tem um nó `BehaviorController`. Ele é o gerente que conecta tudo.

### 3.1. O Ciclo de Vida

1.  **Inicialização:** Lê o `CharacterProfile`, cria os Atributos e carrega a Biblioteca de Ações.
2.  **Contexto:** Monitora o estado do mundo (`No Chão`, `Com Arma`, `Atordoado`).
3.  **Decisão (Score System):** Quando o jogador aperta um botão, o Controller pergunta:
    > _"Qual ação da minha biblioteca melhor atende aos meus requisitos atuais e ao input?"_

### 3.2. O Algoritmo de Score

Se duas ações servirem (ex: "Ataque Genérico" e "Ataque de Espada"), o sistema dá pontos por especificidade:

- **Match Exato:** +1 Ponto (ex: Requer `SWORD` e tem `SWORD`).
- **Match Genérico:** 0 Pontos (ex: Requer `ANY`).

O mais específico (maior pontuação) vence. Isso permite especializar personagens sem quebrar a lógica base.

---

## 4. Integração de Sistemas

### 4.1. Itens e Inventário

O Inventário é apenas uma lista de dados (`ItemInstance`).
Quando você equipa um item:

1.  O Item injeta **Tags** no Contexto (`Weapon: SWORD`).
2.  O Item aplica **Modificadores** nos Atributos (`+5 Força`).
3.  O `BehaviorController` reavalia as Ações disponíveis (agora ele pode usar "Ataque de Espada").

### 4.2. Efeitos e Consequências

Ações causam Efeitos.

- **Ataque:** Gera um `DamageEffect`.
- **Poção:** Gera um `HealEffect` ou `BuffEffect`.

O `BehaviorController` recebe esses efeitos e os aplica aos Atributos (reduz vida, aumenta força temporariamente).

---

## 5. Implementação de Referência

### A. ActionData (O Comportamento)

```gdscript
class_name ActionData extends Resource

@export var id: String
@export var animation: String
@export var priority: int = 0

# Requisitos para esta ação ser válida
@export var req_tags: Dictionary = {} # {"Physics": "GROUND", "Weapon": "SWORD"}

# Efeitos que esta ação aplica ao executar
@export var effects: Array[Effect]
```

### B. BehaviorController (O Gerente)

```gdscript
class_name BehaviorController extends Node

var attributes: Dictionary # Mapa de Attribute
var context: Dictionary # Mapa de Tags atuais
var action_library: BehaviorCompose

func perform_action(input_tag: String):
    # Busca a melhor ação baseada no input e no contexto atual
    var action = _find_best_action(input_tag)
    if action:
        _execute_action(action)

func apply_effect(effect: Effect):
    # Processa dano, cura ou buffs
    match effect.type:
        EffectType.DAMAGE:
            var hp = attributes["health"]
            hp.current_value -= effect.value
```

---

## 6. Conclusão

O **Behavior Engineering** é a espinha dorsal lógica do personagem. Ele centraliza a matemática (RPG) e o fluxo de decisão (Ações), permitindo que você crie sistemas complexos e profundos apenas manipulando Resources, sem escrever código novo para cada item ou habilidade.
