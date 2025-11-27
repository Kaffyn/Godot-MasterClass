# Módulo 13: Plugins e Modularidade – Estendendo a Engine

## 1. A Filosofia da Modularidade: Por que Plugins são a Solução

Todo projeto de jogo começa simples. Um script de player, um de inimigo. Logo, o script do player tem 800 linhas e controla movimento, inventário, vida e diálogo. Isso é um **monolito**: frágil, difícil de manter e impossível de reutilizar.

O "Godot MBA" ensina a pensar como um arquiteto. Em vez de um bloco de código gigante, construímos um **ecossistema de sistemas independentes** que conversam entre si. Plugins são a materialização dessa filosofia, a ferramenta que nos força a criar fronteiras saudáveis e nos traz vantagens arquiteturais claras:

- **Reusabilidade**: O benefício óbvio. Um sistema de inventário bem feito pode ser usado em seu RPG, no seu jogo de sobrevivência e no seu FPS com pouca ou nenhuma alteração.

- **Separação de Responsabilidades (SoC)**: O benefício real. Criar um plugin força você a pensar: "O que o meu sistema de Áudio _realmente_ precisa saber sobre o Player?". A resposta deveria ser "nada". Ele só precisa receber eventos como `"play_sound('player_footstep')"`. Essa disciplina cria código mais limpo e desacoplado.

- **Manutenção e Escalabilidade**: Um bug no sistema de quests está confinado ao plugin de quests. É imensamente mais fácil de achar e consertar sem quebrar o resto do jogo. Adicionar uma nova feature significa adicionar um novo plugin ou estender um existente, não mexer em um script de 2000 linhas.

- **Colaboração em Equipe**: Permite que times trabalhem em paralelo. O "Time de UI" pode focar no plugin de UI, enquanto o "Time de Gameplay" trabalha no de combate, sem pisar no pé um do outro.

Em suma, deixamos de fazer "um script para o player" e passamos a construir "um sistema de personagem", "um sistema de inventário" e "um sistema de efeitos", onde o Player é apenas o cliente que os utiliza.

## 2. Anatomia de um Plugin: O que há por Dentro

Um plugin é, em sua essência, uma pasta dentro do diretório `addons/` do seu projeto que a Godot reconhece e trata de maneira especial. Para que isso aconteça, duas coisas são essenciais: o arquivo de manifesto `plugin.cfg` e, opcionalmente, um script principal.

### A Estrutura de Arquivos

Uma boa estrutura de plugin é auto-contida:

```
addons/
└── meu_plugin/
    ├── plugin.cfg          # O manifesto, obrigatório.
    ├── plugin_script.gd    # O script principal (se for um EditorPlugin).
    ├── icon.svg            # Ícone para seus tipos customizados.
    ├── cenas/
    │   └── hud_custom.tscn
    └── scripts/
        ├── hud_custom.gd
        └── utils.gd
```

### O Manifesto: `plugin.cfg`

Este arquivo `INI` é o coração do seu addon. Ele diz à Godot que a pasta existe e como deve ser tratada.

```ini
[plugin]

name="Meu Primeiro Plugin"
description="Um plugin de exemplo para o Godot MBA."
author="Seu Nome"
version="1.0"
script="plugin_script.gd"
```

- `name`, `description`, `author`, `version`: Metadados que aparecem na aba "Plugins" em `Project Settings`.
- `script`: O caminho para o script principal que herda de `EditorPlugin`. Este script só é executado **dentro do editor Godot** e é o ponto de entrada para adicionar novas funcionalidades à IDE.

### Tipos de Plugin e Funcionalidades

#### 1. Plugins de Runtime (Comportamento no Jogo)

Estes são os mais comuns. Eles podem não ter um `script` de `EditorPlugin` no `plugin.cfg`, mas fornecem `Autoloads`, `class_name` (tipos customizados) e cenas reutilizáveis para o seu jogo.

- **Autoloads**: A maneira mais fácil de um plugin fornecer um sistema global (um singleton). Em `Project Settings -> Autoload`, você pode adicionar um script ou cena do seu plugin para que ele esteja sempre acessível. O `SaveMachine` da SoftEngine é um exemplo perfeito.
- **Tipos Customizados (`class_name`)**: Seu plugin pode definir novos tipos de nós e recursos. Ao usar `class_name MeuNoCustomizado extends Node` e um `icon.svg` na raiz da pasta, "MeuNoCustomizado" aparecerá na janela "Create New Node" do editor, com seu ícone!

  _Exemplo de um nó customizado em um plugin:_

  ```gdscript
  # addons/meu_plugin/scripts/health_component.gd
  @icon("res://addons/meu_plugin/icon.svg")
  class_name HealthComponent extends Node

  signal health_changed(new_health)
  signal died

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

#### 2. EditorPlugins (Ferramentas para o Editor)

Estes são plugins que modificam o próprio editor Godot, criando ferramentas para você e sua equipe. O script principal deve herdar de `EditorPlugin` e ser referenciado no `plugin.cfg`.

- **O que podem fazer?**

  - Adicionar menus e botões à interface do editor.
  - Criar `docks` (painéis) customizados.
  - Desenhar coisas na viewport 2D/3D (gizmos).
  - Automatizar tarefas.

- **Funções chave no script do `EditorPlugin`**:
  - `_enter_tree()`: Chamado quando o plugin é ativado. É aqui que você adiciona seus botões, docks, etc.
  - `_exit_tree()`: Chamado quando o plugin é desativado. É **crucial** remover tudo que você adicionou para não deixar lixo na interface do editor.

_Exemplo de um `EditorPlugin` simples que adiciona um botão no topo:_

```gdscript
# addons/meu_plugin/plugin_script.gd
@tool # ESSENCIAL! Informa à Godot que este script pode rodar no editor.
extends EditorPlugin

var toolbar_button

func _enter_tree():
    # Cria um botão e o adiciona à barra de ferramentas principal
    toolbar_button = Button.new()
    toolbar_button.text = "Minha Ação"
    toolbar_button.pressed.connect(_on_button_pressed)
    add_control_to_container(CONTAINER_TOOLBAR, toolbar_button)

func _exit_tree():
    # Limpeza é fundamental! Remove o botão que você adicionou.
    if is_instance_valid(toolbar_button):
        remove_control_from_container(CONTAINER_TOOLBAR, toolbar_button)
        toolbar_button.queue_free()

func _on_button_pressed():
    print("O botão do meu plugin foi pressionado!")
```

## 3. Estudo de Caso: A Arquitetura de Domínios da SoftEngine

A SoftEngine, a suíte de plugins da Kaffyn, é a aula magna sobre arquitetura de plugins. Ela não é um plugin, é uma **suíte de plugins**. Vamos dissecar cada domínio para entender sua responsabilidade e ver exemplos práticos de como eles funcionam.

### Core: A Fundação do Jogo

O `softengine_core` resolve problemas que todo jogo tem, para que você não precise. Sua principal responsabilidade é o sistema de **Save/Load**.

- **Responsabilidades**:
  - Fornecer o `SaveMachine`, um `Autoload` global para salvar e carregar o estado do jogo.
  - Definir `SaveSchema` para configurar _o que_ e _como_ salvar (canais, formato, etc.).
  - Oferecer utilitários de baixo nível em GDExtension para performance.
- **Exemplo de Uso**: Salvar o jogo em um checkpoint.
  ```gdscript
  # Em um script de checkpoint, ao ser ativado pelo jogador:
  func _on_player_entered():
      print("Checkpoint ativado! Salvando o progresso...")
      SaveMachine.save(0) # Salva no slot 0
      await SaveMachine.save_completed
      print("Jogo salvo com sucesso!")
  ```

### Machines: O Cérebro Orquestrador

O `softengine_machines` controla o **fluxo e os estados** do jogo. Ele decide "o que está acontecendo agora?".

- **Responsabilidades**:
  - Fornecer o `StateComponent`, um nó para criar Máquinas de Estado Finitas (FSM) para personagens (parado, andando, atacando).
  - Implementar a `GameMachine`, que gerencia o estado global do jogo (MainMenu, InGame, Paused).
- **Exemplo de Uso**: Controlar o estado de um personagem com base no input.

  ```gdscript
  # No script do Player, que tem um StateComponent como filho
  @onready var state_component: StateComponent = $StateComponent

  func _physics_process(_delta):
      var move_input = Input.get_vector("left", "right", "up", "down")
      if move_input.length() > 0:
          state_component.request_state("Run")
      else:
          state_component.request_state("Idle")

      if Input.is_action_just_pressed("attack"):
          state_component.request_state("Attack")
  ```

### Behavior: A Alma dos Personagens

O `softengine_behavior` é onde a **personalidade** e as **regras** dos seus personagens, itens e habilidades são definidas, tudo através de `Resources`.

- **Responsabilidades**:
  - Definir `CharacterProfile` com stats (vida, ataque, defesa).
  - Definir `ItemResource` para itens (poções, espadas) e seus efeitos.
  - Definir `ProgressionProfile` para gerenciar níveis, XP e árvores de habilidade.
- **Exemplo de Uso**: Definir os dados de um inimigo usando um `Resource`.

  ```gdscript
  # Arquivo: res://enemies/goblin_stats.tres (um Resource do tipo CharacterProfile)
  [gd_resource type="CharacterProfile" load_steps=2 format=3]

  [ext_resource type="Script" path="res://addons/softengine_behavior/character_profile.gd" id="1_abcde"]

  [resource]
  script = ExtResource("1_abcde")
  id = "goblin"
  display_name = "Goblin Batedor"
  max_health = 25
  attack_power = 5
  ```

### World: A Estrutura da Realidade

O `softengine_world` define **o que existe, onde e sob quais regras**. Ele descreve o layout das fases, onde os inimigos aparecem, e como as áreas se conectam.

- **Responsabilidades**:
  - Definir `WorldSegment` para cada fase ou área do seu jogo.
  - Definir `SpawnPoint` para controlar onde, quando e como inimigos ou itens aparecem.
  - Gerenciar a conexão entre as fases (portais, portas).
- **Exemplo de Uso**: Definir um ponto de spawn em uma floresta.

  ```gdscript
  # Arquivo: res://worlds/forest/spawns/goblin_spawn_1.tres (um Resource do tipo SpawnPoint)
  [gd_resource type="SpawnPoint" load_steps=3 format=3]

  [ext_resource type="Script" path="res://addons/softengine_world/spawn_point.gd" id="1_vwxyz"]
  [ext_resource type="CharacterProfile" path="res://enemies/goblin_stats.tres" id="2_12345"]

  [resource]
  script = ExtResource("1_vwxyz")
  entity_profile = ExtResource("2_12345") # O que vai nascer aqui
  respawn_cooldown = 30.0 # Em segundos
  ```

### FX: O Feedback Sensorial

O `softengine_fx` cuida de todo o **feedback audiovisual**: sons, efeitos visuais, música, etc. Ele opera com um catálogo de eventos, desacoplando a lógica do asset.

- **Responsabilidades**:
  - Fornecer o `FXAudio`, um `Autoload` para tocar sons por ID lógico.
  - Gerenciar perfis de áudio (BGM de combate, BGM de exploração).
  - Controlar perfis de câmera e efeitos visuais.
- **Exemplo de Uso**: Tocar um som quando o jogador pula.
  ```gdscript
  # No script do Player, na função de pulo
  func jump():
      # ... lógica do pulo aqui ...
      FXAudio.play_event("player_jump") # O jogo não sabe qual .wav, apenas o evento.
  ```

### UI: A Interface com o Jogador

O `softengine_ui` oferece um **Design System** para Godot, com componentes prontos e um sistema de temas.

- **Responsabilidades**:
  - Fornecer componentes reutilizáveis (`SoftButton`, `SoftHUD`, `SoftInventoryPanel`).
  - Gerenciar temas globais (`Theme` Resources) para manter a consistência visual.
  - Implementar binding de dados para conectar a UI aos dados de `Behavior` e `Machines` facilmente.
- **Exemplo de Uso**: Conectar a barra de vida do HUD aos stats do player.

  ```gdscript
  # No script da sua cena principal, onde o Player e o HUD vivem
  @onready var player = $Player
  @onready var hud = $HUD

  func _ready():
      # O HUD usará este caminho de string para encontrar o valor dinamicamente
      hud.bind_property("health_bar.value", player, "stats.health")
      hud.bind_property("health_bar.max_value", player, "stats.max_health")
  ```

Essa separação clara, exemplificada por cada domínio, é o que permite construir jogos complexos que são, ao mesmo tempo, fáceis de manter, escalar e dar manutenção.

## 4. Diretrizes de Ouro para Criar Plugins (The Machi Way)

1.  **Pense em Domínios, não em Features**: Seu plugin não é "a porta que precisa de chave", é o "Sistema de Interações e Travas".
2.  **API Clara, Implementação Oculta**: Outros sistemas só devem interagir com seu plugin através de `Autoloads` bem definidos ou sinais. Eles nunca devem precisar de `get_node()` para fuçar a estrutura interna do seu plugin.
3.  **Use `class_name` e um `icon.svg`**: Dê uma identidade visual e programática aos seus tipos customizados. Facilita o uso no editor e a leitura do código.
4.  **Documente com um `README.md`**: Todo plugin na pasta `addons/` deve ter um `README.md` explicando o que faz, como configurar e qual sua API principal.
5.  **Cuidado com Autoloads em Excesso**: Nem todo plugin precisa de um singleton global. Use-os para gerentes de sistema (Managers), não para dados ou helpers.

## 5. Mão na Massa: Criando um Plugin de Quests

Vamos criar um sistema de quests que se integra à SoftEngine.

### Passo 1: A Arquitetura

- **Domínio**: Gerenciar o ciclo de vida de quests.
- **Resources**:
  - `QuestResource.gd` (`quest_id`, `title`, `objectives: Array[ObjectiveResource]`, `rewards: Array[RewardResource]`).
  - `ObjectiveResource.gd` (base, com subtipos como `KillObjective`, `FetchObjective`).
  - `RewardResource.gd` (base, com subtipos como `XpReward`, `ItemReward`).
- **Runtime**: `QuestManager.gd` (Autoload) para gerenciar quests ativas.
- **API (Sinais)**:
  - O Manager escuta: `GameEvents.enemy_defeated`, `GameEvents.item_collected`.
  - O Manager emite: `quest_started`, `objective_completed`, `quest_completed`.

### Passo 2: A Implementação

1.  Crie a pasta `addons/quest_system/` com `plugin.cfg` e o script que registra o `QuestManager.gd` como Autoload.
2.  Crie os scripts de `Resource` (`QuestResource.gd`, etc.) usando `class_name`.
3.  Implemente a lógica no `QuestManager.gd` para carregar quests, escutar eventos do jogo e atualizar o progresso.

### Passo 3: A Integração

- **Com Machines/Behavior**: A state machine de um inimigo, ao morrer, emite um sinal global `GameEvents.enemy_defeated`. O `QuestManager` escuta esse sinal.
- **Com a UI**: Um painel `QuestLog.tscn` escuta os sinais do `QuestManager` para exibir o status das quests, sem nunca falar com a lógica de combate ou inventário.
- **Com o `SaveMachine`**: O `QuestManager` implementa `export_save_data()` e `import_save_data()`. Adicionamos o canal `"quests"` ao `SaveSchema`. O `SaveMachine` cuida do resto.

## 6. Arquitetura Multiplataforma: Criando um Plugin de Conquistas (Achievements)

Seu jogo vai para a Steam/Xbox/PlayStation. Como lidar com Conquistas/Troféus? Com uma **camada de abstração**.

### Passo 1: A Arquitetura da Abstração

- **O Problema**: `Steam.setAchievement()`, `Xbox.unlockAchievement()`, etc., são APIs diferentes. Não queremos `if/elif` no nosso código de jogo.
- **A Solução**: Um novo plugin, `AchievementSystem`, que define uma **interface**.

- **Resources**:
  - `AchievementResource.gd` (`achievement_id`, `title`, `description`, `platform_id_steam`, `platform_id_psn`).
- **Runtime (O Cérebro)**:
  - `AchievementManager.gd` (Autoload): Expõe um único método: `unlock(achievement_id: String)`.
- **A Interface (O Contrato)**:
  - `AchievementProvider.gd`: Um script base com funções vazias: `initialize()`, `unlock_achievement(platform_id)`.
- **As Implementações Concretas**:
  - `SteamProvider.gd` (estende `AchievementProvider`): Aqui dentro vai a chamada `Steam.setAchievement()`.
  - `LocalProvider.gd` (estende `AchievementProvider`): Nosso fallback. Salva a conquista em um arquivo local via `SaveMachine`.

### Passo 2: A Lógica de Seleção

O `AchievementManager`, ao iniciar, detecta a plataforma e escolhe o provedor certo:

```gdscript
# Em AchievementManager.gd
var provider: AchievementProvider

func _ready():
    if OS.has_feature("steam"):
        self.provider = SteamProvider.new()
    else:
        self.provider = LocalProvider.new() # Para testes e outras plataformas

    provider.initialize()
```

### Passo 3: A Integração

- O `AchievementManager` escuta o sinal `quest_completed` do `QuestManager`.
- Ao receber `quest_completed("matar_o_chefe_final")`, ele chama `unlock("chefe_final_derrotado")`.
- O método `unlock` simplesmente faz `provider.unlock_achievement(...)`.
- O código do jogo **nunca sabe** se está falando com a Steam, com o Xbox ou com um arquivo de save local. A arquitetura o protege dessa complexidade.

Com isso, você não apenas criou um plugin, mas uma solução de software robusta, escalável e portável. Este é o nível do Godot MBA.
