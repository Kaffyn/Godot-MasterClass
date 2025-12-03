# Composição vs Herança: O Lego da Engenharia

Herança (`extends`) é poderosa, mas tem um problema: **Rigidez**.
Se você cria uma classe `Monstro` que tem `Vida`, e depois quer criar uma `ParedeDestrutivel` que também tem `Vida`, você faz o que? Herda Parede de Monstro? Não faz sentido.

A solução é **Composição**. Em vez de **SER** algo, o objeto **TEM** algo.

---

## 1. O Padrão Componente

Crie nós pequenos e focados que fazem UMA coisa muito bem.

### Exemplo: HealthComponent

Crie uma cena `HealthComponent.tscn` (Node simples).

```gdscript
# health_component.gd
class_name HealthComponent extends Node

signal died
signal health_changed(new_value)

@export var max_health: int = 100
var current_health: int

func _ready():
    current_health = max_health

func take_damage(amount: int):
    current_health -= amount
    health_changed.emit(current_health)
    if current_health <= 0:
        died.emit()
```

Agora, você pode plugar esse `HealthComponent` no Player, no Inimigo, na Parede e até na Caixa de Correio. Todos ganham vida instantaneamente sem herança complexa.

---

## 2. Comunicação entre Componentes

Como o `HitboxComponent` avisa o `HealthComponent` que tomou dano?
Eles são irmãos na árvore.

**Opção A: Export (Injeção)**
No script da Hitbox:

```gdscript
@export var health_component: HealthComponent

func _on_hit(damage):
    if health_component:
        health_component.take_damage(damage)
```

No Inspector, você arrasta o nó de vida para o slot da hitbox.

**Opção B: Sinais**
O Pai (Player) conecta os filhos.

```gdscript
# player.gd
func _ready():
    $Hitbox.hit.connect($Health.take_damage)
```

---

## 3. Vantagens da Composição

1. **Reutilização:** Escreva o código de Vida uma vez, use em 100 objetos diferentes.
2. **Flexibilidade:** Quer que o Player pare de levar dano? Delete o `HealthComponent` (ou desative-o).
3. **Limpeza:** O script do Player foca em Input e Movimento. O script de Vida foca em Vida. O script de Ataque foca em Ataque.

> **Machi Way:** Use Herança para definir "O que é" (CharacterBody2D). Use Composição para definir "Habilidades" (Health, Inventory, Attack).
