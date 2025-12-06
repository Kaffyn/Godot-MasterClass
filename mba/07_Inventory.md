# Machi Class: Masterclass de Sistemas de Inventário

> **Instrutor:** Machi
> **Nível:** Básico ao Arquiteto Sênior
> **Objetivo:** Ensinar a evolução dos sistemas de inventário, partindo do código ingênuo até arquiteturas robustas usadas em RPGs complexos e Survival Games.

---

## Módulo 1: A Abordagem Ingênua (O Jeito Errado que Funciona)

### O "Array de Strings"

No começo, todo programador tenta resolver inventários da forma mais literal possível.

**A Lógica:**
"Se eu tenho uma espada, eu guardo o nome dela numa lista."

```gdscript
# player.gd
var inventory: Array[String] = ["Espada de Madeira", "Poção", "Pedra"]

func add_item(item_name: String):
    inventory.append(item_name)

func has_item(item_name: String) -> bool:
    return item_name in inventory
```

**O Problema:**

1. **Erros de Digitação:** Se escrever "pocao" em vez de "Poção", o código quebra silenciosamente.
2. **Dados Pobres:** Como saber o dano da "Espada de Madeira"? Você teria que criar um `match` ou `if/else` gigante em outro lugar.
3. **Sem Estado:** Como saber se a poção está cheia ou vazia? Se a espada está quebrada?

**Conclusão do Módulo:** Strings são para nomes, não para dados estruturados.

---

## Módulo 2: Resources Estáticos (O Padrão ROP Intermediário)

### Recursos Compartilhados (`.tres`)

Aqui introduzimos o conceito de `Resource` como um "Objeto de Dados". Em vez de strings, usamos arquivos.

**A Estrutura:**
Cria-se um script base `ItemData.gd`:

```gdscript
# item_data.gd
extends Resource
class_name ItemData
@export var name: String
@export var icon: Texture2D
@export var weight: float
```

E cria-se arquivos `.tres` no editor: `wood_sword.tres`, `stone.tres`.

**O Inventário:**

```gdscript
# player.gd
@export var inventory: Array[ItemData]
```

**A Limitação Fatal (O "Bug do Clone"):**
Como esses resources são arquivos `.tres` carregados do disco, eles são **compartilhados**.
Se você adicionar uma variável `durabilidade` no `ItemData` e mudar a durabilidade da `wood_sword.tres` em tempo de execução (runtime), **TODAS as espadas de madeira do jogo vão quebrar ao mesmo tempo**.

Isso serve para: Jogos onde itens não têm estado individual (ex: Chaves, Itens de Quest, Colecionáveis simples).
Não serve para: RPGs (Equipamentos com stats variáveis) ou Survival (Durabilidade, Comida estragando).

---

## Módulo 3: O Santo Graal - Resources Vivos (Instâncias Únicas)

### O Modelo "Minecraft" / "Diablo"

Para criar um sistema onde cada item é único, tem durabilidade própria, encantamentos específicos e pode ser movido independentemente, precisamos mudar a arquitetura.

Não usamos mais `.tres` diretos para representar o item no inventário. Usamos o `.tres` apenas como **Molde (Blueprint)**.

#### 3.1 A Arquitetura: Definição vs Instância

Separamos o conceito em dois:

1. **`ItemDefinition` (Estático):** O que _é_ esse item? (Nome, Ícone Padrão, Max Stack, Cena do Drop 3D). Isso nunca muda.
2. **`ItemInstance` (Dinâmico):** O item que _existe_ no bolso do jogador. (Referência à Definição + Quantidade + Durabilidade + Metadados).

**O Script da Instância (O Coração do Sistema):**

```gdscript
# item_instance.gd
extends Resource
class_name ItemInstance

# Qual "tipo" de item é esse? (Referência ao molde estático)
@export var definition: ItemDefinition

# Dados exclusivos DESTA cópia
@export var quantity: int = 1
@export var current_durability: float
@export var custom_name: String = "" # Ex: Jogador renomeou a espada

# Construtor inteligente
func _init(def: ItemDefinition, qty: int = 1):
    definition = def
    quantity = qty
    current_durability = def.max_durability # Começa nova
```

#### 3.2 Crafting e Geração de Loot

Quando o jogador crafta uma espada ou pega um item, não carregamos um arquivo. Nós **instanciamos um novo Resource em memória**.

```gdscript
# crafting_system.gd
func craft_sword():
    var molde = load("res://defs/iron_sword_def.tres") # O Molde

    # Criamos um NOVO objeto na memória RAM. Este é único no universo.
    var nova_espada = ItemInstance.new(molde)

    player.inventory.add(nova_espada)
```

#### 3.3 Comportamento Polimórfico (Armas vs Ferramentas vs Blocos)

Como saber se o item é uma arma ou um bloco? Usamos herança nos Scripts de Definição ou Tags.

##### Opção A: Herança de Definição

```gdscript
# weapon_definition.gd extends ItemDefinition
@export var damage: int
@export var attack_speed: float
```

No inventário: `if item.definition is WeaponDefinition: equip(item)`

##### Opção B: Componentes de Visualização (HUD Própria)

O `ItemInstance` pode saber como se desenhar no inventário.

```gdscript
# item_instance.gd
func get_grid_display_info() -> Dictionary:
    var info = { "icon": definition.icon, "text": "" }

    if definition is WeaponDefinition:
        # Mostra barra de durabilidade
        info["bar_value"] = current_durability / definition.max_durability
        info["bar_color"] = Color.GREEN
    elif definition.max_stack > 1:
        # Mostra contador
        info["text"] = str(quantity)

    return info
```

#### 3.4 O Item no Mundo (Drop/Pickup)

Como transformar esse Resource (dado abstrato) em algo físico no chão?
O `ItemDefinition` deve ter um link para um `PackedScene`.

1. Jogador joga o item fora.
2. Sistema instancia a cena `WorldItem.tscn`.
3. Sistema injeta o `ItemInstance` dentro do `WorldItem`.
4. `WorldItem` lê o `definition.icon` e atualiza seu Sprite3D/2D.

Quando o jogador pega de volta, o `WorldItem` passa o `ItemInstance` (com a durabilidade exata que tinha) de volta pro inventário e se auto-destroi (`queue_free`). Nada se perde.

---

## Conclusão do MBA

| Recurso           | Array de Strings | Resources Estáticos (.tres) | Resources Vivos (Instâncias) |
| :---------------- | :--------------- | :-------------------------- | :--------------------------- |
| **Complexidade**  | Baixa            | Média                       | Alta (Engenharia Real)       |
| **Memória**       | Baixa            | Otimizada (Compartilhada)   | Variável (Instâncias únicas) |
| **Uso Ideal**     | Protótipos       | Keys, Collectibles          | RPGs, Survival, Looters      |
| **Durabilidade?** | Não              | Não (Bug do Clone)          | Sim (Perfeita)               |
| **Crafting?**     | Limitado         | Limitado                    | Ilimitado                    |

Você acabou de aprender a arquitetura usada em jogos como Minecraft, Terraria e Diablo. Bem-vindo à elite.
