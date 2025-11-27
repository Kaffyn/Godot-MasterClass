# Machi Game Style: The Engineering Manifesto

> **De:** Machi - Tech Leader da Kaffyn
> **Para:** Desenvolvedores Godot que querem evoluir
>
> Este documento não é um tutorial. É um padrão de arquitetura.
>
> Ele foi forjado nas trincheiras do desenvolvimento de _Lucy & Nero_ e da _SoftEngine_ para garantir escalabilidade, modularidade e performance.
>
> Se você está cansado de "spaghetti code", de tutoriais que ensinam práticas ruins e quer construir sistemas profissionais, bem-vindo ao **Machi Game Style**.

- [Machi Game Style: The Engineering Manifesto](#machi-game-style-the-engineering-manifesto)
  - [0. Índice do MBA (Masterclass)](#0-índice-do-mba-masterclass)
  - [1. Tabela de Decisão Arquitetural](#1-tabela-de-decisão-arquitetural)
  - [2. Detalhamento Técnico](#2-detalhamento-técnico)
    - [A. Scripts vs. Classes (`.gd`)](#a-scripts-vs-classes-gd)
    - [B. Global Class (`class_name`)](#b-global-class-class_name)
    - [C. Resources (`extends Resource`)](#c-resources-extends-resource)
    - [D. O Padrão Singleton (Manual)](#d-o-padrão-singleton-manual)
  - [3. O Poder das Anotações (`@`)](#3-o-poder-das-anotações-)
    - [Classes Abstratas (Godot 4.5+)](#classes-abstratas-godot-45)
  - [4. Ciclo de Vida e Funções Especiais](#4-ciclo-de-vida-e-funções-especiais)
    - [Ciclo de Inicialização e Destruição](#ciclo-de-inicialização-e-destruição)
    - [Loops de Processamento (Game Loop)](#loops-de-processamento-game-loop)
    - [Entrada de Dados (Input)](#entrada-de-dados-input)
    - [Controle de Fluxo e Memória](#controle-de-fluxo-e-memória)
  - [5. Tipagem Estrita (Static Typing)](#5-tipagem-estrita-static-typing)
    - [Regras de Ouro](#regras-de-ouro)
  - [6. Padrões de Comunicação](#6-padrões-de-comunicação)
  - [7. Exemplo Prático de Organização](#7-exemplo-prático-de-organização)
  - [8. Estrutura de Pastas (Feature-based)](#8-estrutura-de-pastas-feature-based)
  - [9. Performance Essencial](#9-performance-essencial)
  - [10. Estruturas de Dados: Resources vs Dictionaries](#10-estruturas-de-dados-resources-vs-dictionaries)
    - [Padrões Avançados de Resources](#padrões-avançados-de-resources)
  - [11. UI e Theming](#11-ui-e-theming)
  - [12. Arquitetura de Áudio](#12-arquitetura-de-áudio)
  - [13. Internacionalização (i18n)](#13-internacionalização-i18n)
  - [14. Blueprints de Sistemas (Arquitetura de Referência)](#14-blueprints-de-sistemas-arquitetura-de-referência)
    - [Itens e Inventário](#itens-e-inventário)
    - [Efeitos e Habilidades (Buffs/Debuffs)](#efeitos-e-habilidades-buffsdebuffs)
    - [Save System](#save-system)
    - [Quests e Missões](#quests-e-missões)
    - [Scene Control](#scene-control)
    - [State Machines (Máquinas de Estado)](#state-machines-máquinas-de-estado)
  - [15. Extensibilidade: Plugins e GDExtensions](#15-extensibilidade-plugins-e-gdextensions)
    - [O Poder do `EditorPlugin`](#o-poder-do-editorplugin)
    - [Criando Painéis (Custom Docks)](#criando-painéis-custom-docks)
    - [Registrando Nós Customizados (`add_custom_type`)](#registrando-nós-customizados-add_custom_type)
    - [GDExtension](#gdextension)
  - [16. Debugging e Profiling](#16-debugging-e-profiling)
    - [Monitores Customizados](#monitores-customizados)
    - [Visual Profiler](#visual-profiler)
  - [17. Qualidade e Testes (QA)](#17-qualidade-e-testes-qa)
    - [GUT (Godot Unit Test)](#gut-godot-unit-test)
    - [Testabilidade](#testabilidade)
  - [18. Padrões Ouro (Gold Standards)](#18-padrões-ouro-gold-standards)
    - [1. AutoLoads (Singletons)](#1-autoloads-singletons)
    - [2. UI \& HUD (Interface)](#2-ui--hud-interface)
    - [3. Resources (Dados)](#3-resources-dados)
    - [4. Static Functions (Utils)](#4-static-functions-utils)
  - [19. Git e Versionamento](#19-git-e-versionamento)
    - [O que IGNORAR (.gitignore)](#o-que-ignorar-gitignore)
    - [Git LFS (Large File Storage)](#git-lfs-large-file-storage)
  - [20. Nomenclatura e Convenções](#20-nomenclatura-e-convenções)
  - [Apêndice A: Checklist de Inicialização (Dia 1)](#apêndice-a-checklist-de-inicialização-dia-1)

---

## 0. Índice do MBA (Masterclass)

Explore os documentos mestres que detalham cada pilar da nossa arquitetura.

### 🏛️ Fundamentos
- [POO e A Filosofia dos Nós](POO.md): Entenda Nodes, SceneTree e Sinais.
- [GDScript Essentials](GDScript.md): O básico bem feito (Movimento, Combate, IA simples).

### 🏗️ Arquitetura Core
- [Programação Orientada a Resources (ROP)](ROP.md): O coração da arquitetura Kaffyn.
- [Sistemas de Spawn e Fábricas](Spawning.md): Instanciando cenas dinamicamente.
- [Gerenciamento de Cenas](SceneManagement.md): Loading screens e troca de fases.

### 📦 Sistemas de Produção
- [Sistema de Inventário](Inventory.md): Do básico ao estilo RPG/Survival.
- [Sistema de Save/Load](SaveSystem.md): Serialização segura e versionamento.
- [Internacionalização (i18n)](i18n.md): Tradução e localização desde o dia 1.

### 🎨 Audiovisual
- [UI Profissional](UI.md): Containers, Themes e Design Responsivo.
- [Animação & Motion](Animation.md): AnimationPlayer vs Tweens.
- [Áudio Dinâmico](Audio.md): AudioStreamRandomizer e Buses.
- [Shaders & Materiais](Shaders.md): Introdução a VFX.

---

## 1. Tabela de Decisão Arquitetural

Use esta tabela para decidir qual estrutura criar.

| Estrutura        | Sintaxe Principal     | O que é?                                          | Para que serve? (Uso Kaffyn)                                                                                         | Quando NÃO usar                                                                                    |
| :--------------- | :-------------------- | :------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| **Global Class** | `class_name Nome`     | Um **Tipo Global** registrado no editor.          | **Tipagem e Herança.** Para criar bases sólidas (`Enemy`, `Interactable`) que aparecem no menu "Create Node".        | Para scripts únicos de uma cena específica (ex: `Level1Manager`). Evite poluir o namespace global. |
| **Resource**     | `extends Resource`    | Um container de **Dados** serializável (`.tres`). | **Dados Compartilhados.** "Fichas" de RPG, configurações de armas, árvores de diálogo. Carregado uma vez na memória. | Para guardar estado volátil da instância (ex: `current_hp` deve ir no Node, `max_hp` no Resource). |
| **Autoload**     | _(Project Settings)_  | Um **Nó Persistente** (`/root/...`).              | **Sistemas Globais.** Gerenciadores de áudio, troca de cenas, analytics.                                             | Para lógica de gameplay local. Se pertence à fase, não deve ser Autoload.                          |
| **Singleton**    | `static var instance` | **Padrão de Projeto** (via código).               | **Acesso Global Local.** Para Managers que vivem dentro da cena mas precisam ser acessados de qualquer lugar nela.   | Se o objeto precisa persistir entre cenas (use Autoload).                                          |
| **Inner Class**  | `class Nome:`         | Uma classe auxiliar dentro de um script.          | **Structs/Helpers.** Dados complexos temporários restritos a um arquivo.                                             | Se a classe precisa ser usada por outros scripts (extraia para um arquivo próprio).                |

---

## 2. Detalhamento Técnico

### A. Scripts vs. Classes (`.gd`)

No Godot, **todo arquivo `.gd` é implicitamente uma classe**.

- Você não precisa de `class_name` para usar um script.
- Ao anexar `player.gd` a um Node, aquele Node se torna uma instância daquela classe anônima.

### B. Global Class (`class_name`)

Use `class_name` para registrar um **Tipo** no sistema da Godot.

- **Benefício 1 (Editor):** O script aparece na janela "Create New Node" com seu ícone personalizado.
- **Benefício 2 (Tipagem):** Permite checagem de tipo robusta (`if body is Enemy:`) em vez de checagem por string ou grupos.
- **Custo:** O Godot carrega todas as Global Classes na inicialização. O excesso pode aumentar o tempo de boot.

**Regra de Ouro:** Só use `class_name` se você pretende reutilizar aquele script em múltiplos lugares ou precisa de verificação de tipo (`is Type`).

### C. Resources (`extends Resource`)

A ferramenta mais poderosa e subutilizada da Godot. Resources são **Dados**, não Lógica.

- **Memória:** Se 100 inimigos usam o mesmo `zombie_stats.tres`, apenas **uma** cópia desse arquivo existe na memória RAM.
- **Serialização:** Resources são salvos em disco (`.tres`). Isso facilita versionamento e edição por game designers sem tocar em código.
- **Injeção de Dependência:** Em vez de hardcodar valores no script, exporte uma variável Resource. Isso torna o comportamento do Node modular.

```gdscript
# Errado (Hardcoded)
extends Node
var damage = 10
var speed = 50

# Certo (Modular)
extends Node
@export var stats: EnemyStats # Arraste um .tres aqui para mudar o comportamento
```

### D. O Padrão Singleton (Manual)

Diferente do Autoload, este script pode estar na cena e ser destruído. Ele permite acesso global via variável estática.

```gdscript
class_name BattleManager extends Node

static var instance: BattleManager

func _enter_tree():
    if instance:
        queue_free() # Garante unicidade
        return
    instance = self

func _exit_tree():
    if instance == self:
        instance = null
```

---

## 3. O Poder das Anotações (`@`)

O GDScript moderno utiliza anotações para configurar comportamento no editor e compilador.

| Anotação          | Uso Principal                                                   | Exemplo                                                                 |
| :---------------- | :-------------------------------------------------------------- | :---------------------------------------------------------------------- |
| `@tool`           | **Rodar no Editor.** Permite que o script execute sem dar Play. | Ferramentas de Level Design ou visualizar mudanças de UI em tempo real. |
| `@export`         | **Expor ao Inspector.**                                         | `@export_category("Stats")`, `@export_range(0, 100)`                    |
| `@onready`        | **Inicialização Segura.**                                       | `@onready var sprite = $Sprite2D` (Garante que o nó filho já existe).   |
| `@icon`           | **Identidade Visual.**                                          | `@icon("res://icons/enemy.svg")` para diferenciar classes no editor.    |
| `@warning_ignore` | **Controle de Linter.**                                         | `@warning_ignore("unused_parameter")`                                   |
| `@abstract`       | **Classes Bases.**                                              | `@abstract`                                                             |

### Classes Abstratas (Godot 4.5+)

Use `@abstract` para criar contratos rígidos e impedir o uso indevido de classes base.

```gdscript
@abstract
class_name Animal extends Node

# Obriga quem herdar a implementar esta função
@abstract
func make_sound() -> void
```

Isso garante que ninguém coloque um "Animal" genérico na cena, apenas "Dog" ou "Cat".

---

## 4. Ciclo de Vida e Funções Especiais

Entenda a ordem de execução para evitar bugs de "Node not found" e problemas de performance.

### Ciclo de Inicialização e Destruição

| Função          | Quando roda?                          | Uso Kaffyn                                               |
| :-------------- | :------------------------------------ | :------------------------------------------------------- |
| `_init()`       | Ao criar o objeto (`.new()`).         | Configuração interna. O nó **NÃO** está na árvore ainda. |
| `_enter_tree()` | Assim que entra na SceneTree.         | Registro em Managers globais.                            |
| `_ready()`      | Após todos os filhos estarem prontos. | Inicialização de gameplay, acesso a `@onready`.          |
| `_exit_tree()`  | Logo antes de sair da árvore.         | Desregistrar de Managers, salvar dados parciais.         |

### Loops de Processamento (Game Loop)

| Função                    | Frequência                      | Uso Principal                                                      |
| :------------------------ | :------------------------------ | :----------------------------------------------------------------- |
| `_process(delta)`         | A cada frame visual (variável). | Animações manuais, interpolação de UI, input contínuo.             |
| `_physics_process(delta)` | Taxa fixa (padrão 60fps).       | **Toda** lógica de movimento e colisão (CharacterBody, RigidBody). |

### Entrada de Dados (Input)

- **`_input(event)`:** Captura bruta. Use para atalhos globais ou debug.
- **`_unhandled_input(event)`:** Captura o que a UI não consumiu. **Use para gameplay** (pular, atirar) para evitar que o clique no botão "Pause" dispare a arma.

### Controle de Fluxo e Memória

- **`queue_free()`:** Marca o nó para ser deletado no final do frame. Nunca use `free()` direto em Nodes.
- **`call_deferred("func")`:** Agenda a execução para um momento seguro (útil ao manipular física/UI dentro de threads ou loops).
- **`await`:** Pausa a execução até um sinal. Ex: `await get_tree().create_timer(1.0).timeout`.
- **`static func`:** Funções puras que não precisam de instância. Ex: `MathUtils.get_random()`.

---

## 5. Tipagem Estrita (Static Typing)

Na Kaffyn, **tipagem explícita é obrigatória**. Código não tipado é considerado dívida técnica imediata.

### Regras de Ouro

1. **Sempre defina o tipo da variável:**

   ```gdscript
   # Errado
   var score = 0

   # Certo
   var score: int = 0
   ```

2. **Sempre defina tipos de argumentos e retorno:**

   ```gdscript
   func take_damage(amount: int) -> bool:
       return true
   ```

3. **Casting Seguro (Safe Cast):**
   Ao carregar Resources ou interagir com nós genéricos, force o tipo para garantir autocomplete e segurança.

   ```gdscript
   # Loading seguro
   var stats: EnemyStats = load("res://goblin.tres") as EnemyStats

   # Interação física
   func _on_body_entered(body: Node2D) -> void:
       var enemy := body as Enemy # Tenta converter
       if enemy:
           enemy.take_damage(10)
   ```

---

## 6. Padrões de Comunicação

A regra de ouro para evitar "Spaghetti Code": **Call Down, Signal Up**.

- **Pai -> Filho:** O pai já conhece o filho (`$Child`). Chame funções diretamente: `$Weapon.shoot()`.
- **Filho -> Pai:** O filho **NÃO** deve conhecer o pai. Emita um sinal: `signal ammo_depleted`. O pai conecta e reage.
- **Irmão -> Irmão:** Nunca acesse diretamente. Use o Pai como mediador ou um **SignalBus** global.

---

## 7. Exemplo Prático de Organização

Cenário: Sistema de Inimigos para um RPG.

1. **Resource (`EnemyStats.gd`):**
   - Define a estrutura dos dados: `max_hp`, `move_speed`, `attack_range`.
   - _Arquivos gerados:_ `goblin_stats.tres`, `orc_stats.tres`.
2. **Global Class (`class_name Enemy extends CharacterBody2D`):**
   - Define a lógica base: Movimento, receber dano, máquina de estados.
   - _Código:_ `@export var stats: EnemyStats`.
3. **Cena (`Goblin.tscn`):**
   - Raiz tem script `Goblin.gd` (que faz `extends Enemy`).
   - No Inspector, a propriedade `stats` recebe `goblin_stats.tres`.
4. **Singleton (`EnemyManager`):**
   - Gerencia o spawn e contagem de inimigos ativos na cena atual.

---

## 8. Estrutura de Pastas (Feature-based)

Na Kaffyn, organizamos arquivos por **Domínio**, não por Tipo.

**Errado (Tutorial Style):**

- 📁 scripts/
- 📁 scenes/

**Certo (Kaffyn Style):**

- 📁 entities/
  - 📁 enemy/
    - 📄 Enemy.tscn
    - 📄 Enemy.gd
    - 🖼️ goblin.png

---

## 9. Performance Essencial

1. **Object Pooling:** Evite `instantiate()` e `queue_free()` em loops rápidos (tiros, partículas).
2. **Tipagem Estrita:** Aumenta a performance do GDScript.
3. **PhysicsServer:** Para >500 objetos, abandone `CharacterBody2D` e use a API de servidor.

---

## 10. Estruturas de Dados: Resources vs Dictionaries

| Estrutura      | Melhor Uso                                                                        | Exemplo                                                                    |
| :------------- | :-------------------------------------------------------------------------------- | :------------------------------------------------------------------------- |
| **Resource**   | **Dados Estáticos (Design).** Podem conter **Dictionaries** e **Funções Helper**. | Stats de Monstros, Definição de Itens, Árvores de Skill.                   |
| **Dictionary** | **Dados Dinâmicos (Runtime).** Coisas que mudam e precisam ser salvas.            | Save Files, Inventário do Player (se tiver dados variáveis), JSON de APIs. |

### Padrões Avançados de Resources

1. **Nested Resources:**
   Use Resources dentro de Resources para modularidade.

   - Exemplo: `CharacterStats` (Resource) contém um slot para `WeaponData` (Resource), que contém um slot para `ElementalEffect` (Resource).

2. **Lógica em Resources:**
   Resources _podem_ ter funções, mas com limites estritos.
   - **Permitido:** Funções que operam _apenas_ nos dados do próprio Resource (ex: `ItemData.get_display_name()`, `Upgrade.calculate_cost(level)`).
   - **Proibido:** Acessar a `SceneTree`, `Input`, ou outros nós globais. O Resource deve ser agnóstico de onde está sendo usado.
   - **Reatividade (Fluent Interface):** Funções que alteram dados devem retornar o próprio Resource ou o valor alterado para que sistemas possam reagir.
     - _Ex:_ `func upgrade() -> ItemData:` (Retorna `self` após aumentar o nível, permitindo `inventory.update_ui(item.upgrade())`).

---

## 11. UI e Theming

A interface do usuário deve ser consistente e fácil de manter.

- **Themes (Obrigatório):** Nunca ajuste fontes, cores ou bordas diretamente nas propriedades de um `Label` ou `Button`. Crie um `Theme` Resource (`main_theme.tres`) e aplique na raiz da sua UI.
  - _Vantagem:_ Alterar a fonte do jogo inteiro leva 10 segundos.
- **Variações:** Use "Theme Variations" para criar estilos derivados (ex: `HeaderLabel` herda de `Label` mas tem fonte maior).
- **Separation of Concerns:** A lógica da UI (`MainMenu.gd`) não deve saber sobre a cor do botão. Ela apenas conecta o sinal `pressed`.

---

## 12. Arquitetura de Áudio

Evite espalhar `AudioStreamPlayer` por todas as cenas.

- **Audio Buses:** Configure o layout de mixagem no painel "Audio" do Godot.
  - `Master` -> `Music`, `SFX`, `UI`.
  - Isso permite criar menus de volume facilmente.
- **AudioManager (Singleton):** Crie um Autoload para tocar sons globais.
  - `AudioManager.play_sfx("explosion")`
  - Implemente **Pooling** de players de áudio para evitar instanciar nós a cada tiro.

---

## 13. Internacionalização (i18n)

Assim como separamos Dados (Resources) de Lógica (Nodes), separamos **Texto** de **Cenas**.

- **A Regra:** Nunca escreva textos finais no Inspector ou Script. Use **Chaves**.

  - _Errado:_ `Label.text = "Game Over"`
  - _Certo:_ `Label.text = "UI_GAME_OVER"`

- **O Formato (.po):**
  Usamos arquivos Gettext (`.po`). O Godot importa automaticamente e substitui as chaves em tempo de execução baseada no locale do usuário.

- **Benefício:** Seu jogo está pronto para localização desde o primeiro commit, e você não precisa caçar strings espalhadas em 50 cenas diferentes.

---

## 14. Blueprints de Sistemas (Arquitetura de Referência)

Não reinvente a roda. Use estes padrões aprovados para sistemas comuns.

### Itens e Inventário

- **Item:** Crie `class_name ItemData extends Resource`. (Nome, Ícone, Peso).
- **Inventário:** Um `Resource` ou `Node` contendo `var items: Array[ItemData]` ou `Array[Dictionary]` para itens únicos/gerados.

### Efeitos e Habilidades (Buffs/Debuffs)

- **Arquitetura:** Use Composição.
- **Data:** `EffectResource` (Define: Dano, Tipo Elemental, Duração).
- **Runtime:** `StatusComponent` (Node) processa os efeitos a cada frame (dano contínuo) ou ao entrar (instantâneo).

### Save System

- **Formato:** Use `Dictionaries` para estruturar o save state.
- **Persistência:** `FileAccess.store_var()` para binário (rápido) ou JSON (debugável).
- **Regra:** Nunca salve Nodes inteiros (`PackedScene`). Salve apenas os dados necessários para reconstruí-los (posição, hp, path do resource).

### Quests e Missões

- **Quest:** `Resource` contendo título, descrição e condições.
- **Manager:** Singleton que escuta sinais globais (`enemy_killed`, `item_collected`) e atualiza o estado das quests ativas.

### Scene Control

- **Manager:** `SceneLoader` (Singleton).
- **Funcionalidade:** `change_scene_to_file(path)` com loading screen intermediária para carregar assets pesados.

### State Machines (Máquinas de Estado)

Não use `bool is_running`, `bool is_jumping`. Use Estados.

1. **Game Flow (Global):**
   Use para gerenciar o ciclo do jogo.

   - Estados: `Menu`, `Loading`, `Gameplay`, `Paused`.
   - Implementação: Autoload `GameStateMachine`.

2. **Entity State (Local):**
   Use Nodes para lógica complexa (`_physics_process`).

   - Estrutura: Node Pai `StateMachine` com filhos `Idle`, `Walk`, `Attack`.

3. **Resources em States (Kaffyn Style):**
   Não hardcode valores nos estados. Injete Resources.
   - _Cenário:_ Um Boss tem 3 fases.
   - _Solução:_ O script `BossPhaseState` é genérico. Ele exporta `@export var config: BossPhaseResource`.
   - _Resultado:_ Você cria 3 arquivos `.tres` (Fase 1, 2, 3) com HP, velocidade e ataques diferentes, e usa o mesmo script lógico.

---

## 15. Extensibilidade: Plugins e GDExtensions

Ferramentas customizadas aceleram a produção. Não tenha medo de estender o editor.

### O Poder do `EditorPlugin`

Para criar um plugin, crie uma pasta em `addons/nome_do_plugin/` e um script `plugin.gd` herdando de `EditorPlugin`.

**Ciclo de Vida Crítico:**

- `_enter_tree()`: Onde você inicializa sua ferramenta.
- `_exit_tree()`: **OBRIGATÓRIO** limpar tudo o que você criou (remover docks, desfazer gizmos). Se esquecer, o editor vaza memória e trava ao recarregar o plugin.

### Criando Painéis (Custom Docks)

Você pode injetar suas próprias interfaces no editor da Godot.

1. **A Cena:** Crie uma cena normal (`MyTool.tscn`) com raiz em `Control`. Use `VBoxContainer`, `Button`, etc.
2. **O Script:** Adicione lógica (`MyTool.gd`) conectando sinais dos botões. Use `tool` no topo se precisar rodar no editor.
3. **A Integração:**

```gdscript
@tool
extends EditorPlugin

var dock_instance

func _enter_tree() -> void:
    # Carrega e instancia sua cena
    dock_instance = preload("res://addons/my_tool/MyTool.tscn").instantiate()
    # Adiciona em um slot do editor (Ex: Esquerda Superior)
    add_control_to_dock(EditorPlugin.DOCK_SLOT_LEFT_UL, dock_instance)

func _exit_tree() -> void:
    # Limpeza essencial
    remove_control_from_docks(dock_instance)
    dock_instance.free()
```

### Registrando Nós Customizados (`add_custom_type`)

Embora `class_name` seja suficiente para a maioria dos casos internos, plugins podem registrar nós explicitamente para distribuição ou organização.

```gdscript
func _enter_tree():
    # Nome, Nó Pai, Script, Ícone
    add_custom_type("MySuperNode", "Node2D", preload("my_super_node.gd"), preload("icon.svg"))

func _exit_tree():
    # Limpeza obrigatória
    remove_custom_type("MySuperNode")
```

### GDExtension

Use GDExtension para escrever código em C++ (ou Rust) que se comporta como nativo.

- **Quando usar:** Cálculos matemáticos pesados, geração de mesh em tempo real, ou integração com SDKs de terceiros.
- **Vantagem:** Performance de C++ com a facilidade de uso de Nodes/Resources no editor.

---

## 16. Debugging e Profiling

"Se você não pode medir, você não pode melhorar."

### Monitores Customizados

Não adivinhe o que está pesando. Crie monitores para ver no gráfico de performance do editor.

```gdscript
func _ready():
    # Aparece na aba "Monitors" do Debugger
    Performance.add_custom_monitor("Game/Enemies Active", func(): return EnemyManager.active_count)
```

### Visual Profiler

Use a aba **Profiler** e **Visual Profiler** para identificar gargalos.

- **CPU Time:** Se alto, otimize seus scripts (`_process`).
- **GPU Time:** Se alto, reduza draw calls, luzes ou complexidade de shaders.

---

## 17. Qualidade e Testes (QA)

A arquitetura Kaffyn facilita testes.

### GUT (Godot Unit Test)

Recomendamos o uso do addon **GUT** para testes automatizados.

### Testabilidade

- **Resources:** São perfeitos para testes unitários pois não dependem da SceneTree.
  - _Ex:_ Testar a fórmula de evolução de nível de um RPG apenas instanciando o Resource e chamando funções.
- **Nodes:** Use testes de integração para validar se sua State Machine transita corretamente de `Idle` para `Walk`.

---

## 18. Padrões Ouro (Gold Standards)

Referência rápida dos "building blocks" padrão da Kaffyn.

### 1. AutoLoads (Singletons)

Não crie Singletons aleatórios. Use estes canônicos:

- **`Global` (ou `Game`):** O cérebro. State Machine do jogo (Menu/Game), Score, Pause.
- **`Config` (ou `Settings`):** Persistência de preferências (Volume, Resolução, Keybindings). Carrega no boot.
- **`SoundManager`:** Camada acima do `AudioServer`. Toca sons, gerencia buses e pooling.
- **`SceneLoader`:** Gerencia `change_scene`, telas de loading e transições.
- **`SaveSystem`:** Serializa e deserializa o `user://savegame.dat`. Não guarda estado, apenas grava/lê.

> **Nota sobre Comunicação:** Na Kaffyn, **não usamos `SignalBus` genéricos**.
>
> - **Global:** Acesse AutoLoads diretamente (`Global.score += 10`). Eles existem para isso.
> - **Local:** Use detecção de física (`Area2D`) e verificação de tipos (`if body is Enemy`) ou Grupos.

### 2. UI & HUD (Interface)

Organize sua UI em camadas usando **CanvasLayers** com Z-Index definidos:

1. **`WorldLayer` (Z 0):** O jogo em si.
2. **`HUDLayer` (Z 10):** Vida, Munição. Fixo na tela, não segue a câmera.
3. **`MenuLayer` (Z 20):** Pause, Inventário. Bloqueia input do HUD.
4. **`OverlayLayer` (Z 100):** Transições (Fade), Debug Console, Mouse Customizado.

**Regras de Ouro de UI:**

- **Containers:** Proibido posicionar na mão. Use `VBoxContainer`, `HBoxContainer`, `GridContainer`.
- **Safe Area:** Tudo começa dentro de um `MarginContainer`.
- **Componentização:** Uma `HealthBar` deve ser um componente isolado que funciona tanto no HUD (Player) quanto no Mundo (sobre a cabeça do Inimigo).

### 3. Resources (Dados)

- **`ItemData` / `EnemyData`:** Definições de entidades.
- **`GameConfig`:** Resource global com constantes de balanceamento (gravidade, speed base) para fácil ajuste por designers.
- **`Theme`:** `main_theme.tres` é obrigatório na raiz da UI.

### 4. Static Functions (Utils)

Classes utilitárias puras (não herdam de Node).

- **`MathUtils`:** `choose_random_weighted()`, `damp()`.
- **`DebugUtils`:** `draw_sphere()`, `log_error()`.
- **`FormatUtils`:** `format_currency()`, `format_time()`.

---

## 19. Git e Versionamento

Mantenha o repositório limpo. Versionamos **Código** e **Assets Originais**, não artefatos gerados.

### O que IGNORAR (.gitignore)

Na Kaffyn, a regra é estrita. Adicione ao seu `.gitignore`:

- `.godot/` (Cache e imports internos).
- `*.uid` (Identificadores únicos locais).
- `*.import` (Configurações de importação locais).

> **Nota:** Ignorar `*.import` e `*.uid` força que cada desenvolvedor reimporte os assets localmente, evitando conflitos de IDs e caminhos absolutos entre máquinas diferentes (Windows vs Linux).

### Git LFS (Large File Storage)

Arquivos binários não devem poluir o histórico do Git. Use LFS para:

- Imagens: `.png`, `.jpg`, `.tga`, `.psd`
- Áudio: `.wav`, `.ogg`, `.mp3`
- Modelos: `.blend`, `.fbx`, `.gltf`

---

## 20. Nomenclatura e Convenções

Para que o código pareça escrito por uma única pessoa.

| Elemento              | Convenção                    | Exemplo                                   |
| :-------------------- | :--------------------------- | :---------------------------------------- |
| **Arquivos e Pastas** | `snake_case`                 | `enemy_controller.gd`, `main_menu.tscn`   |
| **Classes**           | `PascalCase`                 | `EnemyController`, `ItemData`             |
| **Variáveis**         | `snake_case`                 | `move_speed`, `current_hp`                |
| **Privados**          | `_snake_case`                | `_recalculate_stats()`, `_internal_cache` |
| **Constantes**        | `SCREAMING_SNAKE`            | `MAX_SPEED`, `DEFAULT_GRAVITY`            |
| **Sinais**            | `snake_case` (Verbo Passado) | `died`, `item_collected`, `level_started` |

---

## Apêndice A: Checklist de Inicialização (Dia 1)

Antes de escrever a primeira linha de código:

1. [ ] **Git:** Criar repositório e adicionar `.gitignore` (Godot template).
2. [ ] **Project Settings:**
   - Definir Resolução e Stretch Mode (`canvas_items` para Pixel Art).
   - Configurar **Input Map** (pular, atirar).
   - Nomear **Collision Layers** (Player, Enemy, World, Hitbox).
3. [ ] **Estrutura de Pastas:**
   - `assets/`, `entities/`, `resources/`, `ui/`, `systems/`.
4. [ ] **Style:**
   - Definir `class_name` para as entidades principais.
   - Configurar o `Theme` padrão da UI.