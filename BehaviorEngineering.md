# Godot MBA: Behavior Engineering

> **De:** Machi
> **Nível:** Arquiteto (Level 4)
> **Objetivo:** Criar um motor de RPG robusto. Sair de `var strength = 10` para um sistema de **Atributos**, **Modificadores** e **Efeitos** que permite buffs, debuffs, equipamentos e escalabilidade infinita.

---

## 1. O Problema dos "Números Soltos"

Em jogos simples, você faz:
```gdscript
var damage = 10
var speed = 100
```

Em um RPG ou jogo sistêmico, isso quebra rápido.
- O jogador pega uma espada (+5 Dano).
- Toma uma poção (+10% Dano por 30s).
- Entra na lama (-50% Speed).
- Upa de nível (Base Dano aumenta).

Se você tentar gerenciar isso com variáveis soltas (`base_damage`, `bonus_damage`, `temp_damage`), você cria um inferno de manutenção. Você precisa de **Behavior Engineering**.

---

## 2. A Arquitetura do Sistema de Stats

Não guarde números. Guarde **Históricos de Modificação**.

### 2.1. Attribute (O Container Inteligente)
Um Atributo não é um `int`. É um Objeto que sabe calcular seu valor final.

Fórmula Universal de RPG:
`Valor Final = (Base + Add) * Multiplicadores`

### 2.2. Modifier (A Regra - Resource)
Em vez de mudar o valor diretamente (`damage += 5`), nós **aplicamos um Modificador**.
Isso permite rastrear a origem ("Isso veio da Espada", "Isso veio da Poção") e remover depois.

```gdscript
class_name StatModifier extends Resource

enum Type { FLAT, PERCENT_ADD, PERCENT_MULT }
@export var id: String = "modifier_id"
@export var stat_key: String = "strength"
@export var type: Type = Type.FLAT
@export var value: float = 10.0
```

### 2.3. StatsComponent (O Gerente)
Um nó no Player que segura todos os atributos.
Quando alguém pede `stats.get_value("strength")`, ele roda o cálculo.

---

## 3. Arquitetura de Efeitos (O "O Que Acontece")

O Behavior Engineering também resolve **Ações**.
Um ataque não causa "dano". Um ataque aplica um **Efeito de Dano**.

### 3.1. Effect (Resource)
Define *o que* acontece, mas não *como*.

```gdscript
class_name Effect extends Resource

@export var id: String = "fire_damage"
@export var type: String = "damage"
@export var value: float = 10.0
@export var element: String = "fire"
```

### 3.2. Integração com State Engineering
Aqui a mágica acontece.
- O **State Engineering** (Machine) decide **QUANDO** atacar (animação, hitbox).
- O **Behavior Engineering** decide **QUANTO** dói (cálculo de stats).

O `AttackData` (do State) carrega um `Effect` (do Behavior).
Quando a espada bate:
1. `Machine` detecta colisão.
2. `Machine` pega o `Effect` do ataque atual.
3. `Machine` passa o `Effect` para o `StatsComponent` do alvo.
4. `StatsComponent` do alvo processa (aplica armadura, resistências) e reduz o HP.

---

## 4. Implementação de Referência

### 4.1. Attribute Class (Inner Class ou Helper)

```gdscript
class Attribute:
	var base_value: float
	var _modifiers: Array[StatModifier] = []
	var _cached_value: float
	var _is_dirty: bool = true

	func get_value() -> float:
		if _is_dirty: _recalculate()
		return _cached_value

	func add_modifier(mod: StatModifier):
		_modifiers.append(mod)
		_is_dirty = true

	func remove_modifier(mod: StatModifier):
		_modifiers.erase(mod)
		_is_dirty = true

	func _recalculate():
		var flat = 0.0
		var mult = 1.0
		
		for mod in _modifiers:
			if mod.type == StatModifier.Type.FLAT: flat += mod.value
			elif mod.type == StatModifier.Type.PERCENT_ADD: mult += mod.value # Ex: 0.1 para +10%
			
		_cached_value = (base_value + flat) * mult
		_is_dirty = false
```

### 4.2. StatsComponent (O Nó)

```gdscript
class_name StatsComponent extends Node

# Dicionário de Atributos (String -> Attribute)
var attributes: Dictionary = {}

func get_stat(key: String) -> float:
	if attributes.has(key):
		return attributes[key].get_value()
	return 0.0

func add_modifier(mod: StatModifier):
	if attributes.has(mod.stat_key):
		attributes[mod.stat_key].add_modifier(mod)

# Exemplo de Processamento de Efeito
func apply_effect(effect: Effect, source_stats: StatsComponent):
	if effect.type == "damage":
		var dmg = effect.value
		
		# Exemplo: Adicionar Força do atacante
		if source_stats:
			dmg += source_stats.get_stat("strength") * 0.5
			
		# Exemplo: Reduzir Defesa do alvo (eu)
		dmg -= get_stat("defense")
		
		_take_damage(max(0, dmg))

func _take_damage(amount: float):
	var hp = attributes["health"]
	hp.base_value -= amount # Dano altera o BASE, não é um modificador temporário
	print("Took damage: ", amount)
```

---

## 5. Conclusão

O **Behavior Engineering** transforma a matemática do jogo em **Dados**.
- Uma "Maldição" não é um script que roda a cada segundo. É um `Modifier` (-5 Str) anexado ao `StatsComponent`.
- Quando a maldição acaba, removemos o `Modifier`, e o `Attribute` se recalcula automaticamente para o valor original.
- Zero bugs de "o status ficou negativo pra sempre".

Isso é controle total sobre a progressão do jogo.
