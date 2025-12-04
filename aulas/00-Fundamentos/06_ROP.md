# Resource-Oriented Programming (ROP): O Segredo da Godot

Se você entender esta aula, você estará à frente de 90% dos usuários de Godot.
ROP é a filosofia de usar `Resources` não apenas para dados passivos (como Sprites), mas para **Dados de Gameplay e Lógica**.

---

## 1. A Realidade Oculta (Você já usa Resources)

A Godot é construída sobre eles.

Quando você arrasta uma imagem para um Sprite, você está usando um Resource.
Quando você escolhe uma fonte para um Label, você está usando um Resource.

**Exemplos Nativos que você usa todo dia:**

1. **`Texture2D` (.png, .jpg):** Dados de imagem.
2. **`AudioStream` (.wav, .ogg):** Dados de som.
3. **`Font` (.ttf):** Dados de tipografia.
4. **`StyleBox`:** Define a aparência de botões e painéis (bordas, cores).
5. **`Theme`:** Um pacote de StyleBoxes e Fontes para toda a UI.
6. **`Shape2D` (Rectangle, Circle):** A forma física de um colisor.
7. **`Animation`:** Dados de keyframes e curvas de tempo.
8. **`TileSet`:** A biblioteca de tiles e suas colisões.
9. **`Material` (ShaderMaterial):** Instruções gráficas para a GPU.
10. **`Curve`:** Uma curva matemática (usada em partículas ou tweens).
11. **`Gradient`:** Uma transição de cores.
12. **`PackedScene` (.tscn):** Sim, a própria cena salva em disco é um Resource!

**A Grande Vantagem (Flyweight Pattern):**
Se 500 inimigos usam a mesma `Texture2D` ("goblin.png"), a Godot carrega a imagem na memória **apenas uma vez**. Todos os 500 inimigos apontam para o mesmo lugar na memória.
Isso é eficiência pura. E você ganha isso de graça ao criar seus próprios Resources.

---

## 2. O que é um Custom Resource?

Agora que você sabe que Resources são apenas "containers de dados eficientes", vamos criar os **nossos**.

É uma classe de dados que você define.

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

## 3. Por que isso é genial? (Data-Driven Design)

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

## 4. Resources com Comportamento

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

## 5. O Perigo do "Local to Scene"

Por padrão, Resources são compartilhados.
Se você mudar `goblin_stats.hp = 0` em um goblin, **TODOS** os goblins morrem, porque todos apontam para o mesmo arquivo na memória.

Se você precisa que um Resource seja único para cada instância (ex: um Inventário que muda durante o jogo), marque a opção **Local to Scene** no Inspector do Resource (dentro da cena onde ele é usado) ou duplique-o via código (`resource.duplicate()`).

> **Machi Way:** Use Resources para tudo que é **Definição** (O que é uma Espada?). Use Nodes ou Dictionaries para **Estado** (Quantas balas tem na arma agora?).
