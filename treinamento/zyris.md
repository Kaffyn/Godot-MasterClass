# Zyris Framework

Meu nome é Machi, sou professor de Godot e estou desenvolvendo o **Zyris**, um framework modular em C++ (GDExtension) para Godot 4.5+.

**Servers / Services:** Funcionam como "sistemas nativos". Não estão na SceneTree, não estendem `Node` (geralmente `Object` ou gerenciados internamente em C++) e são acessados globalmente como Singletons da Engine (ex: `PhysicsServer2D`). Não possuem `_process` no sentido de Node, mas rodam em seus próprios loops internos.

- [Zyris Framework](#zyris-framework)
  - [Arquitetura \& Design Patterns](#arquitetura--design-patterns)
    - [1. Servers (C++ Singletons)](#1-servers-c-singletons)
    - [2. Resources (Data-Driven)](#2-resources-data-driven)
    - [3. Nodes (Visualizers)](#3-nodes-visualizers)
    - [4. Scenes (Prefabs)](#4-scenes-prefabs)
  - [Ability System](#ability-system)
    - [Componentes do Ability System](#componentes-do-ability-system)
    - [Systems Internos](#systems-internos)
    - [Multiplayer](#multiplayer)
    - [Debug \& Tooling](#debug--tooling)
    - [Integração com o Sonhar](#integração-com-o-sonhar)
    - [Princípio Central](#princípio-central)
  - [Behavior Tree](#behavior-tree)
    - [O Cérebro Híbrido](#o-cérebro-híbrido)
    - [Integração com Ability System](#integração-com-ability-system)
    - [Multiplayer \& Determinismo](#multiplayer--determinismo)
    - [Perfis Dinâmicos](#perfis-dinâmicos)
  - [Director](#director)
  - [Gaia](#gaia)
    - [Arquitetura: Clima Sistêmico](#arquitetura-clima-sistêmico)
    - [Integração de Gameplay (Ability System)](#integração-de-gameplay-ability-system)
    - [Integração com outros Sistemas](#integração-com-outros-sistemas)
  - [Inventory](#inventory)
  - [Kinesis](#kinesis)
  - [Sonhar](#sonhar)
  - [Mimir](#mimir)
  - [Mythos](#mythos)
  - [Osmo](#osmo)
    - [Arquitetura: Virtual Cameras](#arquitetura-virtual-cameras)
    - [Conceitos Fundamentais](#conceitos-fundamentais)
    - [Integração com Director](#integração-com-director)
  - [Quests](#quests)
  - [Sounds](#sounds)
    - [O Compositor (Visual Music Sequencer)](#o-compositor-visual-music-sequencer)
    - [Arquitetura: Containers \& Events](#arquitetura-containers--events)
    - [Conceitos Fundamentais](#conceitos-fundamentais-1)
    - [Workflow](#workflow)
  - [Synapse](#synapse)
    - [Arquitetura: Native Physics Backend](#arquitetura-native-physics-backend)
    - [Conceitos Fundamentais](#conceitos-fundamentais-2)
  - [Yggdrasil](#yggdrasil)
    - [Game State \& Lifecycle](#game-state--lifecycle)
    - [Level Streaming (Portals \& Chunks)](#level-streaming-portals--chunks)
    - [Emissores (Nodes)](#emissores-nodes)
  - [Yggdrasil](#yggdrasil-1)

---

## Arquitetura & Design Patterns

O Zyris se diferencia por sua **rigidez estrutural** em prol da escalabilidade e performance. Abaixo definimos os três pilares de implementação para qualquer módulo do framework.

### 1. Servers (C++ Singletons)

> **Extends:** `Object`

Funcionam como subsistemas nativos da engine. Eles **NÃO** são Nodes, não estão na SceneTree e não possuem `_process` convencional.

- **Ciclo de Vida:** Gerenciados internamente pela Engine e acessíveis globalmente via Singletons (como `PhysicsServer2D` ou `DisplayServer`).
- **Função:** Processamento pesado, gerenciamento de estado global e queries espaciais.
- **Exemplo:** `InventoryServer`, `SynapseServer`, `GaiaServer`, `SoundServer`, `QuestServer`, `DirectorServer`, `SonharServer`, `OsmoServer`, `MimirServer`, `Kinesis`, `Yggdrasil`.

**Por que usar Servers?**

1. **Persistência Real:** Nodes morrem quando a cena muda. Servers vivem desde o boot até o shutdown (ideal para Inventário, Quests, SaveData).
2. **Árbitro Global:** Quando 10 Câmeras querem ser ativas, quem decide? O `OsmoServer`. Quando 50 sons tocam, quem corta o menos importante? O `SoundServer`.
3. **Performance:** Servers rodam lógica pesada (Spatial Hashing, AI Ticks) em C++ puro, sem o overhead da SceneTree.
4. **Clean Architecture:** Separa a _Lógica_ (Server) da _Visualização_ (Node). O Node apenas "pede", o Server "executa".

### 2. Resources (Data-Driven)

> **Extends:** `Resource`

Unidade básica de dados e lógica de comportamento. No Zyris, a lógica de gameplay reside nos Resources, não nos Nodes.

- **Conceito:** "Scriptable Objects" com superpoderes.
- **Função:** Definir **O QUE** acontece (Stats, Skills, Quests).
- **Vantagem:** Serializável, compartilhável entre instâncias e editável no Inspector.

### 3. Nodes (Visualizers)

> **Extends:** `Node`, `Node2D`, `Node3D`

Apenas a manifestação visual ou auditiva do sistema na cena.

- **Função:** Visualizar o estado atual ou capturar input.
- **Regra:** Nodes não devem conter lógica de persistência ou regras de negócio críticas. Eles apenas consultam os Servers ou leem Resources.

### 4. Scenes (Prefabs)

> **Composition:** Nodes + Resources (`.tscn`)

O Zyris fornece implementações de referência prontas para uso, combinando Nodes e Resources para resolver problemas comuns de Gameplay e UI.

- **Conceito:** "Lego Blocks" pré-montados.
- **Função:** Acelerar o desenvolvimento entregando componentes complexos já configurados.
- **Exemplos:**
  - `ItemPickup.tscn`: (Area3D/2D + Sprite/Mesh + `Item` Resource).
  - `InventorySlot.tscn`: (TextureRect + Label + Lógica de Drag & Drop).
  - `VirtualJoystick.tscn`: (Control + TouchScreenButton do módulo **Kinesis**).

## Ability System

> Motor de Gameplay Orientado a Dados, Contexto e Composição
>
> **Interface Jogador -> Jogo:** Transforma intenção (Input) em Realidade (Gameplay). O sistema valida e executa o que o jogador deseja fazer.
>
> **Arquitetura (Hash Map Invertido):**
> Diferente de uma FSM comum, aqui **os Resources definem quando querem rodar**.
> O sistema indexa essas intenções e o _Ability System_ seleciona o melhor candidato para o contexto atual.

O Ability System é um package central do Framework responsável por decisão, seleção e aplicação de gameplay.

Ele não executa ações diretamente, não depende de FSM tradicional e não utiliza singletons globais.
Seu papel é avaliar o contexto atual, consultar possibilidades válidas e decidir o melhor candidato com base em dados.

O sistema é orientado a Resources, determinístico, escalável e compatível com multiplayer.

**Servers / Services:** Nenhum.

> O sistema é orientado a Resources.
> O sistema é instanciado conscientemente e pode existir múltiplas vezes (player, NPC, boss, summon, companion).

**Nodes:**

- **AbilitySystem (extends Node):** Núcleo do sistema. Responsável por:
  - Construção do Contexto
  - Consulta indexada de candidatos
  - Validação de requisitos
  - Scoring
  - Seleção
  - Execução por fases
  - Gerenciamento de ciclo de vida

**Resources Fundamentais:**

- Character
- State
- Skill
- Effect

Todos os Resources são orientados à composição, formados por `AbilityComponents`.

### Componentes do Ability System

**Character (Resource)**

O `Character` representa a Character Sheet viva.
Ele é o modelo completo do personagem, independente de Node, cena ou representação visual.

> Pense nele como “o que o personagem é e tem”, não o que ele está fazendo agora.

O `Character` é a fonte de verdade do gameplay. Ele não executa lógica.

**Responsabilidades do Character:**

- Definir atributos base e máximos
- Armazenar progressão
- Conter referências para States, Skills, Effects e Itens
- Servir como alvo principal de aplicação de gameplay

**Componentes de Character:**

- Atributos base e atuais
- Pools de States, Skills e Effects
- Inventário
- Tags e Flags
- Progressão
- Dados de versão

**AbilityComponent**

Classe base abstrata usada por:

- Character
- State
- Skill
- Effect

**AbilityComponents:**

- Descrevem regras
- Descrevem consequências
- Não executam lógica

**State (Resource)**

States representam modos de existência do personagem.
Um State pode ser híbrido (ex: ataque + dash).

**Componentes de State:**

- Filter Components
- Score Components
- Movement Modifiers
- Physics Modifiers
- Combat Modifiers
- Combo Rules
- Reaction Rules
- Cost Definition
- Duration & Cooldown
- Lifecycle Rules

> Um State só é válido se o Character e o Contexto permitirem.

**Effect (Resource)**

Effects representam modificações temporárias ou condicionais aplicadas ao Character.

**Componentes de Effect:**

- Entry
- Exit
- Elemental
- Buffs / Debuffs
- Impact
- Costs
- Duration & Cooldown

**Skill (Resource)**

Skills representam desbloqueios e permissões de gameplay, não ações diretas.

**Componentes de Skill:**

- Requirements Skills
- Unlock
- Effect
- State
- Impact
- Buffs
- Elemental
- Costs
- Duration & Cooldown

### Systems Internos

**Context System:**

Constrói um snapshot imutável considerando:

- Character, Alvo(s), Ambiente, Superfície
- Estados físicos, Tags e Flags
- **Environmental Tags:** `Env.Weather.Rain`, `Env.Biome.Desert` (Injetadas pelo Gaia)
- States e Effects ativos
- Seed determinístico de RNG, Timestamp lógico

**Candidate Query System (Hash Map Invertido):**

A descoberta de candidatos e a filtragem fazem parte de um único sistema.
O Ability System utiliza Hash Maps Invertidos indexados por Enums de filtro, permitindo queries rápidas e previsíveis.

**Pipeline:** Construção do Context -> Consulta aos Hash Maps -> União -> Validação -> Scoring -> Seleção -> Execução.

**Scoring System:**

Após validação, cada candidato é pontuado. Scoring é explícito e previsível.
Composto por: Base Score, Modificadores, Pesos, Penalidades, Fator aleatório.

**Selection System:**

O sistema seleciona o vencedor baseado em: Maior score, Top N ou Escolha ponderada.

**Execution System:**

Execução ocorre em fases bem definidas:

1. PreCheck
2. Cost Commit
3. Execution
4. PostExecution
5. Cleanup

**Apply System:**

Aplica consequências no Character: Entrada/saída de States, Aplicação de Effects, Modificações de atributos.

**Lifecycle System:**

Gerencia Duração, Manutenção, Refresh, Stacking e Expiração.

**Reaction & Interruption:**

Nada reage automaticamente. O Ability System decide baseado em Prioridades, Níveis de interrupção e Imunidades.

### Multiplayer

- Autoridade no servidor
- Execução determinística
- Replicação de Context
- Rollback e prediction

### Debug & Tooling

O Ability System expõe dados (Context Snapshot, Candidates, Scores, Timeline) que o Sonhar visualiza.

**Integração com o Sonhar:**

O Ability System possui o seu **Dominio no Sonhar**, onde as regras de combate, skills e efeitos são forjados. Diferente do Behavior Tree (lógica), aqui desenha-se o fluxo de dados e eventos do gameplay.

**Funcionalidades do Domínio:**

- **Flow Editor:** Visualização completa do fluxo de uma habilidade através de um grafo visual (Input → Cast Time → Projectile → Impact → Damage), permitindo compreender e validar toda a cadeia de execução sem rodar o jogo.
- **Tag Management:** Sistema visual hierárquico para gerenciar `GameplayTags` e suas relações (ex: `Damage.Fire` herda de `Damage`), com coloração por categoria e busca inteligente.
- **Effect Composition:** Editor de `GameplayEffects` para criar buffs/debuffs complexos através de composição visual de componentes, eliminando a necessidade de código para efeitos novos.
- **Context Debugger:** Visualização em tempo real do `AbilityContext` durante o gameplay, mostrando todos os candidatos válidos, scores e o vencedor selecionado.

**Estrutura Técnica:**

- **Componente Visual:** `SonharGraph` (editor de grafos node-based)
- **Componente de Dados:** `AbilityNode` (estende `SonharNodeResource`)
- **Tipos de Nós:** `Task` (ações), `Event` (triggers), `Effect` (modificadores)

**Integração com a Biblioteca:**

- **Assets:** Busca e filtro rápido de `Character`, `State`, `Skill` e `Effect` com drag & drop para o Flow Editor.
- **Workbench:** Edição rápida de valores (dano, custo, duração) sem abrir o editor completo.
- **CraftTable:** Wizards de criação para novos efeitos e skills a partir de templates pré-configurados.

### Princípio Central

> Character existe
> Contexto acontece
> Ability System decide
> Sonhar torna tudo editável

Este package não define ações.
Ele define possibilidades, restrições e decisões.

---

## Behavior Tree

> Cérebro Estratégico & Lógica Viva
>
> **Interface Jogador -> Jogo:** Define a autonomia do avatar, permitindo que comandos do jogador sejam interpretados inteligentemente.
>
> **Stack Híbrida:**
>
> - **Runtime (C++):** GDExtension de alta performance para o jogo final (No-Python dependency).
> - **Training (Python):** Backend de desenvolvimento para treinar redes neurais e conectar com a Gemini API.

Este package implementa um motor de IA de última geração, com um **Editor Visual (Main Panel)** inspirado na Unreal/LimboAI e um pipeline de treinamento nativo.

**Servers / Services:** Nenhum.

**Nodes:**

- **BehaviorTreePlayer (extends Node):** Executor de runtime da árvore (C++).
  - Multithreaded tick execution.
  - Gerenciamento de memória via Pools.

**Resources:**

- **BehaviorTree:** O grafo lógico.
- **RLConfig:** Configuração de treinamento (Sensores/Objetivos).
- **MachiBrain:** Arquivo otimizado (`.machi_brain`) contendo a Policy treinada.

### O Cérebro Híbrido

A arquitetura integra duas inteligências:

1. **Reflexos (Reinforcement Learning):** Para combate e física (Bosses). Treinado nativamente via Python backend e executado em C++.
2. **Cognição (Gemini Coach):** O Gemini analisa logs de treino para ajustar a dificuldade e gera assets estáticos (diálogos) em tempo de desenvolvimento.

### Integração com Ability System

| Sistema            | Papel          | Responsabilidade                                              |
| :----------------- | :------------- | :------------------------------------------------------------ |
| **Behavior Tree**  | **Estratégia** | Decide _o que_ fazer (ex: "Inimigo perto -> Atacar").         |
| **Ability System** | **Execução**   | Decide _como_ fazer (ex: "Qual ataque usar? Tenho stamina?"). |

### Multiplayer & Determinismo

- **Server Authority:** A árvore executa e decide exclusivamente no Servidor.
- **Client Prediction:** O Cliente prevê **Intenções**.
- **Snapshot Interpolation:** Blackboard replicado para killcams perfeitas.

**Integração com o Sonhar:**

O Behavior Tree possui o seu **Domínio no Sonhar**, o laboratório onde a cognição da IA é desenhada. Foca em decisões lógicas e fluxos de comportamento complexos.

**Funcionalidades do Domínio:**

- **Visual Debugging:** Veja a árvore "pensando" em tempo real com nós ativos que brilham e pulsam, permitindo debugar a lógica de IA sem guesswork.
- **Blackboard Introspection:** Visualize e edite a memória da IA (variáveis, estados) em tempo real durante o gameplay, facilitando o tuning comportamental.
- **Sub-Trees:** Criação de comportamentos modulares reutilizáveis (ex: "Fugir", "Patrulhar", "Investigar") que podem ser compostos em árvores maiores.
- **Performance Profiler:** Visualização de custo computacional de cada nó, identificando gargalos na lógica de IA.

**Estrutura Técnica:**

- **Componente Visual:** `SonharGraph` (editor de grafos node-based)
- **Componente de Dados:** `BTTask` (estende `SonharNodeResource`)
- **Tipos de Nós:** `Composite` (controle de fluxo), `Decorator` (condições), `Leaf` (ações)

**Integração com a Biblioteca:**

- **Assets:** Busca rápida de `BehaviorTree`, `RLConfig` e `MachiBrain` com drag & drop para composição.
- **Workbench:** Ajuste fino de parâmetros de nós sem abrir o editor completo.
- **CraftTable:** Templates de comportamentos comuns (Patrulha, Combate, Fuga) para início rápido.

### Perfis Dinâmicos

| Perfil      | Cérebro              | Exemplo   |
| :---------- | :------------------- | :-------- |
| **Critter** | BT Simples           | Pássaros. |
| **NPC**     | BT + Gemini Assets   | Aldeões.  |
| **Boss**    | BT + RL (Neural Net) | Chefes.   |

---

## Director

> O Maestro da Narrativa
>
> **Sequencer & Cutscenes:** Uma ferramenta de timeline unificada para orquestrar momentos cinematográficos.
>
> **Filosofia (Sequencer):** Inspirado no Unreal Sequencer. Manipula estado de jogo (Câmera, Som, TimeScale) em tempo real, sem "baking".

O `Director` orquestra os outros packages via **Semantic Tracks**.

**Server:**

- **DirectorServer:** Singleton gestor de estado (Bloqueia inputs em cutscene). Não é um Node.

**Nodes:**

- **SequencePlayer:** O executor da timeline na cena.
- **CinemaZone:** Trigger de início de cutscene.

**Features:**

- **Osmo Track:** Controla cortes de câmera e vCams.
- **Sound Track:** Dispara SoundCues com prioridade.
- **Synapse Track:** Emite estímulos de IA sincronizados.
- **State Restore:** Restaura o estado anterior do jogo ao fim da cutscene.

**Resources:**

- **Cutscene:** O asset da timeline.
- **ActionClip:** Bloco de ação para NPCs.

**Integração com o Sonhar:**

O Director se integra à **Biblioteca do Sonhar** para gerenciamento e edição rápida de Resources.

**Integração com a Biblioteca:**

- **Assets:** Busca e filtro de `Cutscene` e `ActionClip` com preview de duração e tracks.
- **Workbench:** Edição rápida de timings, transições e eventos sem abrir o sequencer completo.
- **CraftTable:** Wizards de criação de cutscenes com templates (Diálogo, Ação, Revelação).

---

## Gaia

> A Atmosfera & o Tempo
>
> **Interface Jogo -> Jogador:** Gaia não apenas simula o clima, mas dita o **"Mood"** do jogo. É a camada final de imersão que conecta o jogador emocionalmente ao mundo virtual.

Simulação de Ambiente Sistêmico. Mais do que chuva visual, Gaia gerencia estados globais que afetam gameplay, física e shaders.

### Arquitetura: Clima Sistêmico

Gaia opera em **Estados Globais** que são propagados para todos os materiais e entidades do jogo.

**1. Global Wetness & Wind**

O clima não é local. É uma variável global.

- **Wetness (0.0 - 1.0):** Controla o quanto o mundo está molhado.
  - **Shaders:** Aumenta roughness, escurece albedo (via `global_shader_parameter`).
  - **Gameplay:** Aumenta chances de escorregar (Physics Materials), apaga fogo.
- **Wind Vector:** Vetor 3D global de vento.
  - Afeta partículas (chuva, folhas), balística de projéteis e movimentação de vegetação.

**2. Temperatura & Sobrevivência**

Gaia rastreia a temperatura ambiente baseada na Hora, Estação e Clima.

- **Gameplay Tag:** O jogador recebe Tags como `Status.Cold` ou `Status.Hot`.
- **Animação:** Procedural blending para animações de frio (tremer) ou calor (limpar suor).

**3. Biomas & Zonas**

O mundo é dividido em Biomas (Deserto, Tundra, Floresta).

- **Weather Weights:** Um deserto tem 99% de chance de Sol e 1% de Chuva. Uma Tundra tem 50% de Neve.
- **Temperature Offset:** O bioma define a base térmica (ex: Deserto +20ºC durante o dia, -10ºC à noite).
- **Audio Ambience:** O bioma dita o som de fundo (vento, fauna) em integração com o `Sounds`.

**4. Ciclo Dia/Noite**

Simulação astronômica simplificada.

- **Sun & Moon:** Controle orbital de luzes direcionais.
- **Curve-Driven:** Cor da luz, Neblina e Densidade de Nuvens são controlados por curvas (Resources), permitindo dias de "Inverno Pálido" ou "Verão Dourado".

**Servers:**

- `GaiaServer` (C++): Gerencia o loop de simulação e update de parâmetros globais de shader.

**Nodes:**

- `DayNightCycle` / `DayNightCycle2D`: Controlador de tempo (Suporte a 2D e 3D).
- `WeatherController`: Gerencia transições de clima (ex: Ensolarado -> Tempestade leva 30 segundos).
- `GaiaZone`: Área override (ex: Entrar numa caverna para a chuva/vento locais).

**Resources:**

- **BiomeDef:** Definição de tabelas de clima e offsets de temperatura.
- `WeatherDef`: Define um tipo de clima (Chuva, Neve, Tempestade de Areia).
- `SeasonDef`: Define durações de dias e curvas de temperatura.

### Integração de Gameplay (Ability System)

Gaia não é apenas visual. Ele conversa diretamente com o `AbilitySystem` via **Tags**.

1. **Tag Emitter:**

- Se chover, Gaia adiciona a tag `Env.Weather.Rain` ao Contexto Global.
- Se estiver frio, adiciona `Env.Temperature.Cold`.
- Se estiver no d
