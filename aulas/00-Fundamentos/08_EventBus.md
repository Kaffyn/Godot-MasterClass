# O Grande Desacoplador: Event Bus

À medida que seu jogo cresce, os nós começam a virar um espaguete de referências.
O Player quer atualizar a HUD. O Inimigo quer avisar o GameManager que morreu. O Menu quer pausar o Player.

Se todos conversarem diretamente (`get_node`), seu jogo se torna impossível de manter.

A solução é o **Event Bus** (Barramento de Eventos).

---

## 1. O Conceito

O Event Bus é um **Autoload (Singleton)** que serve apenas para carregar Sinais Globais.
Ele é a "Praça Pública" do seu jogo.

- O Player grita na Praça: "Morri!"
- A HUD ouve na Praça: "Opa, o Player morreu, vou mostrar Game Over."
- O Player não sabe que a HUD existe. A HUD não sabe quem é o Player.

---

## 2. Implementação

1. Crie um script `event_bus.gd` na pasta `systems/`.
2. Adicione em `Project > Project Settings > Autoloads` com o nome `EventBus`.

```gdscript
# event_bus.gd
extends Node

# Sinais Globais
signal player_health_changed(new_hp, max_hp)
signal enemy_died(enemy_type, points)
signal game_paused
signal game_resumed
```

---

## 3. Como Usar

**Emissor (Player):**

```gdscript
func take_damage(amount):
    hp -= amount
    # Emite globalmente. Zero dependência de UI.
    EventBus.player_health_changed.emit(hp, max_hp)
```

**Receptor (HUD):**

```gdscript
func _ready():
    # Conecta no sinal global
    EventBus.player_health_changed.connect(_update_health_bar)

func _update_health_bar(new_hp, max_hp):
    bar.value = new_hp
```

---

## 4. Quando usar (e quando não usar)

- **Use para:** Eventos que afetam sistemas distantes (UI, Audio, Game State, Achievements).
- **Não use para:** Lógica local. Não use o EventBus para o Player avisar a própria Espada para atacar. Use sinais locais para isso.

O Event Bus limpa as dependências cruzadas e permite que você teste sistemas isoladamente.
