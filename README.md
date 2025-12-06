# Machi Game Style: The Machi Class

> **De:** Machi - Seu Mentor no Machi Class
> **Para:** Futuros Arquitetos de Software de Jogos

Este repositório contém o conhecimento acumulado para transformar desenvolvedores de jogos em Arquitetos de Software.

A estrutura do curso é dividida em dois níveis de profundidade:

---

## 🎓 Graduação: A Base (Aulas Práticas)

Aqui você aprende a "falar Godot". O foco é na prática, no passo-a-passo e na construção dos fundamentos mentais.
Se você está começando ou quer reforçar a base, comece por aqui.

### [👉 ACESSAR O PORTAL DO ALUNO (Índice das Aulas)](./aulas/README.md)

**Conteúdo do Curso:**

- **Módulo 00:** [Fundamentos da Arquitetura](./aulas/README.md#módulo-00-fundamentos-da-arquitetura)
- **Módulo 01:** [A Tríade Arcade (Snake, Pong, Pacman)](./aulas/README.md#módulo-01-a-tríade-arcade-snake-pong-pacman)
- **Módulo 02:** [Arquitetura de Entidades (Topdown Shooter)](./aulas/README.md#módulo-02-arquitetura-de-entidades-topdown-shooter)
- **Módulo 03:** [Sistemas de Dados e UI (RPG Tático)](./aulas/README.md#módulo-03-sistemas-de-dados-e-ui-rpg-tático)
- **Módulo 04:** [Física Avançada e Estados (Metroidvania)](./aulas/README.md#módulo-04-física-avançada-e-estados-metroidvania)
- **Módulo 05:** [Procedural Generation & Tilemaps (Roguelike)](./aulas/README.md#módulo-05-procedural-generation--tilemaps-roguelike)
- **Módulo 06:** [3D Fundamentals (FPS Retro)](./aulas/README.md#módulo-06-3d-fundamentals-fps-retro)
- **Módulo 07:** [Inteligência Artificial (Stealth Game)](./aulas/README.md#módulo-07-inteligência-artificial-stealth-game)
- **Módulo 08:** [Networking & Multiplayer (Arena)](./aulas/README.md#módulo-08-networking--multiplayer-arena)
- **Módulo 09:** [Shaders & VFX (Juice)](./aulas/README.md#módulo-09-shaders--vfx-juice)
- **Módulo 10:** [Plugins & Tooling (Extensibilidade Nativa)](./aulas/README.md#módulo-10-plugins--tooling-extensibilidade-nativa)
- **Módulo 11:** [TCC (Projeto Final)](./aulas/README.md#módulo-11-tcc-projeto-final)
- **Bônus:** [Rust & GDExtension (Performance Extrema)](./aulas/README.md#bônus-rust--gdextension-performance-extrema)
- **Bônus:** [DevOps & CI/CD (Automação de Builds)](./aulas/README.md#bônus-devops--cicd-automação-de-builds)
- **Bônus:** [Arquitetura de Modding & DLCs](./aulas/README.md#bônus-arquitetura-de-modding--dlcs)
- **Bônus:** [Matemática para Engenheiros de Jogos](./aulas/README.md#bônus-matemática-para-engenheiros-de-jogos)

---

## 🏛️ Pós-Graduação: O MBA (Arquitetura Avançada)

Aqui estão os **Documentos Mestres**.
Estes arquivos não são tutoriais; são especificações técnicas, manifestos de arquitetura e padrões de design avançados. Eles assumem que você já sabe programar e quer aprender a **arquitetar**.

### 🏛️ Fundamentos da Engenharia

- [01. Fundamentos Godot](./mba/01_GodotFundamentals.md): O modelo mental de Nodes e SceneTree.
- [02. Resource-Oriented Programming (ROP)](./mba/02_ResourceOrientedProgramming.md): O coração do Machi Game Style.
- [03. Singletons & Autoloads](./mba/03_Singletons.md): Gerenciamento global correto.
- [04. Gestão de Cenas e Dados](./mba/04_SceneAndDataManagement.md): Loading e persistência.

### 🏗️ Estruturas de Dados e Algoritmos

- [05. Máquinas de Estado](./mba/05_StateMachines.md): FSMs robustas e desacopladas.
- [06. Query Hash Map](./mba/06_HashMap.md): Otimização de busca O(1).

### 📦 Sistemas de Produção

- [07. Sistema de Inventário](./mba/07_Inventory.md): Arquitetura de itens e containers.
- [08. Tradução e Localização (i18n)](./mba/08_Translations.md): Preparando para o mundo.
- [09. Testes e QA](./mba/09_Testing_QA.md): Garantia de qualidade automatizada.
- [12. A Ficha de RPG Suprema](./mba/12_CharacterSheet.md): Integração de sistemas complexos.

### 🎨 Audiovisual e Game Feel

- [10. Game Feel & Juice](./mba/10_GameFeel.md): Polimento e feedback visual.
- [11. Shaders & Materiais](./mba/11_Shaders.md): Introdução técnica a VFX.

### 🚀 Tópicos Avançados (Extensão)

- [13. Plugins & Tooling](./mba/13_Plugins.md): Criando ferramentas para o editor.
- [14. GDExtensions](./mba/14_GDExtensions.md): Performance nativa com C++.
- [15. Rust AI Extension](./mba/15_RustAIExtension.md): Inteligência Artificial com Rust.

> "Não escreva código que funciona. Escreva código que sobrevive."

---

## 1. Escolhendo a Estrutura Certa: Um Guia de Decisão Arquitetural

Na Godot, existem diversas formas de organizar seu código e seus dados. A escolha da ferramenta certa para cada problema é crucial para evitar o "código desorganizado" e garantir a escalabilidade do seu projeto.

### 1.1. Global Class & Singletons: O Poder do `class_name`

**O que é?**
A palavra-chave `class_name` registra seu script como um **Tipo Global** no editor. Isso permite que o script seja acessado por nome em qualquer lugar do projeto, sem `load()` ou `preload()`.

**Visão Machi:**
Embora a ferramenta seja a mesma (`class_name`), ela tem dois usos arquiteturais distintos e opostos. Entender quando usar cada um é vital.

**Uso A: Herança (Base Class)**
Você cria uma classe para servir de **molde** para outras.

- **Como usar:** Você define `class_name Enemy`. Você **NUNCA** coloca o nó `Enemy` puro na cena. Você cria scripts que herdam dele (`extends Enemy`) ou cenas que o usam como base.
- **Exemplo:** `Enemy` (Base) -> `Goblin` (Filho). O jogo só tem Goblins, nunca "Enemies" genéricos.

**Uso B: Singleton Manual (Manager)**
Você cria uma classe para ser um **gerente único** dentro de uma cena.

- **Como usar:** Você define `class_name BattleManager`. Você coloca esse nó **DIRETAMENTE** na cena. Você **NUNCA** herda dele.
- **O Padrão:** Adicione `static var instance` no script para permitir acesso global (`BattleManager.instance.start_battle()`) sem precisar de `get_node`.
- **Diferença do Autoload:** O Autoload vive para sempre (entre cenas). O Singleton Manual vive apenas enquanto a cena atual existir (ex: um Manager de um minigame específico).

**⚠️ A Pegadinha: Script vs. Cena**
Diferente dos **Autoloads** (que podem carregar uma cena `.tscn` inteira com todos os filhos), o `class_name` registra apenas o **Script**.

- Se você for em "Create New Node" e selecionar seu `BattleManager`, a Godot criará apenas o Nó raiz com o script anexado.
- **Os filhos (Timers, AudioPlayers, UI) NÃO virão junto.**

**💡 O "Hack" da PackedScene (Self-Instantiating)**
O problema: Ao adicionar um nó via `class_name` (Create Node), ele vem "pelado" (sem os filhos da cena `.tscn`).
A solução: Faça o script carregar seus próprios filhos automaticamente.

```gdscript
# battle_manager.gd
class_name BattleManager extends Node

static var instance: BattleManager

# Referência para a cena que contém a "carne" do manager (UI, Sons, Timers)
# Use 'Copy UID' no FileSystem para pegar esse caminho
@export var _impl_scene: PackedScene = preload("uid://c8j2k3l4m5n6")

func _enter_tree():
    if instance:
        queue_free()
        return
    instance = self

        # O Hack: O Singleton se "recheia" instanciando sua cena como filha

        # Mas verificamos se ele já tem filhos para evitar duplicação (caso já esteja na cena)

        if _impl_scene and get_child_count() == 0:

            var impl = _impl_scene.instantiate()

            add_child(impl)

```

Agora, sempre que você criar um `BattleManager` (seja por código ou editor), ele trará toda a sua estrutura junto, mas de forma segura e única.

### 1.2. Resource (`extends Resource`)

**O que é?**
Um `Resource` é um container de dados serializável que pode ser salvo em disco (com a extensão `.tres` ou `.res`). Diferente dos `Nodes`, Resources não fazem parte da `SceneTree` e não têm processamento próprio (como `_process` ou `_physics_process`). Eles são pura informação.

**Visão Machi:**
A ferramenta mais poderosa e subutilizada da Godot é o `Resource`. Pense neles como "fichas de RPG" para seus objetos. Eles servem para **Dados Compartilhados**. Se 100 inimigos usam o mesmo `zombie_stats.tres`, apenas **uma** cópia desse arquivo existe na memória RAM, economizando recursos preciosos. Resources facilitam a injeção de dependência e a edição por game designers sem tocar em código.

**Quando usar:**

- Para definir configurações de itens, armas, inimigos (HP máximo, velocidade base, etc.).
- Para árvores de diálogo, definições de habilidades, ou qualquer dado que precise ser compartilhado ou persistido.
- Para injeção de dependência: exporte uma variável `Resource` no seu `Node` para que você possa arrastar um `.tres` e mudar o comportamento do `Node` facilmente pelo editor.
- Para serializar dados complexos para o sistema de `Save Game`.

**Quando NÃO usar:**

- Para guardar estado volátil da instância. Por exemplo, o `current_hp` de um inimigo deve ir no `Node` que representa o inimigo em cena, enquanto `max_hp` (o dado fixo) vai no `Resource`.
- Para lógica de gameplay complexa que precisa interagir com a `SceneTree` ou processamento por frame. Resources podem ter funções, mas elas devem operar _apenas_ nos dados do próprio Resource.

```gdscript
# Exemplo: Definindo atributos de um inimigo usando Resource
# enemy_stats.gd
class_name EnemyStats extends Resource

@export var max_health: int = 100
@export var move_speed: float = 50.0
@export var attack_damage: int = 10

# Opcional: Funções que operam nos próprios dados do Resource
func get_damage_per_second() -> float:
    return float(attack_damage) * 0.5 # Exemplo: 0.5 ataques por segundo
```

```gdscript
# Exemplo: Usando o Resource em um Node
# enemy.gd
extends CharacterBody2D

@export var stats: EnemyStats # Arraste um .tres aqui!

var current_health: int

func _ready():
    if stats:
        current_health = stats.max_health
        print("Inimigo com HP: ", current_health, " e velocidade: ", stats.move_speed)
    else:
        push_warning("EnemyStats Resource não atribuído!")
```

### 1.3. Autoload (Singleton Global)

**O que é?**
Um Autoload (também conhecido como Singleton Global) é um `Node` ou `Script` que o Godot carrega automaticamente na inicialização do jogo e o mantém ativo durante toda a execução, independentemente da cena que esteja carregada. Ele é anexado diretamente à `root` da `SceneTree` e é acessível globalmente por seu nome.

**Visão Machi:**
Autoloads são seus **Sistemas Globais**. Eles são a solução perfeita para gerenciadores que precisam persistir entre cenas e ser acessíveis de qualquer lugar. Pense em sistemas de áudio, troca de cenas, gerenciamento de persistência de dados (save/load), analytics, ou qualquer lógica central do seu jogo.

**Quando usar:**

- Para gerenciadores de áudio (ex: `SoundManager`).
- Para controle de cenas e transições (ex: `SceneLoader`).
- Para um sistema de `Save/Load` que precisa ser acessível de qualquer lugar.
- Para um `GameManager` que mantém o estado geral do jogo (pontuação, fases, etc.).
- Para qualquer funcionalidade que precise existir uma única vez e ser globalmente acessível.

**Quando NÃO usar:**

- Para lógica de gameplay local que pertence a uma cena específica ou a um nó específico. Se a funcionalidade pode ser destruída e recriada com a cena, ela não deve ser um Autoload.
- Para evitar a passagem de referências. Abusar de Autoloads pode levar ao "Singleton Monstro", um Autoload gigante que sabe e faz demais, quebrando o princípio da Responsabilidade Única.

```gdscript
# Exemplo: Um GameManager Autoload para controlar o estado do jogo
# global.gd (ou game_manager.gd, configurado como Autoload "Global")
extends Node

var current_score: int = 0
var current_level: int = 1

func add_score(amount: int) -> void:
    current_score += amount
    print("Pontuação atual: ", current_score)

func go_to_next_level() -> void:
    current_level += 1
    # Global.SceneLoader.change_scene("res://level_" + str(current_level) + ".tscn")
    print("Indo para o nível: ", current_level)
```

**Como configurar um Autoload:**

1. Vá em `Project -> Project Settings -> Autoload`.
2. Clique no ícone de pasta para selecionar seu script ou cena.
3. Dê um `Node Name` (ex: `Global` ou `SoundManager`).
4. Clique em `Add`. Agora ele estará disponível globalmente pelo nome que você deu.

`

### 1.4. Inner Class (`class Nome:`)

**O que é?**
Uma Inner Class (ou classe aninhada) é uma classe definida dentro de outra classe ou script. Ela é útil para agrupar dados ou funcionalidades que estão estritamente relacionadas à classe externa e não precisam ser expostas globalmente.

**Visão Machi:**
Considere as Inner Classes como **Structs/Helpers locais**. Elas são perfeitas para encapsular dados complexos temporários ou lógica auxiliar que é restrita a um único arquivo. Isso ajuda a manter o namespace limpo e a coesão do código.

**Quando usar:**

- Para definir estruturas de dados personalizadas (similar a `structs` em outras linguagens) que serão usadas apenas dentro daquele script.
- Para agrupar constantes ou enumerações que são específicas da classe externa.
- Para criar pequenos helpers de lógica que não precisam ser instanciados como `Nodes` ou `Resources` separados e não serão reutilizados por outros scripts.

**Quando NÃO usar:**

- Se a classe precisa ser usada por outros scripts, Nodes ou Resources. Nesse caso, ela deve ser extraída para um arquivo `.gd` próprio e, se necessário, registrada como uma `class_name`.
- Para lógica que precisa interagir com a `SceneTree` ou ter um ciclo de vida independente (use `Node` ou `Resource`).

```gdscript
# Exemplo: Usando Inner Class para definir um tipo de item dentro de um inventário
# inventory.gd
extends Node

class ItemSlot:
    var item_id: String
    var quantity: int

    func _init(id: String, qty: int):
        item_id = id
        quantity = qty

var slots: Array[ItemSlot] = []

func add_item(id: String, qty: int) -> void:
    var new_slot = ItemSlot.new(id, qty)
    slots.append(new_slot)
    print("Item adicionado: ", new_slot.item_id, " x ", new_slot.quantity)

# Exemplo de acesso (apenas dentro do script inventory.gd ou instâncias dele)
# func _ready():
#    add_item("sword_of_fire", 1)
#    add_item("potion_hp", 5)
#
#    for slot in slots:
#        print("Slot: ", slot.item_id, ", Qty: ", slot.quantity)
```

## Com esta seção, encerramos a primeira parte do nosso guia arquitetural. Entender a finalidade e as limitações de cada uma dessas estruturas é o primeiro passo para construir jogos robustos e escaláveis.

## 2. Scripts vs. Classes: Desmistificando o GDScript

**O que é?**
No Godot, a distinção entre um "script" e uma "classe" é fundamental para entender a arquitetura do motor. Muitos iniciantes se referem a arquivos `.gd` como "scripts", mas na realidade, **todo arquivo `.gd` é implicitamente uma classe**.

**Visão Machi:**
Quando você anexa um arquivo `.gd` a um `Node` na Godot, você não está simplesmente "adicionando um comportamento". Você está, na verdade, transformando aquele nó em uma **instância da classe** definida no seu arquivo `.gd`. Isso significa que seu nó herda todas as propriedades e métodos da classe base (`Node`, `Sprite2D`, `CharacterBody2D`, etc.) e adiciona suas próprias definições.

- Você não precisa de `class_name` para usar um script. Ao anexar `player.gd` a um Node, aquele Node se torna uma instância daquela classe anônima.
- A linha `extends CharacterBody2D` (ou qualquer outro nó) é crucial: ela estabelece a hierarquia de herança, definindo o "É um" relacionamento (ex: "Meu `Player` **É UM** `CharacterBody2D`").

**Analogia:**
Pense em um blueprint de uma casa (a classe) e a casa construída a partir dele (a instância). Quando você "anexa um script", é como pegar um blueprint especializado de "Casa do Jogador" e dizer que o seu "Nó Genérico de Casa" agora **é** essa "Casa do Jogador", com todas as suas características e funcionalidades.

**Exemplo:**

```gdscript
# meu_personagem.gd
extends CharacterBody2D # Meu personagem É UM CharacterBody2D

var velocidade_movimento: float = 100.0

func _physics_process(delta: float) -> void:
    var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
    velocity = input_direction * velocidade_movimento
    move_and_slide()
```

Neste exemplo, ao anexar `meu_personagem.gd` a um `CharacterBody2D` na sua cena, você está dizendo que aquele nó agora é uma instância da classe `MeuPersonagem`. Ele tem acesso a `velocity`, `move_and_slide()` (do `CharacterBody2D`) e à nova variável `velocidade_movimento` e ao método `_physics_process` que você definiu.

Compreender essa natureza de "classe" dos seus scripts é o primeiro passo para pensar em termos de Programação Orientada a Objetos na Godot, um pilar fundamental do Machi Class.

`

`

`

---

## 3. O Poder das Anotações (`@`): Comandos ao Compilador e Editor

**O que são Anotações?**
No GDScript moderno, as anotações (prefixadas com `@`) são mais do que simples comentários. Elas são diretivas poderosas que comunicam intenções tanto para o compilador do GDScript quanto para o editor da Godot, configurando o comportamento do seu script de maneiras que vão além da lógica de execução. Elas permitem que você adicione metadados e controle aspectos como a visibilidade de variáveis no Inspector, a execução de código no editor, ou a imposição de contratos de herança.

**Visão Machi:**
Dominar as anotações é essencial para escrever código limpo, auto-documentado e para tirar o máximo proveito das ferramentas visuais da Godot. Em vez de usar `setget` complexos ou workarounds no editor, as anotações oferecem uma forma declarativa e eficiente de alcançar seus objetivos.

A seguir, exploraremos as anotações mais cruciais para o desenvolvimento profissional na Godot:

`

### 3.5. `@warning_ignore`: Gerenciando Advertências do Linter

**O que é?**
A anotação `@warning_ignore` permite que você suprima advertências específicas do linter do GDScript para o escopo de um script, função ou até mesmo uma linha. O linter é uma ferramenta de análise estática que verifica seu código em busca de problemas de estilo, erros potenciais ou inconsistências, e as advertências são mensagens que ele gera.

**Visão Machi:**
Embora o linter seja uma ferramenta valiosa para manter a qualidade do código, há situações legítimas em que uma advertência pode ser ignorada, especialmente em código específico do jogo onde a regra do linter pode não se aplicar ou onde uma otimização peculiar é intencional. O `@warning_ignore` é uma ferramenta para manter seu console limpo, mas deve ser usado com parcimônia e justificativa, para não esconder problemas reais.

**Quando usar:**

- Quando uma variável ou parâmetro de função é intencionalmente não utilizado (ex: em um slot de sinal que recebe muitos parâmetros, mas você só precisa de um ou dois).
- Quando você está desenvolvendo uma funcionalidade rapidamente e quer focar na lógica principal antes de refatorar e remover as advertências.
- Para casos muito específicos onde o linter está gerando um falso positivo ou onde a regra interfere com um padrão de design específico que você está implementando.

**Quando NÃO usar:**

- Para esconder preguiça ou má prática. Sempre prefira refatorar o código para resolver a advertência em vez de simplesmente ignorá-la.
- Em excesso. Se você está ignorando muitas advertências, é um sinal de que algo pode estar errado com seu código ou com sua compreensão dos padrões.

**Exemplo:**

```gdscript
# my_node.gd
extends Node

# Ignora a advertência para um parâmetro não utilizado em uma função específica
func _on_area_entered(area: Area2D, area_owner: Node2D, other_area: Area2D, other_area_owner: Node2D): #line
    @warning_ignore("unused_parameter")
    print("Área ", area.name, " entrou. Owner: ", area_owner.name)
    # Apenas 'area' e 'area_owner' são usados aqui, os outros são ignorados

# Ignora a advertência de variável não utilizada para todo o script
@warning_ignore("unused_private_class_variable")
var _minha_variavel_secreta: String = "valor" # Pode ser usada mais tarde ou via reflexão
```

### 3.6. `@abstract`: Classes Abstratas e Contratos (Godot 4.5+)

**O que é?**
Introduzida em versões mais recentes do GDScript, a anotação `@abstract` permite definir uma classe (ou método) que não pode ser instanciada diretamente, servindo apenas como um modelo ou "contrato" para classes derivadas.

**Visão Machi:**
Classes abstratas são a pedra fundamental para arquiteturas polimórficas seguras. Elas impedem que desenvolvedores (incluindo você no futuro) usem classes base de forma errada (ex: colocar um `Inimigo` genérico na cena em vez de um `Goblin` ou `Orc`). Além disso, métodos abstratos forçam as classes filhas a implementar comportamentos específicos, garantindo que todas as variações do objeto sigam as mesmas regras.

**Quando usar:**

- Para criar classes base genéricas que nunca devem existir sozinhas no jogo (`Enemy`, `Weapon`, `Interactable`).
- Para definir um "contrato" de interface, obrigando todas as subclasses a terem certas funções (ex: todo `Inimigo` _precisa_ ter uma função `atacar()`).

**Quando NÃO usar:**

- Para classes concretas que você pretende instanciar.
- Se você não precisa impor regras estritas de herança.

**Exemplo:**

```gdscript
# animal.gd
@abstract # Ninguém pode fazer: var a = Animal.new() (Erro!)
class_name Animal extends Node

# Obriga quem herdar a implementar esta função
@abstract
func make_sound() -> void:
    pass

# dog.gd
class_name Dog extends Animal

# O compilador vai reclamar se você esquecer de implementar isso:
func make_sound() -> void:
    print("Woof!")
```

Com `@abstract`, você constrói uma fundação sólida onde o compilador trabalha a seu favor, prevenindo erros lógicos de arquitetura antes mesmo de o jogo rodar.

---

## 4. O Ciclo da Vida de um Nó: Inicialização, Loop e Morte

Entender a ordem exata em que as coisas acontecem na Godot (o "Life Cycle") é o que separa quem chuta soluções de quem resolve problemas. Muitos bugs de "Node not found" ou comportamentos estranhos de física vêm simplesmente de tentar fazer a coisa certa no momento errado.

Nesta aula, vamos dissecar os momentos cruciais da existência de um Nó.

### 4.1. Inicialização e Destruição: Do Nascimento ao Fim

Use esta referência para saber **onde** colocar seu código de setup.

**A. `_init()` - O Construtor**

- **Quando roda:** No momento exato em que o objeto é criado na memória (`.new()` ou instanciado).
- **Estado:** O nó ainda **NÃO** está na `SceneTree`. Ele não tem pai, não tem filhos acessíveis e não sabe onde está no mundo.
- **Uso Machi:** Configuração interna pura (inicializar arrays, dicionários). Evite interagir com outros nós aqui.

**B. `_enter_tree()` - A Chegada**

- **Quando roda:** Assim que o nó é adicionado à árvore de cenas, mas _antes_ de seus filhos.
- **Estado:** O nó tem acesso à `SceneTree` e ao seu `Viewport`, mas seus filhos ainda não estão prontos.
- **Uso Machi:** Registro em Managers globais (ex: `GameManager.register_enemy(self)`).

**C. `_ready()` - O Despertar**

- **Quando roda:** Apenas depois que **todos** os filhos do nó também entraram na árvore e rodaram seus próprios `_ready()`. É uma ordem "de baixo para cima" (filhos primeiro, pai depois).
- **Estado:** Tudo está pronto. Você pode acessar filhos (`$Sprite`), pais e autoloads com segurança.
- **Uso Machi:** Inicialização de gameplay, acesso a `@onready`, conexões de sinais locais e configuração visual inicial.

**D. `_exit_tree()` - A Despedida**

- **Quando roda:** Logo antes do nó ser removido da árvore (por troca de cena ou `queue_free()`).
- **Uso Machi:** Limpeza obrigatória. Desregistrar de Managers, desconectar sinais manuais (se necessário), salvar dados parciais antes da morte.

### 4.2. O Game Loop: `_process` vs `_physics_process`

A confusão entre esses dois é a causa #1 de "jitter" (tremulação) em jogos Godot.

**A. `_process(delta)` - O Loop Visual**

- **Frequência:** Variável. Tenta rodar o mais rápido possível (depende do FPS, VSync e lag da GPU).
- **O que é `delta`?** O tempo (em segundos) que passou desde o último frame.
- **Uso Machi:** Coisas que precisam parecer suaves visualmente, mas não afetam a simulação física.
  - Animações manuais de UI.
  - Interpolação de câmera.
  - Rotação de itens colecionáveis.
  - Input contínuo (verificação de teclas pressionadas).

**B. `_physics_process(delta)` - O Loop Físico**

- **Frequência:** Fixa (padrão 60 ticks por segundo). Configurável em Project Settings.
- **O que é `delta`?** É constante (geralmente 0.0166s).
- **Uso Machi:** **Toda** lógica que envolve movimento e colisão.
  - Mover `CharacterBody2D` ou `RigidBody`.
  - Detecção de Raycasts.
  - Lógica de Estado que precisa ser determinística.

**Regra de Ouro:** Se mexe o corpo do personagem (`move_and_slide`), vai no `physics`. Se mexe apenas o sprite ou a UI, vai no `process`.

### 4.3. Input: Capturando a Intenção do Jogador

A Godot tem uma hierarquia de quem "come" o evento de input primeiro. Entender isso evita que seu personagem pule quando você clica no botão de Pause.

1. **`_input(event)`**: O primeiro a saber. Captura tudo, antes da UI.

   - **Uso:** Atalhos globais de debug, screenshots, input que deve ignorar a UI.

2. **Control Nodes (`_gui_input`)**: Se o mouse estiver sobre um botão, o botão consome o evento.

3. **`_unhandled_input(event)`**: O mais importante para Gameplay. Só roda se ninguém acima consumiu o evento.
   - **Uso Machi:** **Sempre use este para gameplay** (pular, atirar, interagir). Assim, se o jogador clicar em um menu, o personagem não atira sem querer.

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("jump"):
        jump()
```

### 4.4. Controle de Fluxo e Memória

Comandos essenciais para gerenciar a existência dos seus objetos.

- **`queue_free()` vs `free()`**:

  - `free()`: Deleta IMEDIATAMENTE. Perigoso se o nó estiver sendo usado ou processando física. Pode crashar o jogo.
  - `queue_free()`: Agenda a morte para o final do frame ("Obrigado pelos serviços, pode ir embora quando acabar o expediente"). **Sempre use este para Nodes.**

- **`call_deferred("funcao")`**:

  - Pede para a Godot rodar essa função em um momento seguro (geralmente no próximo frame ocioso). Essencial quando você precisa alterar a física ou a árvore de cenas de dentro de um sinal ou thread que não permite modificações imediatas.

- **`await`**:
  - Pausa a execução da função atual (corrotina) até que um sinal seja emitido.
  - Exemplo: `await get_tree().create_timer(1.0).timeout` (Espera 1 segundo antes de continuar).

---

---

## 5. Tipagem Estrita: O Hábito dos Profissionais

GDScript é uma linguagem dinâmica, mas permite tipagem estática opcional. No Machi Game Style, **a tipagem estática não é opcional, é lei**.

**Por que?**

1. **Autocomplete:** O editor sabe o que `body` é, então ele sugere `body.take_damage()` automaticamente.
2. **Segurança:** Evita erros bobos como somar texto com número (`"10" + 5`).
3. **Performance:** O GDScript compilado com tipos é mais rápido.

### As Regras de Ouro da Tipagem

**1. Variáveis Tipadas**
Nunca declare uma variável sem dizer o que ela é. Se o valor inicial não deixar óbvio, force o tipo.

```gdscript
# Ruim (Infernir o tipo é aceitável, mas explícito é melhor)
var score = 0

# Bom
var score: int = 0
var player_name: String = "Hero"
var enemies_nearby: Array[Node2D] = [] # Typed Array!
```

**2. Funções com Contrato Claro**
Sempre defina o que entra e o que sai. Se não retorna nada, retorne `void`.

```gdscript
# Ruim
func take_damage(amount):
    hp -= amount

# Bom
func take_damage(amount: int) -> bool:
    hp -= amount
    return hp <= 0 # Retorna true se morreu
```

**3. Safe Casting (`as`)**
Ao receber um objeto genérico (como em sinais de colisão), converta-o para o tipo esperado para ganhar o autocomplete.

```gdscript
func _on_body_entered(body: Node2D) -> void:
    # Tenta converter 'body' para 'Enemy'. Se falhar, retorna null.
    var enemy := body as Enemy

    if enemy:
        enemy.take_damage(10) # O editor sabe que 'Enemy' tem essa função!
```

---

---

## 6. Arquitetura de Sinais: Call Down, Signal Up

Como seus nós conversam sem virar uma macarronada? Seguindo esta regra sagrada:

**O Mantra:**

> _"O Pai manda no Filho. O Filho avisa o Pai."_

### A. Call Down (Pai chama Filho)

O pai (quem está acima na árvore) detém a referência aos seus filhos. Ele tem autoridade para executar comandos diretamente neles.

_Exemplo:_ O `Player` diz para a `Gun`: "Atire agora!".

```gdscript
# No script do Player
$Gun.shoot()
```

### B. Signal Up (Filho avisa Pai)

O filho (quem está abaixo) **NUNCA** deve saber quem é seu pai. Se você fizer `get_parent().score += 1`, seu código quebra assim que você mudar a estrutura da cena. Em vez disso, o filho grita (emite um sinal) e quem estiver interessado que ouça.

_Exemplo:_ A `Gun` diz: "Estou sem munição!". Ela não sabe se o Player vai recarregar, tocar um som ou mostrar um ícone. Ela só avisa.

```gdscript
# No script da Gun
signal out_of_ammo

func shoot():
    if ammo <= 0:
        out_of_ammo.emit() # Grita!
```

### C. E os Irmãos?

Irmãos não devem se conversar diretamente. Eles brigam. Use o Pai como mediador.
_Exemplo:_ Se a `Gun` precisa avisar a `UI` (que é irmã do Player), a `Gun` emite sinal para o `Player`, e o `Player` atualiza a `UI`. Ou use um `EventBus` global para sistemas distantes.

---

---

## 7. Estudo de Caso: Arquitetando um Inimigo

Vamos aplicar tudo o que aprendemos para criar um sistema de inimigos escalável.

**O Problema:**
Precisamos de Goblins, Orcs e Trolls. Todos têm vida, movimento e ataque, mas com valores diferentes.

**A Solução Machi:**

1. **Os Dados (`EnemyStats.gd` - Resource):**
   Criamos a "Ficha de Personagem".

   ```gdscript
   class_name EnemyStats extends Resource
   @export var max_hp: int = 10
   @export var speed: float = 50.0
   @export var texture: Texture2D
   ```

   _Resultado:_ Criamos `goblin.tres`, `orc.tres` no editor.

2. **A Lógica Base (`Enemy.gd` - Global Class):**
   O cérebro genérico.

   ```gdscript
   class_name Enemy extends CharacterBody2D
   @export var stats: EnemyStats # Injeção de Dependência!

   func _ready():
       # Configura-se baseado nos dados
       $Sprite2D.texture = stats.texture
   ```

3. **A Cena (`Enemy.tscn`):**
   Uma única cena genérica com `Sprite2D`, `CollisionShape2D`.
   Para fazer um Goblin, instanciamos essa cena e arrastamos o `goblin.tres` para o slot `stats`.

**Vantagem:** Não precisamos de `Goblin.tscn`, `Orc.tscn`. Uma cena serve para todos, apenas mudando o Resource.

---

---

## 8. Organização de Pastas: Por Domínio, Não por Tipo

A maioria dos tutoriais diz para criar pastas `Scripts`, `Scenes`, `Sprites`. **Isso é errado para projetos grandes.** Quando você quer mudar o Player, você tem que abrir 3 pastas diferentes.

**O Jeito Machi (Feature-based):**
Agrupe tudo que pertence a uma funcionalidade no mesmo lugar.

```
res://
├── entities/           # Objetos do jogo
│   ├── player/         # Tudo do Player aqui!
│   │   ├── Player.tscn
│   │   ├── player_controller.gd
│   │   ├── player_skin.png
│   │   └── PlayerStats.tres
│   └── enemies/
│       ├── goblin/
│       └── orc/
├── systems/            # Gerenciadores globais
│   ├── audio/
│   └── save_system/
├── ui/                 # Interface
│   ├── hud/
│   └── menus/
└── resources/          # Dados compartilhados globais
    ├── items/
    └── skills/
```

Assim, se você deletar a pasta `player`, o Player some completamente, sem deixar lixo para trás.

---

---

## 9. Performance: Otimizando antes que trave

Não espere o jogo ficar lento. Adote estes hábitos:

1. **Object Pooling (Reciclagem):**
   Instanciar (`.instantiate()`) e deletar (`queue_free()`) é caro para a CPU. Para tiros, moedas e partículas, não destrua. Esconda e reutilize.

2. **Servers API:**
   Se você precisa de 10.000 balas, `Area2D` vai travar. Use `PhysicsServer2D` diretamente. É mais difícil de usar, mas 100x mais rápido.

3. **Evite `_process` desnecessário:**
   Se um objeto está fora da tela, use `VisibleOnScreenNotifier2D` para pausar a lógica dele.

---

---

## 10. Dados: Resources vs Dictionaries

A dúvida eterna: Onde guardo meus dados?

| Estrutura      | O que é?                                                    | Uso Principal                                                                         | Exemplo                                                    |
| :------------- | :---------------------------------------------------------- | :------------------------------------------------------------------------------------ | :--------------------------------------------------------- |
| **Resource**   | Arquivo estático (`.tres`). Tipado e editável no Inspector. | **Dados de Design (ReadOnly).** Coisas que você (o dev) define antes do jogo começar. | Stats de Monstros, Árvore de Itens, Configurações de Arma. |
| **Dictionary** | Estrutura chave-valor em memória. Flexível e dinâmica.      | **Estado de Runtime (SaveData).** Coisas que mudam enquanto o jogador joga.           | Inventário atual, Quests completadas, Posição do Player.   |

### O Padrão ROP (Resource-Oriented Programming)

No Machi Game Style, usamos Resources aninhados.

- Um `CharacterStats` tem um slot para uma `WeaponResource`.
- A `WeaponResource` tem um slot para um `ProjectileResource`.
  Isso permite criar combinações infinitas apenas arrastando arquivos no editor.

---

---

## 11. UI Profissional: Themes e Containers

Criar UI na Godot não é arrastar coisas aleatoriamente.

1. **Themes são Obrigatórios:**
   Nunca mude a fonte de um `Label` individualmente. Crie um `Theme` global (`main_theme.tres`). Se quiser mudar a fonte do jogo todo, você muda em um lugar só.

2. **Containers são Vida:**
   Nunca posicione botões manualmente (pixel perfect). Use `VBoxContainer`, `HBoxContainer`, `GridContainer`. Eles se adaptam a diferentes resoluções automaticamente.

3. **Separação Lógica/Visual:**
   O script do Menu (`MainMenu.gd`) não deve saber a cor do botão. Ele só deve saber o que acontece quando o botão é clicado (`_on_start_pressed`).

---

---

## 12. Áudio: Mais que Play e Stop

Não espalhe `AudioStreamPlayer` em cada moeda.

1. **Audio Buses (Mixer):**
   Configure no painel inferior: `Master` -> `Music`, `SFX`, `UI`.
   Isso permite criar um menu de "Volume" em 5 minutos.

2. **O `SoundManager`:**
   Crie um Autoload.
   - Errado: `$CoinSound.play()` (em cada moeda).
   - Certo: `SoundManager.play_sfx("coin_pickup")`. O Manager cuida de instanciar o player, tocar e deletar depois.

---

---

## 13. i18n: Tradução desde o Dia 1

Seu jogo vai ser em português? Ótimo. Mas não escreva português no código.

**A Regra:** Use chaves (Keys).

- Errado: `Label.text = "Jogo Acabou"`
- Certo: `Label.text = "GAME_OVER_MSG"`

O Godot usa arquivos CSV ou PO (Gettext) para trocar "GAME_OVER_MSG" por "Game Over" (EN) ou "Fim de Jogo" (PT) automaticamente. É fácil de configurar e salva semanas de refatoração depois.

---

---

## 14. Blueprints: Receitas de Arquitetura

Não reinvente a roda. Use estes padrões validados.

- **Inventário:** `Resource` (ItemData) + `Array` (Mochila).
- **Habilidades:** `Resource` (Efeito) + `Node` (Processador de Efeito).
- **Save System:** `Dictionary` -> `FileAccess`. Salve dados, não nós.
- **Quests:** `Resource` (Dados da Quest) + `Autoload` (QuestManager).
- **Máquina de Estados:** Nodes filhos (`Idle`, `Walk`, `Attack`) gerenciados por um Pai (`StateMachine`).

---

---

## 15. Extensibilidade: Criando suas Próprias Ferramentas

O Godot é feito em Godot. Você pode criar janelas, docks e inspetores customizados.

1. **EditorPlugins:** Crie ferramentas para sua equipe. Um editor de diálogos, um gerador de fases, um painel de debug.
2. **GDExtension:** Precisa de performance bruta (C++/Rust)? Use GDExtension. É como escrever código nativo da engine, mas compilado como uma DLL dinâmica.

---

---

## 16. Debugging: Se não medir, não melhora

"Está lento" não é feedback técnico.

1. **Monitores Customizados:**
   Adicione gráficos no debugger para acompanhar quantas balas existem, quantos inimigos estão vivos.

   ```gdscript
   Performance.add_custom_monitor("Inimigos Vivos", func(): return enemies.size())
   ```

2. **Profiler:**
   Use as abas **Profiler** (CPU) e **Visual Profiler** (GPU) para achar o gargalo exato.

---

---

## 17. QA: Testes Automatizados

A arquitetura Machi facilita testes porque desacopla dados de lógica.

- **Ferramenta:** Use o addon **GUT** (Godot Unit Test).
- **Estratégia:**
  - Teste seus `Resources` (Cálculos de dano, evolução de XP) isoladamente.
  - Teste suas `StateMachines` simulando inputs.

---

---

## 18. O Manifesto da Qualidade

Para um código ser considerado "Machi Style", ele deve obedecer:

1. **Zero Warnings:** O console deve estar limpo. Se tem warning, tem bug potencial.
2. **Typed Everything:** Nenhuma variável ou função sem tipo explícito.
3. **Resource First:** Dados sempre em `.tres`, nunca hardcoded em scripts.
4. **Separation:** UI não contém lógica de jogo. Lógica de jogo não acessa UI diretamente.

---

---

## 19. Git: O Seguro de Vida do Projeto

Versionamos **Código** e **Assets Originais**. Nunca artefatos gerados.

**Obrigatório no `.gitignore`:**

- `.godot/` (Cache gigante, nunca commite isso).
- `*.uid` (Evita conflitos de merge em IDs).

**Use Git LFS para binários:**

- `.png`, `.wav`, `.blend`, `.fbx`.

---

---

## 20. Style Guide: Falando a Mesma Língua

| O quê          | Como                         | Exemplo                |
| :------------- | :--------------------------- | :--------------------- |
| **Arquivos**   | `snake_case`                 | `player_controller.gd` |
| **Classes**    | `PascalCase`                 | `EnemyStats`           |
| **Variáveis**  | `snake_case`                 | `move_speed`           |
| **Privadas**   | `_snake_case`                | `_internal_timer`      |
| **Constantes** | `SCREAMING_SNAKE`            | `MAX_SPEED`            |
| **Sinais**     | `snake_case` (verbo passado) | `level_completed`      |

---

---

## Sua Primeira Missão (Checklist Dia 1)

Antes de codar, prepare o terreno:

1. [ ] **Git Init:** Crie o repo e adicione o `.gitignore` padrão da Godot.
2. [ ] **Project Settings:**
   - Setar resolução (ex: 1920x1080).
   - Configurar Input Map (`ui_accept`, `jump`, `fire`).
   - Nomear Collision Layers (Layer 1: World, Layer 2: Player...).
3. [ ] **Pastas:** Crie `entities/`, `resources/`, `ui/`, `assets/`.
4. [ ] **Theme:** Crie um `main_theme.tres` e defina a fonte padrão.

Agora você está pronto. Bem-vindo à elite. 🚀
