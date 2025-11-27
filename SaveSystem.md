# Godot MBA: Sistema de Save/Load Profissional

> **Instrutor:** Machi
> **Objetivo:** Persistir o estado do jogo de forma segura, escalável e versionável. Abandonar a ideia de salvar "a cena inteira".

---

## 1. O Que Salvar? (State vs Scene)

**NUNCA** tente salvar a árvore de nós (`PackedScene.pack(root)`). Isso salva lixo, texturas, scripts e coisas que não deveriam estar no save.

Salve apenas **DADOS**.

- Posição do Player (Vector2)
- Inventário (Array de IDs e Quantidades)
- Flags de progresso ("boss_1_killed": true)
- Stats atuais (HP, XP)

---

## 2. A Estrutura do Arquivo de Save

Usamos um **Dicionário** como raiz do nosso save. É flexível e fácil de converter para JSON ou Binário.

```gdscript
var save_data = {
    "version": "1.0.0",
    "player": {
        "pos_x": 100,
        "pos_y": 250,
        "hp": 80,
        "xp": 1500
    },
    "inventory": [
        {"id": "sword_iron", "qty": 1, "durability": 0.9},
        {"id": "potion_hp", "qty": 5}
    ],
    "world": {
        "level_id": "dungeon_01",
        "chests_opened": ["chest_01", "chest_04"]
    }
}
```

---

## 3. O Gerenciador de Save (`SaveSystem`)

Crie um Autoload.

```gdscript
# save_system.gd
extends Node

const SAVE_PATH = "user://savegame.dat" # "user://" aponta para AppData/Home do usuário
const SECURITY_KEY = "minha_senha_secreta" # Para encriptação básica (opcional)

func save_game(data: Dictionary):
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    if not file:
        push_error("Falha ao salvar jogo!")
        return

    # Opção 1: JSON (Legível, debugável, mas inseguro e lento para dados gigantes)
    # file.store_string(JSON.stringify(data))

    # Opção 2: Binário (Rápido, compacto, difícil de editar na mão) -> Recomendado
    file.store_var(data)

func load_game() -> Dictionary:
    if not FileAccess.file_exists(SAVE_PATH):
        return {} # Retorna vazio se não tem save

    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)

    # Opção 2: Binário
    var data = file.get_var()
    return data
```

---

## 4. O Padrão "Saveable Node"

Como coletar esses dados espalhados pelo jogo?
Não faça o `SaveSystem` percorrer a árvore. Use Grupos.

1. Adicione os objetos relevantes ao grupo "Persist".
2. Obrigue-os a ter métodos `get_save_data()` e `load_save_data(data)`.

```gdscript
# player.gd
func get_save_data() -> Dictionary:
    return {
        "pos_x": global_position.x,
        "pos_y": global_position.y,
        "hp": stats.current_hp
    }

func load_save_data(data: Dictionary):
    global_position.x = data.get("pos_x", 0)
    global_position.y = data.get("pos_y", 0)
    stats.current_hp = data.get("hp", 100)
```

**No momento de salvar:**

```gdscript
func create_save_snapshot() -> Dictionary:
    var snapshot = {}
    var nodes = get_tree().get_nodes_in_group("Persist")

    for node in nodes:
        # Usamos o path do node como ID único (cuidado se nodes mudam de nome!)
        # Ou melhor: Exigimos que cada node tenha um 'export var unique_id: String'
        if node.has_method("get_save_data"):
            snapshot[node.unique_id] = node.get_save_data()

    return snapshot
```

---

## 5. ROP e Save (Salvando Resources)

Se você usa Resources para Inventário (como ensinado no MBA), você não pode salvar o Resource direto com `store_var` se ele tiver Scripts atrelados (perigo de segurança e referência circular).

Converta o Resource para Dicionário antes de salvar.

```gdscript
# item_instance.gd
func serialize() -> Dictionary:
    return {
        "def_path": definition.resource_path, # Salva o caminho do .tres estático
        "qty": quantity,
        "dur": durability
    }

static func deserialize(data: Dictionary) -> ItemInstance:
    var def = load(data["def_path"])
    var item = ItemInstance.new(def)
    item.quantity = data["qty"]
    item.durability = data["dur"]
    return item
```

---

## 6. Versionamento de Save

Se você lançar a v1.0 e depois na v1.1 adicionar um campo novo no player, o save antigo vai quebrar ao tentar ler esse campo.

Sempre use `data.get("campo", valor_padrao)` ao carregar.
E verifique a versão no início do load:

```gdscript
func load_game():
    var data = ...
    if data.version == "1.0.0":
        data = _migrate_v1_to_v2(data)
```
