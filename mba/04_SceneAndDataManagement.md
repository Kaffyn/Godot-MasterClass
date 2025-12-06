# Machi Class: Gestão de Cenas e Persistência de Dados

> **Instrutor:** Machi
> **Objetivo:** Unificar os conceitos de fluxo de cenas, carregamento de dados e salvamento de estado para criar uma arquitetura de jogo robusta e escalável.

---

## Gerenciamento e Persistência de Dados: A Espinha Dorsal do seu Jogo

> **De:** Machi
> **Para:** Você, que está prestes a parar de tratar dados como um detalhe.
>
> A lógica do seu jogo é o músculo. Os gráficos são a pele. Mas os dados são a espinha dorsal. Uma espinha dorsal fraca ou malformada e todo o corpo desmorona no primeiro sinal de estresse. Aprenda a arquitetar seus dados e você construirá sistemas que não apenas funcionam, mas que duram e escalam.

### 1. A Separação Sagrada de Dados e Lógica

Um dos saltos mais importantes para um desenvolvedor amadurecer é entender a **separação entre dados e lógica**.

- **Dados** são a informação, o "quê". _Exemplo: A vida máxima de um personagem é 100, sua velocidade é 50, o dano da sua espada é 15._
- **Lógica** é o comportamento, o "como". _Exemplo: A função `take_damage(amount)` que subtrai vida, o código que move o `CharacterBody2D`._

Um projeto onde os dados estão misturados com a lógica (ex: `var max_hp = 100` dentro do `player.gd`) é um pesadelo de manutenção. Se um game designer quiser balancear o jogo e mudar o HP de 100 para 120, ele precisará editar um script, com grande risco de quebrar a lógica.

A arquitetura profissional isola os dados em arquivos próprios, permitindo que a equipe de design itere e balanceie o jogo sem precisar de um programador.

### 2. O Arsenal de Dados da Godot: Um Guia Comparativo

A Godot oferece múltiplas ferramentas para gerenciar dados. Saber quando usar cada uma é crucial.

| Ferramenta              | O que é?                                                  | Quando Usar (O Jeito Machi)                                                                                                                                               | Quando NÃO Usar                                                                                                       |
| :---------------------- | :-------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------- |
| **Resource (`.tres`)**  | Container de dados nativo, tipado e visível no Inspector. | **A ferramenta padrão para 90% dos casos.** Definição de itens, stats de inimigos, skills, configurações de níveis. O coração da Programação Orientada a Resources (ROP). | Dados de runtime que mudam a todo instante (use Dicionários). Dados para ferramentas externas (use JSON).             |
| **ConfigFile (`.cfg`)** | Arquivo de texto simples no formato INI (chave=valor).    | **Configurações do usuário.** Volume de áudio, qualidade gráfica, mapeamento de teclas. É simples e legível por humanos.                                                  | Estruturas de dados complexas ou aninhadas. Definição de conteúdo de jogo (use Resources).                            |
| **JSON (`.json`)**      | Formato de texto universal para troca de dados.           | **Comunicação externa.** APIs web, importação/exportação de dados com ferramentas que não são a Godot (ex: uma planilha de diálogos, um editor de níveis externo).        | Dados que só a Godot usa. JSON não entende tipos nativos como `Vector2`, `Color` ou referências a outros `Resource`s. |
| **Dictionary**          | Estrutura chave-valor dinâmica, em memória.               | **Estado de Runtime.** O inventário _atual_ do jogador, quests ativas, estado do mundo. É o formato perfeito para ser serializado em um arquivo de save.                  | Para definir conteúdo de jogo (falta de tipagem, não é amigável para designers no Inspector).                         |
| **Array**               | Lista ordenada de itens, em memória.                      | Quando a **ordem importa**. Caminhos de patrulha, sequências de diálogo, um baralho de cartas. Use `Array[Tipo]` para segurança.                                          | Quando você precisa de acesso rápido por uma chave única (use Dictionary).                                            |

### 3. Uma Analogia de Banco de Dados: Resource (Schema) vs. JSON (Tabela)

Para solidificar o entendimento, vamos usar uma analogia do mundo de banco de dados.

Pense em um `JSON` ou um `Dictionary` como uma **tabela de dados** ou uma planilha. Ele é excelente para armazenar e transportar linhas e colunas de informação de forma universal. Qual o propósito original do JSON? Estruturar dados para JavaScript. Sua função é ser facilmente convertido em um objeto genérico em outra linguagem. Ele é um formato de intercâmbio, um "passaporte de dados".

Agora, pense em um `Resource` da Godot como um **Schema de Banco de Dados** completo. Ele é arquiteturalmente superior por várias razões:

1. **Ele Contém as Tabelas**: Um `Resource` pode, obviamente, conter todos os dados que uma tabela/JSON conteria.
2. **Ele Define as Regras entre Elas**: Mais importante, ele define a **estrutura**, os **tipos de dados** (`int`, `String`, `Vector2`), as **regras de validação** (via setters/getters) e os **relacionamentos** entre diferentes "tabelas" (um `Resource` contendo referências a outros `Resource`s).
3. **Ele é um Objeto Inteligente, não um Saco de Dados**: Quando a Godot carrega um `JSON`, você recebe um `Dictionary` genérico. Quando a Godot carrega um `.tres`, ela instancia um **objeto de uma classe específica**, com seus próprios métodos, herança e funcionalidades. Ele não é apenas "parseado", ele é "instanciado".

Enquanto o `JSON` foi criado para ser transformado em objetos JavaScript genéricos, o `Resource` foi criado para ser a espinha dorsal de um ecossistema de objetos de jogo ricos e interconectados, em um padrão de excelência muito superior para o design de games.

### 4. O Padrão Ouro: ROP (Resource-Oriented Programming) em Ação

A filosofia do Machi Class e da SoftEngine é clara: **use `Resource`s para tudo que for conteúdo de design**. Isso significa que, em vez de apenas guardar dados, os `Resource`s se tornam blocos de construção.

O poder real vem da **composição**: `Resource`s que contêm outros `Resource`s.

**Exemplo Prático:**

1. **Você cria um `WeaponData.gd`** que herda de `Resource`. Ele tem `@export var damage: int`.
2. **Você cria um `PlayerProfile.gd`** que também herda de `Resource`. Dentro dele, você declara: `@export var equipped_weapon: WeaponData`.

No editor, você agora pode ter vários arquivos:

- `sword.tres` (dano = 15)
- `axe.tres` (dano = 25)
- `player_warrior.tres` (com o `axe.tres` no slot `equipped_weapon`)
- `player_rogue.tres` (com uma `dagger.tres` no slot `equipped_weapon`)

Um designer pode criar centenas de combinações de personagens e armas **sem escrever uma linha de código**, apenas criando e arrastando esses arquivos `.tres` no Inspector. Isso é ROP em sua forma mais pura.

### 5. Persistência de Dados: Salvando o Estado do Jogo

Arquitetar seus dados é apenas metade da batalha. A outra metade é garantir que o estado do seu jogo possa ser salvo e carregado de forma confiável, segura e flexível.

#### 5.1. O Que Salvar? (Estado vs. Cena)

**NUNCA** tente salvar a árvore de nós (`PackedScene.pack(root)`). Isso salva lixo, texturas, scripts e coisas que não deveriam estar no save.

Salve apenas **DADOS DE ESTADO**:

- Posição do Player (`Vector2` ou `Vector3`)
- Inventário (Array de IDs e Quantidades)
- Flags de progresso (`"boss_1_killed": true`)
- Stats atuais (HP, XP, Nível)
- **Behavior State:** Cooldowns ativos e Contexto persistente (se necessário).

#### 5.2. A Estrutura Interna do Arquivo de Save

Usamos um **Dictionary** como raiz do nosso save. É flexível e fácil de serializar.

```gdscript
var save_data = {
    "version": "1.0.0", # Essencial para migração de saves antigos
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

#### 5.3. O Gerenciador de Save (`SaveManager` Autoload)

A espinha dorsal do seu sistema de persistência será um **Autoload** (`SaveManager`). Ele centraliza a API de save/load.

```gdscript
# save_manager.gd (Configurado como Autoload "SaveManager")
extends Node

const SAVE_FILE_NAME = "savegame.dat"
const SAVE_PATH = "user://" + SAVE_FILE_NAME # "user://" aponta para AppData/Home do usuário
const SECURITY_KEY = "sua_chave_secreta_aqui" # Para encriptação básica (opcional)

func save_game(data: Dictionary) -> void:
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    if not file:
        push_error("SaveManager: Falha ao abrir arquivo para salvar!")
        return

    # Opção Recomendada (Binário Godot): Rápido, compacto, preserva tipos Godot.
    file.store_var(data)
    # Opção JSON (Legível, mas menos eficiente para grandes saves e não preserva tipos Godot):
    # file.store_string(JSON.stringify(data))

    file.close()
    print("Jogo salvo com sucesso em: ", SAVE_PATH)

func load_game() -> Dictionary:
    if not FileAccess.file_exists(SAVE_PATH):
        print("SaveManager: Nenhum arquivo de save encontrado.")
        return {} # Retorna vazio se não houver save

    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    var data = file.get_var() # Para Binário
    # var data = JSON.parse_string(file.get_as_text()) # Para JSON
    file.close()

    print("Jogo carregado com sucesso de: ", SAVE_PATH)
    return data
```

#### 5.4. O Padrão "Saveable Node" (Coletando Dados)

Como o `SaveManager` coleta os dados que estão espalhados por todo o jogo?
Não faça o `SaveManager` vasculhar a árvore de cenas. Use um padrão onde os próprios nós se declaram "salváveis".

1. **Adicione ao Grupo "Persist"**: Adicione todos os nós cujo estado precisa ser salvo a um grupo chamado "Persist".
2. **Contrato de Funções**: Obrigue esses nós a ter métodos `get_save_data() -> Dictionary` e `load_save_data(data: Dictionary) -> void`.

_Exemplo para o Player:_

```gdscript
# player.gd
extends CharacterBody2D

func _ready():
    add_to_group("Persist") # Adiciona-se ao grupo para ser encontrado

@export var unique_id: String = "player_01" # Essencial para identificar o nó no save

func get_save_data() -> Dictionary:
    return {
        "unique_id": unique_id,
        "pos_x": global_position.x,
        "pos_y": global_position.y,
        "hp": current_hp, # Assumindo que current_hp é uma variável de estado do Player
        # ... outros dados de estado
    }

func load_save_data(data: Dictionary):
    global_position.x = data.get("pos_x", global_position.x)
    global_position.y = data.get("pos_y", global_position.y)
    current_hp = data.get("hp", max_hp)
    # ... carrega outros dados de estado
```

_Exemplo de como o `SaveManager` coletaria os dados:_

```gdscript
# save_manager.gd (continuação)
func create_save_snapshot() -> Dictionary:
    var snapshot = {}
    var nodes_to_save = get_tree().get_nodes_in_group("Persist")

    for node in nodes_to_save:
        # Cada nó deve ter um unique_id para evitar conflitos no dicionário
        if node.has_method("get_save_data") and node.has_method("load_save_data") and node.has_node("unique_id"): #unique_id as property
            snapshot[node.unique_id] = node.get_save_data()

    return snapshot

func apply_save_snapshot(snapshot: Dictionary):
    var nodes_to_load = get_tree().get_nodes_in_group("Persist")
    for node in nodes_to_load:
        if node.has_node("unique_id") and snapshot.has(node.unique_id):
            node.load_save_data(snapshot[node.unique_id])
```

#### 5.5. ROP e Save (Serializando Resources)

Se você usa `Resources` para o estado de runtime (ex: um `InventoryResource` que guarda `ItemInstance`s), você não pode simplesmente salvar o `Resource` direto com `store_var` se ele tiver scripts atrelados ou referências à SceneTree (risco de segurança, referência circular).

**A Solução:** Converta o `Resource` para um `Dictionary` antes de salvar.

```gdscript
# item_instance.gd (continuação)
# Função para converter a instância de Item para um Dicionário serializável
func to_dictionary() -> Dictionary:
    return {
        "definition_path": definition.resource_path, # Salva o caminho do .tres estático
        "quantity": quantity,
        "current_durability": current_durability
        # ... outros dados da instância
    }

# Função estática para reconstruir a instância de Item a partir de um Dicionário
static func from_dictionary(data: Dictionary) -> ItemInstance:
    var def = load(data["definition_path"]) as ItemDefinition
    var item = ItemInstance.new(def) # Assumindo um construtor que aceita a definição
    item.quantity = data.get("quantity", 1)
    item.current_durability = data.get("current_durability", def.max_durability)
    return item
```

#### 5.6. Versionamento de Save (Compatibilidade Futura)

Se você lançar a v1.0 e depois na v1.1 adicionar um novo campo (`mana_regen`) ao player, o save antigo pode falhar ao carregar.

- **Sempre Inclua `version`**: Adicione uma chave `"version"` no dicionário raiz do seu save.
- **Use `.get("campo", valor_padrao)`**: Ao carregar dados, sempre use `get()` com um valor padrão, para que campos ausentes não causem erros.
- **Funções de Migração**: Crie funções que migram dados de saves antigos para a versão atual.

  ```gdscript
  func load_game_with_migration():
      var data = SaveManager.load_game() # Carrega o dicionário bruto

      var save_version = data.get("version", "0.0.0") # Save antigo não tem versão

      # Exemplo de migração
      if save_version == "1.0.0":
          data = _migrate_v1_to_v2(data)
          save_version = "1.1.0" # Atualiza a versão do save migrado

      if save_version == "1.1.0":
          data = _migrate_v1_1_to_v1_2(data)
          save_version = "1.2.0"

      # ... aplicar os dados já migrados ...
      return data
  ```

### Conclusão: A Ferramenta Certa para o Trabalho Certo

- **Para definir "o que é" um item/inimigo/skill (dados de design):** Use `Resource` (`.tres`).
- **Para guardar as configurações do jogador (volume, teclas):** Use `ConfigFile` (`.cfg`).
- **Para falar com o mundo exterior (APIs, web):** Use `JSON`.
- **Para guardar o estado "vivo" e mutável do seu jogo (o que está acontecendo agora):** Use `Dictionary` e `Array` em memória, e serializáveis para o seu arquivo de save.

Dominar esse fluxo é dominar a arquitetura de dados e garantir a persistência e a evolução do seu jogo.

---

## Gerenciamento de Cenas (Scene Flow)

> **Objetivo:** Controlar o fluxo do jogo. Loading Screens, transições suaves e carregamento em background.

---

### 1. O Básico: `change_scene_to_file`

Para protótipos, `get_tree().change_scene_to_file("res://level2.tscn")` funciona.
**Problema:** O jogo trava (congela) enquanto carrega a nova cena. Se for pesada, o jogador acha que crashou.

---

### 2. Carregamento Assíncrono (Background Loading)

A Godot tem a classe `ResourceLoader` que permite carregar em uma thread separada.

#### O `SceneLoader` (Singleton)

Vamos criar um Autoload robusto.

```gdscript
# scene_loader.gd
extends Node

var _loading_path: String = ""
var _loading_status: int = 0
var _progress: Array = []

func load_scene(path: String):
    _loading_path = path
    # Inicia o carregamento em background
    ResourceLoader.load_threaded_request(path)

    # Troca para uma cena de "Loading..." temporária
    get_tree().change_scene_to_file("res://ui/loading_screen.tscn")

    # Liga o processamento para checar o progresso a cada frame
    set_process(true)

func _process(delta):
    if _loading_path == "":
        set_process(false)
        return

    # Consulta o status atual
    _loading_status = ResourceLoader.load_threaded_get_status(_loading_path, _progress)

    # Atualiza a barra de progresso (se existir na cena atual)
    # _progress[0] vai de 0.0 a 1.0
    var loading_screen = get_tree().current_scene
    if loading_screen.has_method("update_bar"):
        loading_screen.update_bar(_progress[0])

    if _loading_status == ResourceLoader.THREAD_LOAD_LOADED:
        # Terminou!
        set_process(false)

        # Pega o recurso carregado
        var new_scene_resource = ResourceLoader.load_threaded_get(_loading_path)

        # Troca a cena manualmente
        get_tree().change_scene_to_packed(new_scene_resource)
        _loading_path = ""
```

---

### 3. Estrutura de Mundos (World vs UI)

Em jogos complexos, você não quer destruir TUDO ao trocar de fase.
Exemplo: O HUD e o Player podem persistir.

**Estrutura da Main.tscn:**

```text
Main (Node)
├── UILayer (CanvasLayer) -> Nunca é destruído
├── MusicLayer (Node) -> Nunca é destruído
└── WorldContainer (Node) -> Aqui trocamos as fases
    └── Level01 (Node3D)
```

#### Script de Troca Local

```gdscript
func change_level(level_packed: PackedScene):
    # Remove fase antiga
    var old_level = $WorldContainer.get_child(0)
    old_level.queue_free()

    # Adiciona nova
    var new_level = level_packed.instantiate()
    $WorldContainer.add_child(new_level)
```

Isso mantém a música tocando sem cortes e o HUD intacto.

---

### 4. Passando Dados entre Cenas

Como dizer para o "Level 2" que o player deve spawnar na "Porta B" e não na "Porta A"?
Não use globais (`Global.next_spawn_point`).

Use um **Payload** ou **Contexto**.

Se estiver usando o `SceneLoader` acima, adicione um parâmetro `params: Dictionary`.
Após instanciar a nova cena, injete os dados.

```gdscript
# No SceneLoader
func _on_load_complete(resource):
    var new_scene = resource.instantiate()

    # Injeção de Dependência
    if new_scene.has_method("setup"):
        new_scene.setup(_saved_params)

    get_tree().root.add_child(new_scene)
    get_tree().current_scene = new_scene
```

**No Level Script:**

```gdscript
func setup(params):
    var spawn_id = params.get("spawn_point", 0)
    var point = get_node("Spawns/Point_" + str(spawn_id))
    $Player.global_position = point.global_position
```
