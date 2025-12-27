# Externals: Análise Arquitetural para Engenharia Reversa (Zyris Framework)

Este diretório contém plugins do ecossistema Godot analisados para extração de padrões arquiteturais aplicáveis ao **Zyris Framework**. Cada módulo Zyris possui contrapartes externas que servem como referência para design patterns, implementações e melhores práticas.

**Estrutura:** Cada plugin analisado possui documentação detalhada sobre arquitetura, padrões identificados, aplicações no Zyris e anti-padrões a evitar.

## Plugins Godot

1. **Phantom Camera** - Sistema de câmeras virtuais (inspirado no Cinemachine)
2. **LimboAI** - Behavior Trees e Hierarchical State Machines em C++
3. **Dialogic** - Sistema modular de diálogos e narrativa
4. **Gloot** - Sistema de inventário e gestão de itens
5. **Beehave** - Implementação de Behavior Trees em GDScript

## Unity Technologies

6. **Addressables** - Gestão assíncrona de assets e memória
7. **Cinemachine** - Suite de câmeras dinâmicas e inteligentes
8. **DOTS ECS** - Data-Oriented Technology Stack para performance massiva
9. **GraphView** - Engine de visual scripting e diagramas
10. **Timeline** - Sequenciamento e coreografia de eventos

## Unreal Engine

11. **GAS (Gameplay Ability System)** - Framework profissional para RPG e combate
12. **Behavior Tree** - IA orientada a eventos e consultas espaciais
13. **Common UI** - Interface modular e gestão de focos
14. **Mass Entity** - Simulação de multidões via ECS nativo
15. **Lyra Starter Game** - Orquestração sistêmica e arquitetura de "Experiences"
16. **Blueprint VM** - VM de scripting visual e C++ Reflection
17. **Wwise & FMOD** - Middleware de áudio profissional

---

## Comparação Rápida

| Aspecto             | Phantom Camera   | LimboAI                   | Dialogic               | Gloot                   | Beehave             |
| :------------------ | :--------------- | :------------------------ | :--------------------- | :---------------------- | :------------------ |
| **Linguagem**       | GDScript         | C++ (GDExtension)         | GDScript               | GDScript                | GDScript            |
| **Arquitetura**     | Nodes + Autoload | Resources + C++ Runtime   | Resources + Autoload   | Nodes + JSON Prototypes | Nodes + Blackboard  |
| **Padrão Zyris**    | ❌ Node-based    | ✅ Resource + Server-like | ⚠️ Resource + Autoload | ❌ Node + JSON          | ❌ Node-based BT    |
| **Performance**     | Média            | Alta (C++)                | Baixa (GDScript)       | Baixa (GDScript + JSON) | Baixa (GDScript)    |
| **Aplicação Zyris** | → Osmo           | → Behavior Tree           | → Quests/Mythos        | → Inventory             | → Sonhar (Debug UI) |

---

## Phantom Camera

Plugin de câmeras virtuais que permite múltiplas câmeras competindo por controle da Camera3D/2D real via sistema de prioridades.

### Arquitetura do Phantom Camera

**Componentes:**

- `PhantomCamera3D`/`2D` (Node) - Câmera virtual individual
- `PhantomCameraHost` (Node) - Árbitro local que decide qual câmera está ativa
- `PhantomCameraManager` (Autoload GDScript) - Singleton global

**Follow Modes:**

- GLUED - Cola no alvo
- SIMPLE - Segue com offset e damping
- GROUP - Segue centro de múltiplos alvos
- PATH - Limitado a um Path3D
- FRAMED - Dead zones (só move quando alvo sai da área)
- THIRD_PERSON - SpringArm3D automático

### Padrões Identificados

✅ **Priority System**: Sistema elegante de arbitragem por prioridade  
✅ **Tween Configurations**: Resources reutilizáveis para transições  
✅ **Follow Modes Variados**: Comportamentos plugáveis  
❌ **Node-Based**: Câmeras morrem com a cena (não persistem)  
❌ **Autoload GDScript**: Não é C++ Server verdadeiro

### Aplicação no Osmo (Zyris)

**Adaptar:**

```cpp
// VirtualCamera como Resource (não Node)
class VirtualCamera : public Resource {
    enum FollowMode { FIXED, FOLLOW_TARGET, PATH, FRAMED };
    int priority;
    Transform3D evaluate(float delta);
};

// OsmoServer arbitra globalmente
class OsmoServer : public Object {
    void register_vcam(Ref<VirtualCamera> vcam);
    Ref<VirtualCamera> get_active_vcam(); // Arbitra por prioridade
    void blend_to_vcam(Ref<VirtualCamera> target, float duration);
};
```

**Lições:**

- ✅ Sistema de prioridades para arbitragem
- ✅ Múltiplos modos de comportamento
- ✅ Tween system para transições suaves
- ❌ **Não** usar Nodes para dados de câmera (usar Resources)
- 📋 Integrar com Director (Osmo Track para cutscenes)

---

## LimboAI

Plugin C++ (GDExtension/Module) para Behavior Trees e Hierarchical State Machines com foco em performance e editor visual.

### Arquitetura do LimboAI

**Componentes Core:**

- `BehaviorTree` (Resource) - Grafo da árvore
- `BTPlayer` (Node) - Executor de runtime
- `Blackboard` (RefCounted) - Sistema de memória compartilhada com scopes
- `LimboHSM` (Node) - Event-driven state machine

**Hierarquia de Tasks:**

```
BTTask (abstract Resource)
├── BTAction        # Ações (folhas)
├── BTCondition     # Condições booleanas
├── BTComposite     # Controle de fluxo (Sequence, Selector, Parallel)
└── BTDecorator     # Modificadores (Repeat, Invert, Cooldown)
```

### Padrões Identificados

✅ **BehaviorTree como Resource**: Persiste, compartilhável, editável  
✅ **C++ Runtime**: Performance crítica  
✅ **Blackboard com Scopes**: Hierarquia parent/child para compartilhamento  
✅ **Executor Pattern**: BTPlayer executa, BT define  
✅ **HSM Event-Driven**: States reagem a eventos, não polling  
✅ **Editor Visual**: Main Panel com debugger em tempo real

### Blackboard System (Destaque)

```cpp
class Blackboard : public RefCounted {
    HashMap<StringName, BBVariable> data;
    Ref<Blackboard> parent; // Scoping hierárquico

    Variant get_var(const StringName& name); // Sobe para parent se não achar
    void bind_var_to_property(Object* obj, const StringName& prop);
};
```

**Uso:**

```
GlobalBlackboard (Jogo inteiro)
  ↓ parent
SquadBlackboard (Grupo de inimigos)
  ↓ parent
AgentBlackboard (Inimigo individual)
```

### Aplicação no Behavior Tree (Zyris)

**Já Alinhado:**

- ✅ BehaviorTree como Resource
- ✅ C++ runtime (GDExtension)
- ✅ BTPlayer como Node executor

**Adaptar:**

```cpp
// Blackboard pode integrar com AbilitySystem Context
class AbilityContext {
    HashMap<StringName, Variant> context_vars; // Similar ao Blackboard
    Character* character;
    // ... tags ambientais do Gaia, etc.
};

// BT popula context com dados de Synapse
blackboard->set_var("nearest_enemy", synapse->get_nearest_stimulus());
blackboard->set_var("health_ratio", character->get_health_ratio());
```

**Lições:**

- ✅ Blackboard System completo (implementar no Zyris)
- ✅ HSM como complemento (para bosses com fases)
- 📋 BT consulta Synapse → decide ação → chama AbilitySystem
- 📋 Visual Debugger no Sonhar Domain

---

## Dialogic

Plugin GDScript modular para diálogos ramificados e narrativa, com sistema de subsistemas plugáveis.

### Arquitetura do Dialogic

**Componentes:**

- `DialogicGameHandler` (Autoload) - Orquestrador central
- `DialogicTimeline` (Resource) - Sequência de eventos
- `DialogicEvent` (Resource) - Base abstrata para eventos
- Subsistemas (Modules) - Audio, Character, Choice, Text, Variables, etc.

**Tipos de Eventos:**

```
DialogicEvent (Resource)
├── TextEvent          # Exibe texto de personagem
├── ChoiceEvent        # Apresenta escolhas
├── ConditionEvent     # Ramificação condicional
├── JumpEvent          # Pula para label/timeline
├── SignalEvent        # Emit signal
└── VariableEvent      # Set/get variáveis
```

### Padrões Identificados

✅ **Timeline como Resource**: Narrativa serializável  
✅ **Event-Based Execution**: Sequencial e ramificado  
✅ **Modularidade**: Subsistemas auto-registráveis  
✅ **Character Resource**: Personagens com portraits e dados  
❌ **Autoload GDScript**: Não é C++ Server  
❌ **Subsistemas GDScript**: Performance limitada

### Plugin Architecture (Subsistemas)

```gdscript
# Cada módulo é um Subsystem que se registra no Handler
class DialogicSubsystem extends Node:
    var dialogic: DialogicGameHandler # Injetado

    func save_game_state():
        dialogic.current_state_info[name] = get_state()

    func load_game_state():
        restore_state(dialogic.current_state_info[name])
```

### Aplicação em Quests/Mythos (Zyris)

**Quests (Progressão):**

````cpp
class QuestGraph : public Resource {
    Vector<Ref<QuestNode>> nodes; // Similar a DialogicTimeline
};
```gdscript
# Exemplo de uso no Dialogic 2.0
var dialog = Dialogic.start("timeline_name")
````

```cpp
class QuestNode : public Resource {
    enum Type { OBJECTIVE, CONDITION, REWARD, DIALOGUE };
    virtual void execute(QuestContext* ctx) = 0;
};

class QuestServer : public Object {
    void start_quest(Ref<QuestGraph> quest);
    void advance_quest(const StringName& quest_id);
};
```

**Mythos (Narrativa):**

````cpp
class DialogueTree : public Resource {
    Vector<Ref<DialogueLine>> lines;
};

class DialogueLine : public Resource {
    Ref<DialogueCharacter> speaker;
    String text_key; // Localization (usa .po nativo)
    String audio_clip; // Voice acting
};

class MythosServer : public Object {
```gdscript
# Exemplo de uso no Dialogue Manager
DialogueManager.show_example_dialogue_balloon(resource, title)
````

    String get_localized_text(const StringName& key); // .po support

};

````

**Divisão de Responsabilidades:**

| Sistema    | Papel                                          |
| :--------- | :--------------------------------------------- |
| **Quests** | Progressão (objetivos, condições, recompensas) |
| **Mythos** | Narrativa (diálogos, escolhas, localização)    |

**Comunicação:**

```cpp
// Quest trigger Mythos
QuestServer::advance_quest("talk_to_blacksmith");
MythosServer::start_dialogue("blacksmith_intro");

// Mythos trigger Quest (via signal)
MythosServer::on_dialogue_choice.connect(QuestServer::on_choice_made);
````

**Lições:**

- ✅ QuestGraph/DialogueTree como Resources
- ✅ Event-based execution para narrativa
- ❌ **Não** usar Autoload GDScript (usar Server C++)
- ❌ **Não** replicar subsistemas GDScript (C++ nativo)
- 📋 Suporte nativo a `.po` (gettext) para Mythos

---

## Padrões Comuns (Cross-Plugin)

### 1. Resource-Oriented Configuration ✅

Todos usam Resources para dados reutilizáveis:

- Phantom: `PhantomCameraTween`
- LimboAI: `BehaviorTree`, `BTTask`
- Dialogic: `DialogicTimeline`, `Character`

**Zyris já faz isso corretamente.**

### 2. Executor Pattern (Separation of Concerns) ✅

LimboAI e Dialogic separam **dados (Resource)** de **execução (Node/Server)**:

- `BTPlayer` executa `BehaviorTree`
- `DialogicGameHandler` executa `DialogicTimeline`

**Zyris deve manter:**

```cpp
Resource → Define "O QUÊ"
Node → Executa localmente (se necessário)
Server (C++) → Gerencia globalmente
```

### 3. Priority/Arbitration System 📋

Phantom Camera arbitra entre múltiplas câmeras por prioridade.

**Aplicar em:**

- **Osmo**: Múltiplas VirtualCameras competindo
- **Sounds**: Múltiplos sons (cortar o menos importante)

### 4. Signal-Driven Communication ✅

Todos usam sinais extensivamente para desacoplamento.

**Zyris já usa**, manter.

---

## Server vs Node vs Autoload: Análise Crítica

### Phantom Camera: Node + Autoload ❌

- Nodes na SceneTree (morrem com cena)
- Autoload GDScript (não C++)
- **Para Zyris**: Inadequado (Osmo precisa persistir)

### LimboAI: Resource + Node + C++ ✅

- BehaviorTree é Resource (persiste)
- BTPlayer é Node (executor)
- Runtime C++ (performance)
- **Para Zyris**: **Padrão Ideal**

### Dialogic: Resource + Autoload ⚠️

- Timeline é Resource (persiste)
- Autoload GDScript (não C++)
- **Para Zyris**: Conceito bom, mas trocar Autoload por Server C++

### Recomendação Zyris Confirmada ✅

```cpp
Resource      → Dados de gameplay (persiste)
Node          → Executores locais (opcional)
Server (C++)  → Gerenciamento global (obrigatório)
Autoload      → EVITAR (usar Servers C++)
```

---

## Top 10 Lições para Zyris

### 1. Resources > Nodes para Dados ✅

**Observado**: LimboAI, Dialogic  
**Status**: Zyris já faz

### 2. C++ Servers > GDScript Autoloads ✅

**Observado**: LimboAI usa C++, outros usam Autoload  
**Status**: Zyris já faz (manter)

### 3. Separar Executor de Dados ✅

**Observado**: LimboAI (BTPlayer + BehaviorTree)  
**Status**: Aplicar em todos os pacotes

### 4. Sistema de Blackboard/Context 📋

**Observado**: LimboAI (Blackboard com scopes)  
**Aplicar**: Behavior Tree + Ability System integration

### 5. Arbitragem Centralizada de Prioridades 📋

**Observado**: Phantom Camera (PhantomCameraHost)  
**Aplicar**: Osmo, Sounds

### 6. Event-Based Execution 📋

**Observado**: Dialogic (DialogicEvent), LimboAI (HSM)  
**Aplicar**: Quests (QuestNode), Mythos (DialogueLine)

### 7. Modularidade via Composition ✅

**Observado**: Todos  
**Status**: Zyris já usa (AbilityComponents)

### 8. Visual Debugger Essencial 📋

**Observado**: LimboAI (BT Debugger), Phantom (Viewfinder)  
**Aplicar**: Sonhar Domains (BT, Ability, Osmo)

### 9. Tween/Blend System 📋

**Observado**: Phantom Camera  
**Aplicar**: Osmo (blend VCams), Gaia (transições climáticas)

### 10. Separação Runtime vs Editor ✅

**Observado**: LimboAI  
**Status**: Zyris já faz

---

## Recomendações por Pacote Zyris

### Osmo (Câmeras)

**Do Phantom Camera:**

- ✅ Sistema de prioridades
- ✅ Follow Modes variados
- ✅ Tween configurations
- ❌ Trocar Nodes por Resources

**Implementação:**

```cpp
class VirtualCamera : public Resource {
    int priority;
    FollowMode mode;
};
class OsmoServer : public Object {
    Ref<VirtualCamera> get_active_vcam();
};
```

### Behavior Tree (IA)

**Do LimboAI:**

- ✅ BehaviorTree como Resource
- ✅ Blackboard com scopes
- ✅ HSM para bosses com fases
- ✅ C++ runtime

**Já alinhado com LimboAI.**

### Quests (Progressão)

**Do Dialogic:**

- ✅ QuestGraph como Resource
- ✅ Event-based execution
- ❌ Trocar Autoload por QuestServer (C++)

### Mythos (Diálogos)

**Do Dialogic:**

- ✅ DialogueTree como Resource
- ✅ Character Resource
- ➕ Adicionar suporte nativo a `.po` (gettext)
- ❌ Trocar Autoload por MythosServer (C++)

---

## Anti-Padrões Identificados (Evitar)

### ❌ 1. Autoloads GDScript para Core Systems

**Observado**: Phantom Camera, Dialogic  
**Solução Zyris**: C++ Singletons (`Engine::register_singleton`)

### ❌ 2. Nodes para Dados de Gameplay

```gdscript
# Exemplo de uso no PhantomCamera
var pcam = $PhantomCamera3D
pcam.set_priority(20)
```

**Solução Zyris**: VirtualCamera é Resource

### ❌ 3. Misturar Runtime e Editor

**Observado**: Alguns plugins  
**Solução Zyris**: Separação estrita (`package/` vs `package_editor/`)

### ❌ 4. Dual Build (Module + GDExtension)

**Observado**: LimboAI  
**Solução Zyris**: Apenas GDExtension

---

## Conclusão

A análise dos três plugins **confirma a filosofia arquitetural do Zyris**:

**✅ Padrões Corretos:**

1. Resource-Oriented Design
2. C++ Servers (não Autoloads)
3. Separação dados/execução
4. Performance-first (C++ para core)

**📋 Padrões a Adicionar:**

1. Blackboard System (LimboAI → Behavior Tree)
2. Priority Arbitration (Phantom → Osmo, Sounds)
3. Event-based Execution (Dialogic → Quests, Mythos)
4. Visual Debuggers (Todos → Sonhar Domains)

**❌ Anti-Padrões a Evitar:**

1. Autoloads GDScript
2. Nodes para dados principais
3. Runtime/Editor misturado

**O Zyris está no caminho arquitetural correto.** As análises fornecem padrões concretos para implementação dos pacotes Osmo, Behavior Tree, Quests e Mythos.

---

## Beehave

## Áudio Middleware (Wwise & FMOD)

Sistemas de terceiros que oferecem pipelines de áudio profissional, superiores ao motor de som padrão das engines.

### Arquitetura Middleware

- **Event-Based:** O código chama `SoundServer.play("Explosão")`, o middleware cuida da randomização, camadas e filtros.
- **DSP Graph:** Processamento de sinal digital em tempo real (Pitch, Reverb, Low-pass).
- **Voice Management:** Sistema de prioridades que corta sons irrelevantes para economizar CPU.

### Wwise vs FMOD (Lessons for Zyris)

| Característica | Wwise (Authoring-Centric)         | FMOD (API-Centric)                      |
| :------------- | :-------------------------------- | :-------------------------------------- |
| **Poder**      | Pipeline de design visual massivo | API C++ Core extremamente flexível      |
| **Licença**    | Mais restritiva para indies       | Mais amigável (grátis até certo limite) |
| **Integração** | Plugins nativos pesados           | Light-weight e fácil de embutir         |

### Aplicação no Sounds (Zyris)

- ✅ **SoundCue Resource**: Inspirado nos "Events" do FMOD, contendo múltiplos streams e regras de pitch/volume.
- ✅ **Priority System**: O `SoundServer` C++ deve ter um limitador de vozes nativo.
- ✅ **FMOD Core Lessons**: Seguir o padrão de API limpa para manipulação de canais e efeitos por software.

---

## Gloot

Plugin GDScript para inventário universal no Godot 4.4+. Sistema stack-based com constraints modulares (Grid, Weight, ItemCount), prototypes JSON hierárquicos e UI controls prontos.

### Arquitetura

**Componentes Core:**

- `Inventory` (Node) - Container de itens com constraint system
- `InventoryItem` (RefCounted) - Stack de item baseado em prototype
- `Prototree` (JSON Resource) - Árvore de protótipos de itens
- `Constraints` (Nodes filhos) - GridConstraint, WeightConstraint, ItemCountConstraint
- `ItemSlot` (Node) - Slot individual para equip/hotbar
- `StackManager` - Lógica de merge/split de stacks
- `UI Controls` - CtrlInventory, CtrlInventoryGrid, CtrlItemSlot

**Hierarquia de Classes:**

```
Inventory (Node)
├── GridConstaint (Node filho)
├── WeightConstraint (Node filho)
└── ItemCountConstraint (Node filho)

InventoryItem (RefCounted)
├── Prototype (via Prototree JSON)
└── Properties (Dictionary overrides)
```

### Padrões Identificados

✅ **Prototype Pattern (JSON)**: Items herdam de prototypes hierárquicos  
✅ **Constraint System**: Composição de restrições modulares via Nodes  
✅ **Stack Management**: Merge/split automático de stacks compatíveis  
✅ **Serialização Nativa**: `serialize()`/`deserialize()` para save/load  
✅ **Property Override**: Items podem sobrescrever properties do prototype  
✅ **Signal-Driven**: Sinais para item_added, item_removed, property_changed  
❌ **Node-Based Inventory**: Inventory é Node (morre com cena)  
❌ **GDScript Only**: Sem C++ runtime

### Prototype System (JSON-Based)

**Prototree JSON:**

```javascript
{
  "melee_weapon": {
    "weapon_type": "melee",
    "damage": 1
  },
  "knife": {
    "inherits": "melee_weapon",
    "damage": 10,
    "stack_size": 1,
    "max_stack_size": 1
  },
  "arrows": {
    "damage": 5,
    "stack_size": 20,
    "max_stack_size": 50
  }
}
```

**Property Lookup Chain:**

1. Item `_properties` (overrides)
2. Prototype properties
3. Inherited prototype properties (recursivo)

**⚠️ Problema Arquitetural:**

❌ Usa JSON em vez de Godot Resources nativos (ineficiente)  
❌ Parsing JSON em runtime (overhead desnecessário)  
❌ Sem type safety nativo do Godot  
❌ Prototypes não são Resources reutilizáveis

**Solução Zyris:**

````cpp
// Usar Resources nativos em vez de JSON
class ItemPrototype : public Resource {
    GDCLASS(ItemPrototype, Resource);

    String prototype_id;
```cpp
// Padrão de Identificação de Item no Gloot
struct ItemData {
    String id;
    Dictionary properties;
};
````

### Stack Management

```gdscript
# Auto-merge ao adicionar
inventory.add_item_automerge(item)  # Merge com stacks compatíveis

# Auto-split se não couber
inventory.add_item_autosplit(item)  # Divide se não houver espaço

# Combo: merge + split
inventory.add_item_autosplitmerge(item)

# Split manual
var new_stack = item.split(10)  # Cria novo stack de 10

# Merge manual
item1.merge_into(item2, split=true)  # Merge com split opcional
```

### Cons

traint System

**GridConstraint:**

```gdscript
# Item properties interpretadas:
{
  "size": "Vector2i(2, 2)",  # Tamanho no grid
  "rotated": "true"           # Rotação 90°
}
```

**WeightConstraint:**

```gdscript
{
  "weight": 20,  # Peso unitário
  "stack_size": 10  # Peso total = 20 * 10 = 200
}
```

### Aplicação no Inventory (Zyris)

**Adaptar:**

```cpp
// InventoryServer como Singleton C++ (não Node)
class InventoryServer : public Object {
    void add_item(const StringName& inventory_id, Ref<Item> item);
    void remove_item(const StringName& inventory_id, Ref<Item> item);
    Vector<Ref<Item>> get_items(const StringName& inventory_id);
};

// Item como Resource (não RefCounted)
class Item : public Resource {
    String prototype_id;
    HashMap<StringName, Variant> properties; // Overrides
    int stack_size;
    int max_stack_size;

    Variant get_property(const StringName& name);  // Lookup chain
    void set_property(const StringName& name, const Variant& value);
};

// Prototree em JSON (manter)
class ItemPrototree : public Resource {
    Dictionary prototypes;  // Parsed from JSON
    Ref<ItemPrototype> get_prototype(const String& id);
};
```

**Lições:**

- ✅ Prototype JSON hierarchy (excelente para data-driven)
- ✅ Constraint composition (modular e extensível)
- ✅ Stack management (merge/split automático)
- ✅ Serialization (serialize/deserialize nativos)
- ❌ **Não** usar Node para Inventory (usar Server C++)
- ❌ **Não** usar RefCounted para Item (usar Resource)
- 📋 Implementar constraint system similar (GridConstraint para grid inventories)
- 📋 Property lookup chain (prototype → inherited → overrides)

---

## Beehave

Plugin GDScript para Behavior Trees no Godot 4.x. Sistema node-based onde a BT é composta na SceneTree, com debug view integrado, performance monitors e Blackboard simples.

### Arquitetura

**Componentes Core:**

- `BeehaveTree` (Node) - Executor da árvore, gerencia tick e blackboard
- `Blackboard` (Node) - Memória compartilhada (Dictionary wrapper)
- `BeehaveNode` (Node abstract) - Base para todos os nodes de BT
- `ActionLeaf` (Node) - Ações (folhas)
- `ConditionLeaf` (Node) - Condições booleanas
- `Composites` (Node) - Selector, Sequence, Parallel
- `Decorators` (Node) - Inverter, Repeater, Cooldown, Limiter
- `Debug View` - Bottom panel com visualização real-time
- `Performance Metrics` - Custom monitors do Godot

**Hierarquia de Nodes:**

```
BeehaveTree (Node)
├── Blackboard (Node filho opcional)
└── Sequence (Composite Node)
    ├── ConditionLeaf (Condition)
    ├── ActionLeaf (Action)
    └── Selector (Composite)
        ├── ActionLeaf
        └── ActionLeaf
```

### Padrões Identificados (Beehave)

✅ **Node-Based BT**: Árvore inteira na SceneTree (fácil de visualizar)  
✅ **Debug View Integrado**: Bottom panel com estado real-time  
✅ **Performance Monitoring**: Custom monitors via `Performance.add_custom_monitor()`  
✅ **Process Thread Control**: IDLE, PHYSICS, MANUAL  
✅ **Tick Rate**: Controle de quantos frames entre ticks  
✅ **Actor System**: BT age sobre um `actor` (Node externo)  
⚠️ **Blackboard Simples**: Apenas Dictionary wrapper (sem scoping hierárquico como LimboAI)  
❌ **GDScript Only**: Sem C++ runtime  
❌ **BT não é Resource**: Morre com cena, não é reutilizável

### Bla

ckboard System (Limitado)

````gdscript
# Blackboard.gd (simplificado)
class_name Blackboard extends Node

var _data: Dictionary = {}  # Dados por scope

func set_value(key: Variant, value: Variant, blackboard_name: String = "default"):
    if not _data.has(blackboard_name):
        _data[blackboard_name] = {}
    _data[blackboard_name][key] = value
```gdscript
# Exemplo de uso no Beehave
var blackboard = $BeehaveTree.blackboard
blackboard.set_value("is_angry", true)
````

```gdscript
func get_value(key: Variant, default = null, blackboard_name: String = "default"):
    if has_value(key, blackboard_name):
        return _data[blackboard_name].get(key, default)
    return default
```

**Diferença vs LimboAI:**

| Aspecto     | Beehave                        | LimboAI                   |
| :---------- | :----------------------------- | :------------------------ |
| Scoping     | Named scopes (Dictionary keys) | Hierarchical parent/child |
| Binding     | Não suporta                    | `bind_var_to_property()`  |
| Type Safety | Não                            | `BBParam<T>` tipado       |

### BeehaveTree Executor

```gdscript
class_name BeehaveTree extends Node

enum { SUCCESS, FAILURE, RUNNING }
enum ProcessThread { IDLE, PHYSICS, MANUAL }

@export var enabled: bool = true
@export var tick_rate: int = 1  # Ticks por frame
@export var actor_node_path: NodePath
@export var process_thread: ProcessThread = ProcessThread.PHYSICS
@export var blackboard: Blackboard  # Opcional (cria interno se null)
@export var custom_monitor: bool = false  # Performance tracking

var actor: Node

func tick() -> int:
    var child = get_child(0)  # Primeiro filho é root da BT

    if status != RUNNING:
        child.before_run(actor, blackboard)

    status = child.tick(actor, blackboard)

    if status != RUNNING:
        child.after_run(actor, blackboard)

    return status
```

### Debug View (Destaque)

**Features:**

- Visualização em tempo real da árvore executando
- Nodes ativos brilham durante execução
- Inspeção de Blackboard values
- Integrado ao Bottom Panel do Godot

**Implementação:**

```gdscript
# BeehaveTree registra-se no debugger global
func _ready():
    BeehaveDebuggerMessages.register_tree(_get_debugger_data(self))

func _get_debugger_data(node: Node) -> Dictionary:
    return {
        path = node.get_path(),
        name = node.name,
        type = node.get_class_name(),
        id = str(node.get_instance_id()),
        children = [...]  # Recursivo
    }
```

### Comparação com LimboAI

| Aspecto            | Beehave                | LimboAI                  |
| :----------------- | :--------------------- | :----------------------- |
| **Arquitetura**    | Node-based             | Resource + Node Executor |
| **Performance**    | GDScript               | C++ Runtime              |
| **BT Persistence** | Não (morre com cena)   | Sim (BT é Resource)      |
| **Blackboard**     | Dictionary simples     | Hierarchical com binding |
| **Debug**          | Bottom panel integrado | Main panel + debugger    |
| **Editor**         | Scene tree manual      | Visual graph editor      |
| **HSM**            | Não                    | Sim (LimboHSM)           |

### Aplicação no Sonhar (Zyris)

**Do Beehave:**

✅ Bottom Panel Debug View (visualização real-time)  
✅ Performance Metrics Integration (custom monitors)  
✅ Actor-based execution (BT age sobre entity)  
✅ Simple UX (arrastar nodes na scene)

**Do LimboAI (já analisado):**

✅ Main Panel Visual Editor (arrastar tasks visualmente)  
✅ BT como Resource (reutilizável)  
✅ C++ Runtime (performance)  
✅ Blackboard robusto

**Recomendação para Sonhar:**

```
Sonhar BT Domain = LimboAI (editor visual) + Beehave (debug UI)
- Editor visual de BT (LimboAI style) no Main Panel
- Debug view real-time (Beehave style) no Bottom Panel
- BT como Resource (Zyris pattern)
- C++ runtime (Zyris pattern)
- Blackboard hierárquico (LimboAI pattern)
```

**Implementação:**

```cpp
// Sonhar BT Domain (Main Panel)
class SonharBTDomain : public EditorPlugin {
    VirtualBTEditor* editor;  // Graph editor visual

    void _make_visible(bool visible) override;
    void _edit(Object* object) override;  // Edit BehaviorTree Resources
};

// Sonhar BT Debugger (Bottom Panel)
class SonharBTDebugger : public EditorPlugin {
    BTreeDebugView* debug_view;

    void _process(float delta) override;  // Update tree state real-time
    void highlight_active_nodes();
    void show_blackboard_values();
};
```

**Lições:**

- ✅ Debug view real-time é **essencial** para IA (copiar do Beehave)
- ✅ Performance monitoring via custom monitors (implementar no Zyris)
- ✅ Actor-based execution (BT age sobre Character/enemy)
- ❌ **Não** usar Nodes para BT structure (usar Resources como LimboAI)
- ❌ **Não** usar Blackboard simples (usar hierarchical do LimboAI)
- 📋 Integrar debug view do Beehave com```gdscript

# Exemplo de uso no LimboAI

var bt_player = $BTPlayer
bt_player.bt_instance.set_variable("health", 100)

````
---

## Phantom Camera

## Unity 2D Weather System & Gaia Architecture

A análise de sistemas de clima como "Weather System 2D" e "Enviro" no Unity revela padrões de manipulação de atmosfera baseados em **Perfis Globais** e **Shaders Parametrizados**. O foco no 2D é o uso de iluminação global e tinting para mudar o humor da cena sem alterar assets individuais.

### Componentes de Atmosfera

**1. Day/Night Cycle (Time-based):**

- Uso de **Color Gradients** para definir a cor da luz global (`Global Light 2D`) ao longo de 24 horas.
- Interpolação de valores entre estados (Dawn, Day, Sunset, Night).

**2. Global Shader Uniforms (The "Wetness" Pattern):**

- Ao chover, o sistema não altera cada material. Ele atualiza uma variável global no shader: `_GlobalWetness`.
- Shaders de terreno e personagens leem esse valor para aumentar o `Smoothness` (brilho) e escurecer a `Albedo` (cor), simulando umidade.
- **Puddles**: Uso de máscaras procedurais ou `RenderTextures` para desenhar poças de água dinâmicas.

**3. Particle Management:**

- Emissão de partículas de chuva/neve seguindo a câmera do jogador, mas com cálculo de colisão para evitar chuva "dentro de casas" (Raycast top-down).

### Padrões Identificados

✅ **Weather Profiles (Resources)**: Cada clima é um asset ScriptableObject contendo: Cor de Tint, Densidade de Neblina, Prefab de Partícula e Audio Loop.
✅ **Context Injection**: O clima injeta variáveis no sistema de jogo (ex: `Wetness = 1.0` afeta a fricção do player).
✅ **Canvas Modulation**: Em 2D, o uso de um `CanvasModulate` global é o método mais performático para simular névoa ou escuridão total.

### Mapeamento para Gaia (Zyris)

O **GaiaServer** será o coração atmosférico, gerenciando o `WorldEnvironment` e as variáveis globais de shader do Zyris.

**Mapeamento C++:**

```cpp
// GaiaServer (C++ Singleton)
class GaiaServer : public Object {
    GDCLASS(GaiaServer, Object);

    // Atualiza Uniforms Globais no RenderingServer
    void set_global_wetness(float p_value);
    void set_wind_direction(Vector2 p_dir);

    // Injeta Tags de Contexto para Gameplay
    void update_atmosphere_tags();
};

// GaiaProfile (Resource)
class GaiaProfile : public Resource {
    Color sky_tint;
    float rain_intensity;
    Ref<AudioStream> ambience_loop;
};
````

### Lições para o Zyris

- ✅ **Uniform-First Rendering**: O framework Zyris deve garantir que todos os seus shaders (Sprites, Ground, Water) suportem os mesmos nomes de uniformes globais (`u_gaia_wetness`, `u_gaia_wind`).
- ✅ **Atmosphere Tags**: O Gaia deve emitir sinais como `on_storm_started` para que a IA (Synapse) mude seu comportamento (ex: buscar abrigo).
- ✅ **Smooth Interpolation**: Usar o servidor central para garantir transições suaves (Tweening) entre climas, evitando saltos bruscos de cor.
- 📋 **Coverage Maps**: Para o futuro, implementar um "Coverage Map" (RenderTexture 1D ou 2D) que define onde está "coberto" para evitar efeitos de chuva em interiores automaticamente.

---

## Unity Addressables

O sistema definitivo de gerenciamento de ativos do Unity. Ele abstrai a localização física do arquivo (local ou remota) através de um "Endereço" (string). Resolve o problema clássico de dependências e permite carregamento dinâmico e otimizado de conteúdo (DLCs, streaming).

### Arquitetura

**Componentes Core:**

- **Addressable Asset**: Qualquer recurso marcado com uma string de endereço única.
- **Groups**: Agrupamentos que definem como os assets serão empacotados (AssetBundles).
- **Catalogs**: Mapas JSON/Hash que traduzem endereços em caminhos reais em runtime.
- **AsyncOperationHandle**: Objeto de controle para operações assíncronas (carregamento, instanciação).
- **Profiles**: Configurações de caminhos (Build & Load Paths) para diferentes ambientes (Dev, Prod, CDN).

### Padrões Identificados

✅ **Location Independence**: O código pede por `Addressables.Load("hero_model")` e não se importa se o arquivo está no executável ou em um servidor na nuvem.  
✅ **Reference Counting**: Gerenciamento de memória rigoroso. O sistema conta quantos "donos" um recurso tem e só o descarrega quando a contagem chega a zero.  
✅ **Deterministic Dependency Loading**: Carrega automaticamente todas as dependências (materiais, texturas) recursivamente.  
✅ **Incremental Updates**: Permite atualizar apenas partes específicas do jogo através de catálogos remotos.  
✅ **AssetReferences**: Permite que designers arrastem assets no inspetor sem criar referências fortes que forçam o carregamento imediato (Lazy Loading).

### Reference Counting (Destaque)

O sistema de **Reference Counting** é o coração do Addressables. Todo `LoadAssetAsync` deve ter um `Release` correspondente. Se você esquecer de liberar, o asset ficará na memória para sempre (VRAM/RAM Leak). Se liberar cedo demais, os objetos na cena perderão suas texturas/meshes.

### Aplicação no Yggdrasil (Zyris)

O Yggdrasil (Espaço & Tempo) deve usar esse padrão para o carregamento de cenas e chunks de mundo.

**Mapeamento C++:**

```cpp
// YggdrasilServer (Singleton C++)
class YggdrasilServer : public Object {
    GDCLASS(YggdrasilServer, Object);

    // Gerencia o catálogo de endereços
    Dictionary address_map;

    Ref<YggdrasilHandle> load_resource(String address);
    void release_resource(Ref<YggdrasilHandle> handle);
};

// YggdrasilHandle (RefCounted C++)
class YggdrasilHandle : public RefCounted {
    String address;
    Variant result;
    bool is_done;
    float progress;

    // Sinais de conclusão
    void on_completed();
};

// YggdrasilCatalog (Resource)
class YggdrasilCatalog : public Resource {
    String version;
    Dictionary entries; // "address": "res://path/to/file.pck"
};
```

### Lições

- ✅ **Abstração de "Endereço"**: Fundamental para permitir que o Zyris suporte DLCs e Mods nativamente.
- ✅ **Reference Counting Manual**: O Zyris deve expor um sistema de `Release()` claro para o usuário, similar ao Unity, para controle preciso de VRAM.
- ✅ **Async First**: Todas as APIs do Yggdrasil devem ser assíncronas por padrão, evitando "freezes" durante o carregamento de novas áreas.
- ✅ **Group-based Packaging**: Usar o sistema de Exportação do Godot (.pck secundários) como análogo aos AssetBundles, mas orquestrados pelo Yggdrasil.

---

## Unity Cinemachine

O padrão ouro para sistemas de câmeras em motores de jogo. O Cinemachine separa a inteligência da câmera (onde ela deve estar e para onde apontar) da implementação física (a Main Camera). Ele usa um sistema de "operadores virtuais" (Virtual Cameras) que são orquestrados por um "cérebro" central.

### Arquitetura

**Componentes Core:**

- **CinemachineBrain**: Componente na Main Camera. Faz a arbitragem de qual câmera é a "Live" baseada em prioridade e gerencia o blending (tweening) entre elas.
- **CinemachineVirtualCamera (vcam)**: Objeto leve que define as propriedades da câmera (Lens, Body, Aim, Noise). Não renderiza nada.
- **Body (Follow)**: Algoritmo de posicionamento (ex: Transposer, Framing Transposer, Tracked Dolly).
- **Aim (LookAt)**: Algoritmo de orientação (ex: Composer, Group Composer, Hard Look At).
- **Noise (Shake)**: Sistema de trepidação procedural (Basic Multi Channel Perlin).
- **Priority System**: Cada vcam tem um valor inteiro de prioridade. O Brain seleciona a de maior valor.

### Padrões Identificados

✅ **Separation of Concerns**: Virtual Cameras (dados/intenção) vs Main Camera (render/física).  
✅ **Priority-based Arbitration**: Sistema determinístico para troca de câmeras.  
✅ **Modular Pipeline**: O comportamento da câmera é composto por Body + Aim + Noise.  
✅ **Procedural Noise**: Uso de Perlin Noise para movimentos orgânicos sem keyframes.  
✅ **State-Driven Animation**: Integração com máquinas de estado (Animator) para trocar câmeras.  
✅ **Lazy Follow (Damping)**: Suavização independente por eixo (X, Y, Z).

### Noise System (Trepidação)

O sistema de Noise do Cinemachine é aplicado **depois** de todos os outros cálculos, garantindo que o shake não quebre o rastreio (Follow/Aim).

- **Noise Profile**: Resource que define curvas de Perlin Noise com múltiplas frequências e amplitudes.
- **Amplitude Gain**: Intensidade do shake.
- **Frequency Gain**: Velocidade do shake.

### Aplicação no Osmo (Zyris)

O módulo Osmo deve ser uma implementação C++ "Cinemachine-like" para Godot, focada em performance nativa.

**Mapeamento C++:**

```cpp
// OsmoBrain (Server C++)
class OsmoBrain : public Object {
    GDCLASS(OsmoBrain, Object);

    Ref<OsmoVirtualCamera> active_vcam;
    Vector<Ref<OsmoVirtualCamera>> vcam_stack; // Ordenada por prioridade

    void process(double delta); // Realiza o blending na Main Camera
    void register_vcam(Ref<OsmoVirtualCamera> vcam);
};

// OsmoVirtualCamera (Resource)
class OsmoVirtualCamera : public Resource {
    int priority;
    Transform3D target_transform;

    // Componentes plugáveis (Strategy Pattern)
    Ref<OsmoBody> body;
    Ref<OsmoAim> aim;
    Ref<OsmoNoise> noise;
};

// OsmoNoise (Resource)
class OsmoNoise : public Resource {
    float amplitude;
    float frequency;
    Ref<Curve> noise_profile; // Perlin 1D/2D
};
```

### Lições

- ✅ **Câmeras como Dados**: Tratar Virtual Cameras como Resources/Objects leves, não Nodes pesados.
- ✅ **Server-First**: O `OsmoBrain` deve ser um Server/Singleton que manipula o `Viewport` atual.
- ✅ **Pipeline Modular**: Permitir que o usuário troque o "Body" ou "Aim" dinamicamente (ex: passar de Third Person para Orbital).
- ✅ **Damping**: Implementar suavização (Damping) individual para X, Y e Z no posicionamento.
- 📋 **State-Driven**: Integrar com o pacote Behavior Tree do Zyris para mudar câmeras baseada em estados de IA ou Game State.

---

## Unity DOTS Ability System (ECS Analysis)

O Unity DOTS (Data-Oriented Technology Stack) representa a fronteira de performance da Unity. Sua arquitetura de habilidades abandona o `MonoBehaviour` em favor do **ECS (Entity Component System)**, onde dados e lógica são totalmente desacoplados para máxima eficiência de cache.

### Arquitetura Data-Oriented

**1. Components (IComponentData):**

- As habilidades são representadas por structs de dados puros (ex: `AbilityCooldown`, `ManaCost`, `DamageEffect`).
- **AbilityTags**: Structs vazias usadas como marcadores (ex: `FireTag`, `MeleeTag`) para que os sistemas filtrem entidades em O(1).

**2. Systems (ISystem):**

- A lógica é dividida em sistemas independentes (ex: `CooldownSystem`, `CastProgressSystem`).
- Cada sistema processa milhares de entidades simultaneamente usando o **Burst Compiler** para gerar código nativo altamente otimizado.

### Comparação: Unity DOTS vs Unreal GAS

| Feature                  | Unity DOTS (ECS)                  | Unreal GAS                             |
| :----------------------- | :-------------------------------- | :------------------------------------- |
| **Arquitetura**          | Pure ECS (Data-First)             | Hybrid Actor (Object-Oriented)         |
| **Performance**          | Extrema (Milhares de entidades)   | Alta (Focada em heróis/NPCs complexos) |
| **Multiplayer**          | Manual (Requer manual Sync)       | Nativa (Prediction & Rollback)         |
| **Curva de Aprendizado** | Muito Alta (Boilerplate complexo) | Média/Alta (Conceitos de Tags/Effects) |

### Padrões de Design Identificados

✅ **Component Atomicity**: Dividir uma habilidade em micro-componentes em vez de uma classe gigante. Isso permite que um "Stun" seja reaproveitado em qualquer habilidade apenas adicionando o componente `StunEffect`.  
✅ **Job System parallelism**: Executar cálculos de área de efeito (AoE) em múltiplas threads sem race conditions.  
✅ **Cooldown Registry**: Usar timers baseados em `Time.elapsedTime` armazenados nas entidades, processados por um único sistema central.

### Mapeamento para AbilitySystem (Zyris)

O Zyris manterá a facilidade de uso do Godot (`Resources`), mas usará C++ Servers para emular a performance do ECS.

**Mapeamento C++:**

```cpp
// AbilityServer (C++ Singleton)
class AbilityServer : public Object {
    GDCLASS(AbilityServer, Object);

    // Processamento de Lote (similar ao System do ECS)
    void process_active_abilities(float delta);
    void apply_effect(Ref<AbilityEffect> effect, Node* target);
};

// AbilityData (Resource - similar ao IComponentData)
class AbilityData : public Resource {
    float cooldown;
    float cast_time;
    TypedArray<String> tags; // "Fire", "AoE", "Projectile"
};
```

### Lições para o Zyris

- ✅ **Tag Hierarchy**: Implementar um sistema de `GameplayTags` (inspirado no Unreal e emulado no DOTS via Tags) para permitir que efeitos interajam entre si (ex: "Água" apaga "Fogo").
- ✅ **Stat Aggregation**: Calcular o custo/dano final apenas no momento do "Cast", agregando modificadores de outros sistemas (Buffs/Passivas).
- ✅ **Async Execution**: Usar `WorkerThreadPool` do Godot no `AbilityServer` para processar lógica de projéteis e detecção de hit em paralelo, especialmente em jogos Bullet Hell.
- 📋 **Zero-Allocation**: Minimizar a criação de novos objetos durante o gameplay, preferindo reaproveitar dicionários e arrays.

---

## Unity GraphView (UI Toolkit)

A API moderna do Unity para criação de editores de grafos (usada no Shader Graph e VFX Graph). Ao contrário do antigo IMGUI, ela utiliza um modelo de **Retained Mode UI**, permitindo layouts complexos com alta performance e estilos via USS (CSS-like).

### Arquitetura

**Componentes Core:**

- **GraphView**: O canvas principal que gerencia zoom, pan e a coleção de elementos.
- **Nodes**: Unidades de lógica/dados. Possuem containers específicos para Título, Input, Output e Extensões.
- **Ports**: Pontos de conexão com tipos definidos (`portType`) e capacidades (`Single` ou `Multi`).
- **Edges**: As conexões visuais e lógicas entre portas.
- **Blackboard**: Painel para definir parâmetros expostos (variáveis) do grafo.
- **Search Window**: Menu flutuante extensível para criação rápida de nós.

### Padrões Identificados

✅ **Decoupling Data/View**: O GraphView foca na visualização. Os dados reais geralmente residem em `ScriptableObjects` serializáveis.  
✅ **GUID-based Persistence**: Como referências diretas de memória quebram ao fechar o editor, o sistema usa GUIDs (strings únicas) para salvar e re-conectar nós no carregamento.  
✅ **Style Sheets (USS)**: Permite que a aparência dos nós seja alterada sem tocar no código C#, facilitando temas e customizações visuais.  
✅ **Search Provider**: Um sistema baseado em interfaces para injetar novos nós no menu de busca de forma modular.

### Blackboard & Graph Variables (Destaque)

O uso de um **Blackboard** centralizado permite que o grafo se comporte como um "Script Visual", onde variáveis podem ser expostas para o Inspetor do Unity sem precisar criar nós de "Get Variable" para tudo.

### Aplicação no Sonhar (Zyris)

O Sonhar deve ser o laboratório visual do Zyris, e o GraphView é a inspiração para como construir editores de Quests, Behavior Trees e Abilities.

**Mapeamento C++:**

```cpp
// SonharGraphNode (C++ Tool/Node)
class SonharGraphNode : public Control {
    GDCLASS(SonharGraphNode, Control);

    // Containers
    VBoxContainer* input_container;
    VBoxContainer* output_container;

    void add_port(Ref<SonharPort> port);
};

// SonharGraphResource (Persistence)
class SonharGraphResource : public Resource {
    TypedArray<SonharNodeData> nodes;
    TypedArray<SonharEdgeData> edges;
};

// SonharNodeData (Data)
struct SonharNodeData : public Resource {
    String guid;
    Vector2 position;
    Dictionary data;
};
```

### Lições

- ✅ **Retained Mode**: O Sonhar deve evitar redesenhar o grafo inteiro a cada frame. Usar a SceneTree do Godot como o sistema de Retained Mode (Nodes).
- ✅ **Type Safety em Ports**: Implementar cores e formas diferentes para tipos de dados (ex: Circular para Execução, Quadrado para Dados) para feedback visual imediato.
- ✅ **GUID Manager**: O Zyris precisa de um utilitário robusto de GUIDs para garantir que saves de grafos não quebrem ao renomear arquivos.
- 📋 **Integrated Blackboard**: Todos os editores de grafo no Sonhar devem compartilhar a mesma interface de Blackboard para consistência.

---

## Unity Input System (New)

Uma reconstrução completa do sistema de input do Unity, substituindo o antigo Input Manager. Ele foca em abstração total (Actions), suporte a múltiplos dispositivos e um workflow baseado em ativos (Assets). É a referência para sistemas modernos de controle em games.

### Arquitetura

**Componentes Core:**

- **Input Action Asset**: O "cérebro" que contém toda a configuração (Action Maps, Actions, Bindings).
- **Action Maps**: Agrupamentos contextuais de ações (ex: "Player", "Menu", "Vehicle"). Apenas um ou poucos estão ativos simultaneamente.
- **Actions**: Representação abstrata do desejo do jogador (ex: "Jump", "Fire").
- **Bindings**: O link físico (ex: "Spacebar", "Gamepad Button South").
- **Processors**: Filtros matemáticos aplicados ao dado bruto (ex: Deadzone, Invert Y, Normalize Vector2).
- **Interactions**: Lógica de detecção de padrões (ex: Hold por 0.5s, Multi-Tap, Press & Release).

### Padrões Identificados

✅ **Input Abstraction**: O código do jogo pergunta por `Jump.performed`, não por `Input.GetKey(Space)`.  
✅ **Contextual Switching**: Ativar/Desativar Action Maps inteiros facilita o gerenciamento de estados (ex: travar movimento durante diálogos).  
✅ **Multi-Device Support**: Suporte nativo a Keyboard, Mouse, Gamepads, Touch e VR com a mesma API.  
✅ **Composite Bindings**: Permite criar vetores 2D a partir de 4 teclas (WASD) ou modificadores (Shift + Click) nativamente.  
✅ **Runtime Rebinding**: API dedicada para que o jogador altere teclas sem reiniciar ou scripts complexos.

### Interactions & Processors (Destaque)

O uso de **Processors** permite que a lógica de "limpeza" do dado (Deadzone de joystick) fique no Asset e não no código. As **Interactions** permitem que a "intenção" (ex: Carga de Ataque) seja processada pelo sistema de input, emitindo sinais apenas quando a condição é atendida.

### Aplicação no Kinesis (Zyris)

O Kinesis deve ser um **C++ Server** que abstrai o `Input` do Godot para fornecer essa flexibilidade de modern engine.

**Mapeamento C++:**

```cpp
// KinesisServer (Singleton C++)
class KinesisServer : public Object {
    GDCLASS(KinesisServer, Object);

    Ref<KinesisActionSet> active_set;

    void load_action_set(Ref<KinesisActionSet> set);
    bool is_action_pressed(StringName action);
};

// KinesisActionSet (Resource)
class KinesisActionSet : public Resource {
    StringName set_name;
    TypedArray<KinesisAction> actions;
};

// KinesisAction (Resource)
class KinesisAction : public Resource {
    StringName action_name;
    enum Type { BUTTON, VALUE, AXIS };

    TypedArray<KinesisProcessor> processors;
    TypedArray<KinesisInteraction> interactions;
};

// KinesisProcessor (Resource)
class KinesisProcessor : public Resource {
    virtual Variant process(Variant input_data);
};
```

### Lições

- ✅ **Contextual Maps**: No Zyris, o Kinesis deve permitir "Stacks" de Action Maps (ex: Menu sobre Gameplay).
- ✅ **Decoupled Bindings**: Bindings devem suportar "Control Schemes" (Keyboard vs Gamepad) para troca dinâmica de ícones na UI.
- ✅ **Native Rebinding**: O `KinesisServer` deve ter persistência via `Mimir` para salvar as preferências de controle do usuário.
- 📋 **Visual Editor**: O `Sonhar` precisa de um editor de Input similar ao do Unity para facilitar a vida do designer.

---

## Unity Inventory Pro (Asset Analysis)

O Unity Inventory Pro é um dos sistemas de inventário mais robustos do Unity, focando em uma arquitetura extensível baseada em banco de dados de itens e separação clara entre Lógica e UI. Ele é o benchmark para o módulo **Inventory** do Zyris.

### Arquitetura Core

**Componentes Principais:**

- **InventoryManager**: Orquestra as coleções e referências de UI.
- **ItemDefinition**: ScriptableObjects que atuam como "Blueprints" para os itens. Suportam herança de atributos (ex: uma Espada de Ferro herda de Espada Base).
- **ItemCollection**: Abstração para grupos de itens (Inv do Player, Baús, Loot).
- **ItemSlot**: O container individual. Gerencia a lógica de stack, split e restrições.
- **Currency System**: Gerenciamento de economia com suporte a conversão automática (ex: 100 Cooper = 1 Silver).

### Padrões Identificados

✅ **Manager-based Registry**: O sistema usa um `ItemDatabase` central. Ao salvar, ele armazena apenas o `Item_ID` e a `Quantity`, resolvendo a referência ao ScriptableObject original no carregamento.  
✅ **Slot Wrapper UI**: O slot de UI (`ItemCollectionSlotUI`) é apenas um wrapper visual para o dado (`InventoryItemBase`). Isso permite trocar a skin do inventário sem reescrever a lógica.  
✅ **Multi-Slot (Grid)**: Suporte nativo para itens que ocupam espaços variados (ex: 1x2, 2x2), similar ao sistema de Diablo.  
✅ **Equipment Handlers**: O sistema de equipamento é desacoplado. Quando um item é colocado no slot "Cabeça", um `EquipmentHandler` é disparado para instanciar o modelo visual no personagem.

### Persistência (Save/Load)

O sistema foca em **Collection Names**. Cada inventário tem um nome único que o Mimir (ou sistema de save equivalente) usa para localizar o arquivo correspondente. A serialização é tipicamente baseada em dicionários de `ID:Amount`.

### Aplicação no Inventory (Zyris)

O Zyris adotará a abordagem Server-First para o Inventário, garantindo que a lógica de "posso adicionar este item?" ocorra sempre no C++.

**Mapeamento C++:**

```cpp
// InventoryServer (C++ Singleton)
class InventoryServer : public Object {
    GDCLASS(InventoryServer, Object);

    // Transações seguras entre inventários
    bool transfer_item(String from_collection, String to_collection, String item_id, int amount);
};

// Item Resource (Data)
class ZyrisItem : public Resource {
    String item_id;
    int max_stack;
    Ref<Texture2D> icon;
    Dictionary attributes; // Stats dinâmicos
};

// InventoryComponent (Scene Integration)
class InventoryComponent : public Node {
    String collection_name;
    int slot_count;
};
```

### Lições

- ✅ **Database Registry**: O `InventoryServer` deve manter um registro de todos os `ZyrisItem` encontrados no projeto para resolver referências em O(1).
- ✅ **Currency Conversion**: Implementar a lógica de conversão de moedas no `InventoryServer` como um utilitário nativo, não como script de UI.
- ✅ **Stat Injection**: Equipar itens deve injetar "Tags de Contexto" e modificar Atributos no `CharacterResource` automaticamente via sinais.
- 📋 **Grid Inventory**: O Zyris começará com inventário de slots simples, mas o C++ deve ser projetado para suportar `Vector2i size` por item para expansão futura para Grid.

---

## Unity Quest System Pro & Quest Machine

A análise de sistemas de quest no Unity (Quest System Pro e Quest Machine) revela uma transição de sistemas lineares (lista de tarefas) para **Grafos Narrativos** (Flow-based). Estes sistemas definem como o jogador progride na história através de condições e eventos.

### Estrutura de Dados (Hierarquia)

1.  **Quest (Resource)**: O container principal. Contém metadados (Título, Descrição, ID).
2.  **Quest Step (Node)**: Uma fase específica da missão (ex: "Explore a Caverna").
3.  **Objective (Task)**: A unidade atômica (ex: "Mate 10 Goblins", "Fale com o Ferreiro").

### Padrões de Design Identificados

✅ **Narrative Graph (Node-based)**: Diferente de uma lista simples, o uso de grafos (como no Quest Machine) permite ramificações, missões que falham se outra for concluída, e convergências de história.  
✅ **Observer Pattern (Event-Driven)**: O sistema de quests não interage diretamente com o combate. Ele "ouve" um evento global `on_entity_died(entity_type)`. Se o tipo for "Goblin" e a quest ativa pedir Goblins, o contador sobe. Isso garante desacoplamento total.  
✅ **State Persistence**: Apenas o "Estado Dinâmico" é salvo (ID da Quest, Step Atual, Contadores de Objetivos). O "Estado Estático" (diálogos, recompensas) permanece no Resource original.  
✅ **Prerequisite Engine**: Sistema de condições complexas (ex: "Só libera se Player Level > 10 E Quest 'X' está Completa E Clima é 'Chuvoso'").

### Mapeamento para Quests (Zyris)

O Zyris usará o **Sonhar** para editar esses grafos narrativos, com o `QuestServer` executando a lógica em C++.

**Mapeamento C++:**

```cpp
// QuestServer (C++ Singleton)
class QuestServer : public Object {
    GDCLASS(QuestServer, Object);

    // Engine de Eventos
    void notify_event(String event_type, Dictionary data);
    void start_quest(Ref<QuestResource> quest);
};

// QuestResource (The Graph)
class QuestResource : public Resource {
    TypedArray<QuestNode> nodes; // Steps e Conexões
    Dictionary global_variables; // Blackboard narrativo
};
```

### Lições para o Zyris

- ✅ **Event-Based Advancement**: O `QuestServer` deve ser um "Listener" passivo de sinais do framework (Synapse, Inventory, Gaia).
- ✅ **Blackboard Integrado**: Usar o sistema de Blackboard (do Módulo BT/Sonhar) para armazenar variáveis narrativas persistentes (ex: `npc_brave_choice = true`).
- ✅ **Visual Graph Editor**: O Sonhar deve oferecer um domínio de "Quest Graph" especializado para facilitar a criação de ramificações.
- 📋 **Fail States**: Implementar suporte nativo a nós de "Falha" no grafo, permitindo que falhar em uma missão abra caminhos exclusivos na história.

---

## Unity Serialization & ScriptableObjects

O sistema de serialização do Unity é a espinha dorsal de como o motor salva cenas, prefabs e dados. Ele opera em nível de C++ internamente, refletindo fields do C# no Editor. O uso de **ScriptableObjects** é o padrão ouro para arquiteturas "Data-Driven" no Unity.

### Arquitetura de Serialização

**Regras de Ouro:**

- Serializa fields `public` ou marcados com `[SerializeField]`.
- Suporta tipos primitivos, enums, built-in Unity (Vector3, Color) e classes marcadas como `[Serializable]`.
- **ISerializationCallbackReceiver**: Interface fundamental para lidar com Dictionaries ou lógica personalizada de pré/pós serialização.

### ScriptableObjects (O Padrão Flyweight)

ScriptableObjects são Assets que existem fora da hierarquia da cena. Eles são usados para:

1. **Configuração de Dados**: Stats de inimigos, definições de itens.
2. **Desacoplamento**: Sistemas de eventos baseados em ScriptableObjects para evitar Singletons.
3. **Eficiência de Memória**: Múltiplos inimigos apontando para o mesmo ScriptableObject em vez de duplicar dados.

> [!WARNING]
> Em builds (runtime), ScriptableObjects são **Read-Only**. Modificações feitas neles não persistem ao fechar o jogo.

### Padrão de Persistência Híbrida

A maioria dos projetos comerciais Unity usa:

- **ScriptableObjects** para dados estáticos (O que o item _é_).
- **JSON / JSON + Encryption** para saves dinâmicos (Quantos itens o player _tem_).

### Mapeamento para Mimir (Zyris)

O Mimir deve evoluir para ser o "Manager de Recursos Serializáveis" do Zyris, aproveitando a eficiência do Godot `Resource` mas com o rigor do C++.

**Mapeamento C++:**

```cpp
// MimirServer (C++ Singleton)
class MimirServer : public Object {
    GDCLASS(MimirServer, Object);

    // Serialização com AES-256 (Diferencial Zyris)
    void save_encrypted(Ref<Resource> data, String password);
    Ref<Resource> load_encrypted(String path, String password);
};

// Implementação do Padrão Callback
class ZyrisSaveable : public Resource {
    virtual void _on_before_serialize(); // Similar ao Unity
    virtual void _on_after_deserialize();
};
```

### Lições para o Zyris

- ✅ **Resource-First**: Manter a filosofia de que tudo é Resource no Godot, similar ao ScriptableObject do Unity, mas com suporte nativo a métodos e sinais.
- ✅ **Encryption by Default**: Ao contrário do Unity (onde criptografia precisa de pacotes externos), o Mimir deve oferecer AES-256 nativo como "Zero-Config".
- ✅ **Callback Registry**: Implementar um sistema de callbacks similar ao `ISerializationCallbackReceiver` para que módulos (como Inventory) possam limpar/validar dados antes do Save.
- 📋 **Versioning**: Adotar um sistema de `data_version` em todos os Saves do Mimir para facilitar migrações (breaking changes), inspirado no `FormerlySerializedAs` do Unity.

---

## Unity Timeline

O motor de sequenciamento e coreografia do Unity. Ele permite orquestrar animações, áudio, câmeras e lógica customizada em uma linha do tempo não-linear. A grande força do Timeline é sua arquitetura desacoplada (Asset vs Instance) e sua extensibilidade via Playables API.

### Arquitetura

**Componentes Core:**

- **TimelineAsset (Resource)**: Arquivo de dados que contém as Tracks e Clips. Não possui referências diretas a objetos da cena.
- **PlayableDirector (Component)**: O "player" na cena. Ele vincula (bind) as Tracks do Asset aos GameObjects reais no mundo.
- **Tracks**: Camadas funcionais (Animation, Activation, Audio, Playable).
- **Clips**: Blocos de dados nas tracks com tempo de início, duração e propriedades.
- **Playables API**: A camada de baixo nível que processa o "PlayableGraph" (uma árvore de nós de processamento).

**Arquitetura de Extensibilidade:**

- **PlayableAsset**: Resource que define os dados do clip (ex: `Color`, `Intensity`).
- **PlayableBehaviour**: Classe de lógica que executa o `ProcessFrame()` a cada tick da timeline.
- **TrackAsset**: Define o tipo de clip aceito e o tipo de objeto que a track pode controlar (Binding).

### Padrões Identificados

✅ **Asset/Instance Separation**: Permite reutilizar a mesma sequência em diferentes personagens/cenas apenas trocando os bindings.  
✅ **PlayableGraph (Node-Based Execution)**: O processamento é uma árvore de mistura, permitindo blending complexo.  
✅ **Stateless Logic**: O `PlayableBehaviour` é leve e processa dados passados pelo Asset, facilitando a performance.  
✅ **Multi-track Blending**: Uso de "Mixers" para suavizar transições entre clips sobrepostos na mesma camada.  
✅ **ExposedReferences**: Sistema para que o Asset (disco) "prometa" resolver um objeto da cena (runtime).

### Playable Pipeline (Destaque)

O segredo da performance e flexibilidade é o pipeline:

1. `PlayableAsset` cria um `PlayableBehaviour`.
2. O `PlayableBehaviour` injeta lógica no `PlayableGraph`.
3. O mixer da track combina os pesos de todos os clips ativos.
4. O resultado final é aplicado ao objeto vinculado.

### Aplicação no Director (Zyris)

O módulo **Director** será o "Sequenciador nativo" do Zyris, integrando todos os outros pacotes (Osmo, Sounds, Ability).

**Mapeamento C++:**

```cpp
// DirectorServer (Singleton C++)
class DirectorServer : public Object {
    GDCLASS(DirectorServer, Object);

    void play_sequence(Ref<DirectorSequence> sequence, Dictionary bindings);
    void pause_sequence(StringName sequence_id);
};

// DirectorSequence (Resource) - Equivalente ao TimelineAsset
class DirectorSequence : public Resource {
    TypedArray<DirectorTrack> tracks;
    float duration;
};

// DirectorTrack (Resource)
class DirectorTrack : public Resource {
    StringName track_name;
    StringName binding_type; // Ex: "OsmoVirtualCamera", "AudioStreamPlayer"
    TypedArray<DirectorClip> clips;

    // Virtual logic
    virtual void process_frame(double time, float weight, Object* target);
};

// DirectorClip (Resource)
class DirectorClip : public Resource {
    float start_time;
    float duration;
    Ref<Resource> data; // Dados específicos (ex: animação, cor, evento)
};
```

### Lições

- ✅ **Binding System**: Fundamental para reusabilidade. O sequence não deve saber _quem_ ele controla, apenas _qual tipo_ de objeto.
- ✅ **Mixer Pattern**: Implementar mixers para tracks de propriedades numéricas (ex: intensidade de luz, FOV da câmera) para permitir cross-fading entre blocos.
- ✅ **Frame-Accuracy**: O processamento deve ser desacoplado do FPS (baseado em tempo acumulado no Server).
- ✅ **Integration Tracks**: Criar tracks nativas para:
  - `OsmoTrack`: Troca de câmeras virtuais e prioridades.
  - `SoundTrack`: Disparo de SoundCues.
  - `ContextTrack`: Injeção de Tags de contexto no Gameplay (ex: "cinematic_active").
- 📋 **Preview Mode**: O `Sonhar` precisará de uma interface de edição visual para esse grafo de tempo.

---

## Unreal Behavior Tree

A Behavior Tree da Unreal é o padrão de indústria para IA modular e orientada a eventos. Diferente de implementações baseadas em polling (como o Beehave), a versão da Unreal é passiva, otimizando performance ao reagir apenas a mudanças no Blackboard ou eventos externos.

### Arquitetura (Tree & Blackboard)

**Componentes:**

- **Blackboard (Memory):** Totalmente desacoplado. Armazena chaves (variáveis) usadas pela árvore.
- **Decorators (Conditions):** Controlam a execução de nós baseados em condições do Blackboard.
- **Services (Tickers):** Executam lógica periódica enquanto uma subárvore está ativa (ex: buscar inimigo próximo).
- **Tasks (Actions):** Folhas que executam o trabalho real (mover, atacar).

### Diferencial Zyris (vs LimboAI/Beehave)

✅ **Event-Driven:** A árvore só reavalia o que mudou (Observer Aborts).
✅ **Services Pattern:** Excelente para injetar dados no Blackboard sem poluir a árvore principal.
✅ **Performance nativa:** C++ core com suporte a reflexão para Blueprints.

---

## Unreal Environment Query System (EQS)

O EQS é o "cérebro tático" da Unreal. Ele permite que a IA faça perguntas complexas ao ambiente (ex: "Onde é o melhor lugar para me esconder com linha de visão para o player?") e receba um mapa de pontuação.

### Arquitetura do EQS (Generators & Tests)

**Pipeline:**

1. **Generator:** Gera pontos candidatos (Grid ao redor do NPC, Atores próximos).
2. **Context:** Define o referencial (NPC, Player, Querier).
3. **Tests:** Filtram e pontuam os pontos (Line of Sight, Distance, Pathfinding).

### Aplicação no Synapse (Zyris)

- **SynapseQueryServer**: O Synapse deve evoluir de sensores passivos para queries ativas (Inspirado no EQS).
- **Context-Aware Sensing**: Usar o `PhysicsServer3D` para disparar raycasts em lote e pontuar alvos ideais para o `AbilitySystem`.

---

---

## Unreal Blueprint

O Blueprint VM é uma linguagem visual compilada em bytecode, executada em uma pilha virtual altamente otimizada com reflexão total do C++.

### Arquitetura VM

- **Reflection Architecture:** Cada variável `UPROPERTY` e função `UFUNCTION` é visível para a VM.
- **Base Class Pattern:** O C++ define o "Heavy Lifting", o Blueprint define os dados e "Glue Logic".

### Lições para o Sonhar (Zyris)

- ✅ **Base C++ / Extension Sonhar**: Manter a regra: Lógica complexa em C++, composição visual no Sonhar.
- ✅ **Reflection Map**: O Sonhar deve escanear as classes GDExtension para expor propriedades automaticamente.

---

---

---

O Unreal GAS é o "padrão ouro" para sistemas de RPG e combate em larga escala. Sua força reside no desacoplamento total entre **Atributos** (estatísticas), **Efeitos** (modificadores) e **Habilidades** (lógica), tudo orquestrado por um sistema de **Tags Hierárquicas**.

### Arquitetura do GAS (ASC e Atributos)

**1. Ability System Component (ASC):**

- O "Cérebro" do sistema. Todo actor que deseja interagir com habilidades deve possuir um ASC. Ele gerencia a ativação, predição e replicação de rede.

**2. Attribute Sets:**

- Armazenam valores numéricos (Health, Mana, Stamina).
- **Base vs Current**: O sistema mantém o valor base (permanente) e o valor atual (com buffs/debuffs aplicados), permitindo cálculos dinâmicos sem perder o valor original.

**3. Gameplay Effects (GE):**

- Ativos baseados em tempo ou instantâneos que alteram atributos ou tags.
- **Stacking**: Suporte nativo para acumular efeitos (ex: veneno que aumenta o dano por stack).

**4. Gameplay Tags:**

- O sistema de controle de fluxo mais importante. Em vez de `if (is_stunned)`, usa-se `Character.HasTag("State.Stun")`.
- **Abstração**: Permite que a IA e a UI consultem o estado do personagem sem conhecer as classes específicas.

### Padrões de Design Identificados

✅ **Data-Driven Effects**: Os efeitos de gameplay são assets, não código. Isso permite que designers alterem o balanceamento do jogo alterando valores em uma tabela, sem recompilar o C++.  
✅ **Prediction \u0026 Rollback**: O GAS resolve conflitos de rede automaticamente, permitindo que o cliente "preveja" o sucesso da habilidade para resposta imediata.  
✅ **Context Handling**: Cada efeito carrega um `GameplayEffectContext` que diz quem causou o dano, com qual arma e qual nível de habilidade.

### Mapeamento para AbilitySystem (Zyris)

O Zyris adotará a terminologia e a estrutura de separação do GAS para garantir escalabilidade profissional.

**Mapeamento C++:**

```cpp
// AbilityServer (Centro de Operações - similar ao ASC)
class AbilityServer : public Object {
    GDCLASS(AbilityServer, Object);

    // Engine de Efeitos
    void apply_gameplay_effect(Ref<ZyrisEffect> effect, Node* target);
    bool has_tag(Node* node, String tag);
};

// AttributeComponent (C++ Node)
class AttributeComponent : public Node {
    float base_health;
    float current_health;
    // Callbacks para UI
    signal_health_changed;
};
```

### Lições para o Zyris (AbilitySystem)

- ✅ **Hierarchical Tags**: O sistema de Contexto do Zyris deve ser obrigatoriamente hierárquico (`Status.Buff.Speed`) para facilitar filtros.
- ✅ **Base/Current Value Logic**: Implementar a lógica de atributos onde modificadores (`ZyrisEffect`) não alteram o valor real, mas injetam um multiplicador/adicional no cálculo de getter.
- ✅ **Execution Calculations**: Permitir que habilidades complexas usem classes C++ de "Cálculo de Execução" para fórmulas de dano que dependem de múltiplos atributos (ex: `Dano = Força_Atacante - Defesa_Alvo`).
- 📋 **Visual Debugger**: Criar uma ferramenta no **Sonhar** que mostre em tempo real quais Tags e Efeitos estão ativos em cada entidade.

---

## Unreal Motion Graphics (UMG) \u0026 Modular UI Patterns

O UMG é o sistema de UI da Unreal Engine. Sua arquitetura é baseada em **Widgets** compostos, onde o layout é gerido por um sistema rigoroso de **Slots**, e a comunicação segue padrões estritos de **Event Dispatchers** para evitar acoplamento.

### Core Architecture

**1. The Slot System (Layout Glue):**

- Diferente do Godot onde o filho define sua posição, na Unreal, o **Container** atribui um "Slot" ao filho.
- **CanvasSlot**, **GridSlot**, **VerticalSlot**: Cada um expõe propriedades diferentes (Anchors, Padding, Row/Column). Esse padrão garante que o filho só tenha propriedades relevantes ao seu container pai.

**2. Named Slots (Placeholders):**

- Permitem criar templates de UI com "buracos" onde outros widgets podem ser injetados. É a base da UI modular na Unreal.

**3. Event Dispatchers (Communication):**

- **Bubbling Pattern**: O widget filho "dispara" um evento (ex: `OnSaveClicked`) e o pai "se inscreve" para ouvir. O filho nunca tem uma referência direta do pai.

**4. Common UI (Input \u0026 Stacks):**

- Gerencia a "Pilha de Widgets" (Stack). Quando um menu abre sobre outro, o Common UI desativa o input do menu inferior e captura o foco automaticamente.

### Padrões de Design Identificados

✅ **Named Slot Templates**: Essencial para ferramentas que aceitam extensões (como o Sonhar).  
✅ **Widget Stacks**: Evita que o usuário clique em botões de janelas "atrás" de um modal.  
✅ **Glyph Swapping**: Troca automática de ícones de controle (Xbox/PS/Teclado) centralizada.

### Mapeamento para Sonhar (Zyris)

O Sonhar deve adotar o conceito de **Named Slots** para permitir que diferentes módulos (Inventory, Quests) injetem sua própria UI nos nós do grafo.

**Mapeamento C++:**

```cpp
// SonharWorkspace (Gerenciador de Stack)
class SonharWorkspace {
    Vector<Control*> widget_stack;

    void push_view(Control* view) {
        // Desativa foco da view anterior (similar ao CommonUI)
        view->set_focus_mode(Control::FOCUS_ALL);
    }
};

// SonharGraphNode (Template com Slots)
class SonharGraphNode {
    // Slot Nomeado para conteúdo customizado
    Control* custom_content_slot;
};
```

### Lições para o Zyris

- **Slot System**: Layout gerenciado pelo container pai.
- **Named Slots**: Slots para injeção de widgets.
- **Event Dispatchers**: Comunicação bottom-up.

---

## Unreal Mass Entity (ECS) \u0026 High-Performance Simulation

O Mass Entity é o framework ECS (Entity Component System) da Unreal Engine 5, focado em simulações de larga escala ( Crowds, Tráfego). Ele resolve o problema de performance de Atores tradicionais movendo dados para structs contíguos em memória.

### Core Architecture

**1. Fragments \u0026 Tags:**

- **Fragments**: Pequenos `UStructs` C++ que contêm apenas dados (ex: `FTransformFragment`, `FVelocityFragment`).
- **Tags**: Marcadores sem dados usados para filtragem (ex: `FTagIsMoving`).

**2. Archetypes \u0026 Chunks:**

- Entidades com o mesmo conjunto de Fragments são agrupadas em **Archetypes**.
- Seus dados são armazenados de forma contígua em **Chunks**, permitindo que a CPU processe milhares de entidades em uma única passagem de cache.

**3. Processors (Sistemas):**

- Classes sem estado (stateless) que executam a lógica.
- Usam **Entity Queries** para buscar apenas os Fragments necessários (ex: "Todos que têm Transform e Velocity").

**4. Traits (O Manual de Configuração):**

- Definidos no Editor, os Traits dizem ao sistema: "Para ter movimento, adicione estes Fragments e habilite este Processor". É a ponte entre o design visual e o ECS em C++.

### Padrões de Design Identificados

✅ **Hybrid Actor-Entity (Mass Agent)**: Permite que um personagem "longe" seja uma entidade Mass barata e se transforme em um Ator completo e interativo quando o jogador se aproxima (LOD dinâmico).  
✅ **Command Buffers**: Garantia de thread-safety. Mudanças na base de dados (adicionar/remover fragments) são agendadas e executadas em pontos seguros do frame, evitando crashes em sistemas multi-threaded.  
✅ **Signal System**: Em vez de pollings constantes, sistemas podem enviar "Sinais" para entidades específicas ouvirem, economizando ciclos de CPU.

### Mapeamento para Synapse/IA (Zyris)

O Zyris utilizará a lógica de **Fragments** e **Processors** em C++ dentro do `SynapseServer` para gerenciar percepção e movimento de multidões sem o overhead de Nodes.

**Mapeamento C++:**

```cpp
// Fragment (Struct leve em C++)
struct PositionFragment {
    Vector3 pos;
};

// SynapseProcessor (Processamento em Batch)
class MovementProcessor : public ZyrisProcessor {
    void execute(float delta) {
        // Query de entidades que precisam mover
        for (auto& entity : query_entities({typeof(PositionFragment)})) {
            // Lógica SIMD-friendly
        }
    }
};
```

### Lições para o Zyris

- ✅ **Data Contiguity**: No `SynapseServer`, armazenar dados de sensores (visão/audição) em arrays contíguos (`std::vector` de structs) em vez de objetos espalhados na memória.
- ✅ **Traits in Sonhar**: Criar uma interface no **Sonhar** onde o usuário adiciona "Traits" a um modelo de NPC para compor suas capacidades sistêmicas.
- ✅ **Mass-to-Node Sync**: Implementar o "Mass Agent" onde o Server C++ dita a posição global e o Node `Sprite` ou `Mesh` apenas lê e interpola (Unidirectional Data Flow).
- 📋 **Batch Signal Dispatcher**: Implementar um sistema de mensagens em lote no Zyris onde eventos globais (ex: "Explosão") sinalizam milhares de agentes de uma vez de forma performante.

---

## Unreal Lyra Starter Game \u0026 Systemic Orchestration

O Lyra é a referência técnica da Epic para arquiteturas modulares em larga escala. Ele demonstra como desacoplar totalmente o gameplay do "core" usando Plugins de Funcionalidade e um sistema robusto de Experiências.

### Core Architecture

**1. The Experience System:**

- Substitui o `GameMode` estático. Uma **Experience** define:
  - Quais plugins carregar (GFPs).
  - Quais componentes injetar nos Atores.
  - As regras de input e HUD.
- É carregado **assincronamente**, permitindo transições suaves entre modos de jogo.

**2. Modular Gameplay Features (GFP):**

- Funcionalidades (ex: "Shooter", "Vehicles") são plugins independentes.
- Usam o `GameFrameworkComponentManager` para injetar comportamentos em atores existentes sem herança.

**3. Tag-Driven Orchestration:**

- O Input (Enhanced Input) não chama funções, ele dispara **Gameplay Tags** (`Input.Action.Jump`).
- O GAS (Abilities) ouve essas tags.
- A UI (Common UI) ouve essas tags para atualizar ícones ou abrir menus.

### Padrões de Design Identificados

✅ **Component Injection Pattern**: Adicionar lógica a um Ator em tempo real via configuração de dados, evitando a "explosão" de classes filhas.  
✅ **Initialization States**: Uma máquina de estados estrita para componentes modulares (`SPAWNED` -> `DATA_READY` -> `COMPONENTES_READY` -> `GAMEPLAY_READY`) que garante sincronia perfeita em rede.  
✅ **Data-Only Game Feature Plugins**: A habilidade de adicionar armas e skins apenas criando arquivos de dados, sem tocar em uma linha de código C++.

### Mapeamento para Yggdrasil (Zyris)

O **Yggdrasil** atuará como o "Experience Manager" do Zyris, orquestrando a transição entre estados de jogo e a injeção de pacotes.

**Mapeamento C++:**

```cpp
// YggdrasilExperience (Resource de Configuração)
class YggdrasilExperience : public Resource {
    Array<String> required_packages; // Ex: ["Combat", "Inventory"]
    Array<ZyrisTrait> game_rules;    // Ex: "NoGravity", "TimeLimit"
};

// YggdrasilServer (Orquestrador)
class YggdrasilServer : public Object {
    void load_experience(Ref<YggdrasilExperience> exp) {
        // 1. Carrega pacotes assincronamente
        // 2. Injeta componentes nos Singletons relevantes
        // 3. Emite sinal EXPERIENCIA_CARREGADA
    }
};
```

### Lições para o Zyris

- ✅ **Asynchronous Initialization**: O Zyris NUNCA deve bloquear a thread principal durante o carregamento de módulos. Seguir o padrão `ExperienceAction` da Unreal para execução step-by-step.
- ✅ **Trait-Based Composition**: Em vez de `PlayerInventoryNode`, usar um `Pawn` base e injetar o `InventoryComponent` via configuração da Experiência.
- ✅ **Global Tag Registry**: O Zyris precisa de um Singleton `TagServer` para validar e centralizar todas as tags hierárquicas, evitando erros de digitação (ex: `State.Dead` vs `state.dead`).
- 📋 **Modular HUD Extension**: Permitir que cada módulo ou "Experiência" registre seus próprios itens de UI no Sonhar/HUD sem precisar de um "MainHUD" centralizado que conheça todos os módulos.

---

## Wwise

## Yarn Spinner
