# Resource-Oriented Programming (ROP): O Segredo da Godot

Se você entender esta aula, você estará à frente de 90% dos usuários de Godot.
ROP é a filosofia de usar `Resources` não apenas para dados passivos (como Sprites), mas para **Dados de Gameplay e Lógica**.

---

## 1. O que é um Custom Resource?

É uma classe de dados que você cria.

```gdscript
# item_data.gd
class_name ItemData extends Resource

@export var name: String = "Espada"
@export var damage: int = 10
@export var icon: Texture2D
@export var is_rare: bool = false
```

Ao salvar esse script, você criou um novo **Tipo** na Godot.
Agora, no FileSystem, clique com botão direito -> `New Resource` -> Procure por `ItemData`.
Você pode criar `espada_fogo.tres`, `adaga_gelo.tres`, etc.

---

## 2. Por que isso é genial? (Data-Driven Design)

1. **Edição Visual:** Você ajusta o balanceamento do jogo (dano, preço) no Inspector, sem tocar no código. Game Designers amam isso.
2. **Compartilhamento:** 50 Goblins podem usar o mesmo `goblin_stats.tres`. Se você aumentar o HP no arquivo, todos os 50 goblins ficam mais fortes instantaneamente. Economiza memória RAM.
3. **Hot-Swap:** Você pode trocar a arma do player apenas trocando o Resource.

```gdscript
# player.gd
@export var weapon: ItemData

func attack():
    print("Causei " + str(weapon.damage) + " de dano com " + weapon.name)
```

---

## 3. Resources com Comportamento

Resources podem ter funções! Desde que não dependam da SceneTree (não usem `get_node` ou `$`).

```gdscript
# potion_data.gd
class_name PotionData extends Resource

@export var heal_amount: int = 20

func get_description() -> String:
    return "Cura " + str(heal_amount) + " de vida."
```

Isso permite criar sistemas de Habilidades, IAs, e Quests onde a lógica está encapsulada no dado.

---

## 4. O Perigo do "Local to Scene"

Por padrão, Resources são compartilhados.
Se você mudar `goblin_stats.hp = 0` em um goblin, **TODOS** os goblins morrem, porque todos apontam para o mesmo arquivo na memória.

Se você precisa que um Resource seja único para cada instância (ex: um Inventário que muda durante o jogo), marque a opção **Local to Scene** no Inspector do Resource (dentro da cena onde ele é usado) ou duplique-o via código (`resource.duplicate()`).

> **Machi Way:** Use Resources para tudo que é **Definição** (O que é uma Espada?). Use Nodes ou Dictionaries para **Estado** (Quantas balas tem na arma agora?).
