# Portal do Aluno: Machi Class

Este documento delineia a trilha de aprendizagem completa do curso, focando na transformação de desenvolvedores em arquitetos.

## Sumário

- [Módulo 00: Fundamentos da Arquitetura](./00-Fundamentos/README.md)
- [Módulo 01: A Tríade Arcade (Snake, Pong, Pacman)](./01-TriadeArcade/README.md)
- [Módulo 02: Arquitetura de Entidades (Topdown Shooter)](./02-TopdownShooter/README.md)
- [Módulo 03: Sistemas de Dados e UI (RPG Tático)](./03-RPGTatico/README.md)
- [Módulo 04: Física Avançada e Estados (Metroidvania)](./04-Metroidvania/README.md)
- [Módulo 05: Procedural Generation & Tilemaps (Roguelike)](./05-Roguelike/README.md)
- [Módulo 06: 3D Fundamentals (FPS Retro)](./06-FPSRetro/README.md)
- [Módulo 07: Inteligência Artificial (Stealth Game)](./07-Stealth/README.md)
- [Módulo 08: Networking & Multiplayer (Arena)](./08-Multiplayer/README.md)
- [Módulo 09: Shaders & VFX (Juice)](./09-VFX/README.md)
- [Módulo 10: Plugins & Tooling (Extensibilidade Nativa)](./10-Plugins/README.md)
- [Módulo 11: TCC (Projeto Final)](./11-TCC/README.md)
- [Bônus: Rust & GDExtension (Performance Extrema)](./Bonus-Rust_GDExtension/README.md)
- [Bônus: DevOps & CI/CD (Automação de Builds)](./Bonus-DevOps_CICD/README.md)
- [Bônus: Arquitetura de Modding & DLCs](./Bonus-Modding_DLCs/README.md)
- [Bônus: Matemática para Engenheiros de Jogos](./Bonus-Math_Engineers/README.md)

---

## 🟢 [Módulo 00: Fundamentos da Arquitetura](./00-Fundamentos/README.md)

- **Objetivo:** Estabelecer o "Mindset do Engenheiro". Antes de criar jogos, aprendemos a criar sistemas. Entender profundamente como a Godot pensa (Nodes, Resources, Scenes) para não lutar contra a engine.

- **Aulas:**

  - **0.1: Fundamentos da Arquitetura**
    - **Conceito Central:** A trindade da Godot: Nodes (Comportamento), Resources (Dados) e Scenes (Agrupamento).
    - **Tópicos:** SceneTree, Instanciação e a filosofia de "Tudo é uma Cena".

  - **0.2: GDScript Fundamentals**
    - **Conceito Central:** GDScript como ferramenta de engenharia, não apenas script.
    - **Tópicos:** Tipagem Estática, Collections (Array/Dictionary) e Loops eficientes.

  - **0.3: O Game Loop e Ciclo de Vida**
    - **Conceito Central:** Entender quando as coisas acontecem para evitar bugs de ordem de execução.
    - **Tópicos:** `_ready`, `_process` (Visual), `_physics_process` (Físico) e `queue_free`.

  - **0.4: Sistema de Input**
    - **Conceito Central:** Abstração de controle. O jogo ouve "Ações", não teclas.
    - **Tópicos:** InputMap, Eventos (`_unhandled_input`) vs Polling (`Input.is_action_pressed`).

  - **0.5: Física e Colisão**
    - **Conceito Central:** A interação espacial entre objetos.
    - **Tópicos:** CharacterBody vs RigidBody vs Area2D. O sistema de Layers e Masks.

  - **0.6: Resource-Oriented Programming (ROP)**
    - **Conceito Central:** A arquitetura assinatura do Machi Class. Separação total de Dados e Lógica.
    - **Tópicos:** Criando Custom Resources (`class_name ... extends Resource`) e editando no Inspector.

  - **0.7: Composição vs Herança**
    - **Conceito Central:** Evitando a "God Class". Construindo comportamentos através de nós filhos (Componentes).
    - **Tópicos:** Criando um `HealthComponent` ou `HitboxComponent` reutilizável.

  - **0.8: UI e Containers**
    - **Conceito Central:** Interfaces responsivas que funcionam em qualquer tela.
    - **Tópicos:** O sistema de Containers (`HBox`, `VBox`, `Grid`) e Âncoras.

  - **0.9: Debugging e Profiling**
    - **Conceito Central:** Como encontrar e matar bugs profissionalmente.
    - **Tópicos:** Breakpoints, Remote Scene Tree e o Profiler de performance.

  - **0.10: Internacionalização (i18n)**
    - **Conceito Central:** Preparando o jogo para o mundo desde o dia 1.
    - **Tópicos:** Sistema de Locales, arquivos CSV/PO e remap de assets.

---

## 🟡 Módulo 01: A Tríade Arcade (Snake, Pong, Pacman)

- **Objetivo:** Construir três clássicos para dominar os fundametos da lógica de programação aplicada a jogos. O foco é entender lógica de Grid (Snake), Física (Pong) e Inteligência Artificial Básica (Pacman) em escopos fechados.

- **Aulas:**

  - **1.1: Snake: Estruturas de Dados e Grid**

    - **Conceito Central:** Gerenciamento de lógica independente de física. Uso de estruturas de dados (Array) para representar o corpo da cobra e manipulação de coordenadas discretas (Grid) versus coordenadas de tela.
    - **Tópicos a Cobrir:**
      - Timers para o "game tick" e movimento discreto.
      - Manipulação de `Arrays` (`push_front`, `pop_back`).
      - Filas (Queue) para o corpo da cobra.
    - **Objetivo de Aprendizagem:** Entender como separar a lógica do jogo da sua representação visual.
    - **Exercício Prático Sugerido:** Implementar o movimento da cobra usando um Array de Vector2, atualizando a posição a cada tick do Timer.

  - **1.2: Pong: O Motor de Física e Sinais**

    - **Conceito Central:** Introdução ao motor de física (`Area2D` vs `StaticBody` vs `CharacterBody`) e arquitetura baseada em eventos (Sinais).
    - **Tópicos a Cobrir:**
      - `Vector2` para direção, velocidade e reflexão (`bounce`).
      - Detecção de colisão e resposta física.
      - O Padrão Observer: Usando Sinais para pontuação e UI.
    - **Objetivo de Aprendizagem:** Dominar a interação física básica e a comunicação desacoplada entre objetos.
    - **Exercício Prático Sugerido:** Criar uma bola que quica e emite um sinal `goal` ao sair da tela, atualizando um placar na UI.

  - **1.3: Pacman: Navegação e Máquinas de Estado**
    - **Conceito Central:** Inteligência Artificial básica e navegação em labirintos usando TileMaps.
    - **Tópicos a Cobrir:**
      - `TileMap`: Desenhando o labirinto e definindo paredes.
      - Máquina de Estados Simples: Fantasmas com comportamentos `Chase` (Perseguir) e `Flee` (Fugir).
      - Singletons para gerenciamento global do jogo.
    - **Objetivo de Aprendizagem:** Criar agentes autônomos que reagem ao estado do jogo e navegam no ambiente.
    - **Exercício Prático Sugerido:** Criar um fantasma que alterna entre perseguir o jogador e fugir quando um "power pellet" é ativado.

---

## 🟠 Módulo 02: Arquitetura de Entidades (Topdown Shooter)

- **Objetivo:** Criar um jogo de ação expansível, introduzindo a arquitetura profissional de entidades. Foco no "Machi Way": separação estrita entre Dados (Resources) e Comportamento (Nodes), além de herança, polimorfismo e otimização de memória.

- **Aulas:**

  - **2.1: Inimigos: Herança e Polimorfismo**

    - **Conceito Central:** Usar Herança (`extends`) para compartilhar lógica comum (Vida, Hitbox) e Polimorfismo para variar comportamentos específicos sem duplicar código.
    - **Tópicos a Cobrir:**
      - Classe Base `Enemy` (Gerenciamento de HP, Morte e Visual).
      - Especializações: `MeleeEnemy` (Comportamento de perseguição) e `RangedEnemy` (Comportamento de tiro à distância).
    - **Objetivo de Aprendizagem:** Criar uma hierarquia de classes escalável onde novos inimigos podem ser adicionados facilmente herdando da base.
    - **Exercício Prático Sugerido:** Criar um inimigo base e duas variações que sobrescrevem o método `_attack()`.

  - **2.2: Sistemas de Spawn e Object Pooling**

    - **Conceito Central:** Instanciar e destruir objetos (`instantiate`/`queue_free`) é custoso para a CPU. O Object Pooling é uma técnica de otimização onde objetos são reciclados em vez de destruídos.
    - **Tópicos a Cobrir:**
      - Criando um Spawner configurável com Timers e Waves.
      - Por que instanciar/deletar é lento?
      - Implementando um Pool de Projéteis para performance máxima em jogos de tiro intenso ("Bullet Hell").
    - **Objetivo de Aprendizagem:** Aprender a gerenciar a memória do jogo eficientemente, mantendo a performance estável mesmo com centenas de objetos na tela.
    - **Exercício Prático Sugerido:** Implementar um benchmark comparando FPS com e sem Object Pooling ao spawnar 1000 balas.

  - **2.3: Componentização e Armas**
    - **Conceito Central:** Armas e habilidades podem ser tratadas como componentes acopláveis, permitindo que tanto o Player quanto os Inimigos compartilhem a mesma lógica de combate.
    - **Tópicos a Cobrir:**
      - Criando um `WeaponComponent` reutilizável.
      - Uso de `Composition` para adicionar capacidade de tiro a qualquer entidade.
    - **Objetivo de Aprendizagem:** Desacoplar a lógica de combate da entidade, permitindo maior flexibilidade na criação de personagens.
    - **Exercício Prático Sugerido:** Adicionar o `WeaponComponent` tanto ao Player quanto a uma Torre defensiva estática.

---

## 🔴 Módulo 03: Sistemas de Dados e UI (RPG Tático)

- **Objetivo:** Sair da ação em tempo real e focar em sistemas complexos e arquitetura de dados robusta. O desafio é arquitetural: como gerenciar inventários, skills, saves e interfaces ricas usando Resource-Oriented Programming (ROP).

- **Aulas:**

  - **3.1: Custom Resources Avançados**

    - **Conceito Central:** Aprofundamento no sistema de Resources da Godot para criar estruturas de dados complexas e aninhadas, fundamentais para RPGs.
    - **Tópicos a Cobrir:**
      - Definindo `ItemData`, `SkillData`, e `CharacterSheet`.
      - Resources referenciando outros Resources (ex: um Item que tem uma Skill associada).
      - Editando dados complexos diretamente no Inspector.
    - **Objetivo de Aprendizagem:** Modelar todo o banco de dados do jogo usando apenas arquivos `.tres`, sem banco de dados externo.
    - **Exercício Prático Sugerido:** Criar uma árvore de talentos onde cada nodo é um Resource que desbloqueia outros.

  - **3.2: Sistema de Inventário Desacoplado**

    - **Conceito Central:** Separação total entre os Dados Lógicos (`InventoryComponent` - o que tenho na mochila) e a Representação Visual (`InventoryUI` - o que vejo na tela).
    - **Tópicos a Cobrir:**
      - Array de Resources como estrutura de dados da mochila.
      - Sinais para comunicar mudanças de inventário para a UI.
      - Implementação de Drag and Drop na interface.
    - **Objetivo de Aprendizagem:** Criar um sistema de inventário onde a lógica funciona independentemente de existir uma interface gráfica.
    - **Exercício Prático Sugerido:** Implementar uma troca de itens entre dois inventários (mochila e baú) apenas manipulando arrays de dados.

  - **3.3: Arquitetura de UI e Persistência**
    - **Conceito Central:** Organização de interfaces complexas usando o padrão MVC (Model-View-Controller) adaptado e persistência de dados em disco.
    - **Tópicos a Cobrir:**
      - Temas (`Theme`) globais para consistência visual.
      - Serializando Resources para disco (Save Game).
      - Carregando e aplicando o estado salvo ao mundo.
    - **Objetivo de Aprendizagem:** Construir interfaces profissionais e implementar um sistema de Save/Load robusto.
    - **Exercício Prático Sugerido:** Criar um botão "Salvar" que escreve o inventário atual em disco e um botão "Carregar" que restaura o estado.

---

## 🟣 [Módulo 04: Física Avançada e Estados (Metroidvania)](./04-Metroidvania/README.md)

_Controle preciso (Game Feel) e Máquinas de Estado Robustas._

---

## 🔵 [Módulo 05: Procedural Generation & Tilemaps (Roguelike)](./05-Roguelike/README.md)

_Algoritmos, Ferramentas de Editor e Rejogabilidade._

---

## 🟢 [Módulo 06: 3D Fundamentals (FPS Retro)](./06-FPSRetro/README.md)

_A transição para o eixo Z e o mundo 3D._

---

## 🟤 [Módulo 07: Inteligência Artificial (Stealth Game)](./07-Stealth/README.md)

_Comportamento Autônomo e Sistemas de Detecção._

---

## ⚫ [Módulo 08: Networking & Multiplayer (Arena)](./08-Multiplayer/README.md)

_Conectividade, Sincronização e Autoridade._

---

## ⚪ [Módulo 09: Shaders & VFX (Juice)](./09-VFX/README.md)

_Tech Art, Polimento e "Game Feel"._

---

## 🎓 [Módulo 10: Plugins & Tooling (Extensibilidade Nativa)](./10-Plugins/README.md)

_Criando suas próprias ferramentas usando GDScript e GDShaders._

---

## 🎓 [Módulo 11: TCC (Projeto Final)](./11-TCC/README.md)

_O Ciclo Completo de Produção e Lançamento._

---

## 🦀 [Bônus: Rust & GDExtension (Performance Extrema)](./Bonus-Rust_GDExtension/README.md)

_Segurança de Memória, Performance Nativa e Integração de Baixo Nível._

---

## 🏗️ [Bônus: DevOps & CI/CD (Automação de Builds)](./Bonus-DevOps_CICD/README.md)

_Automação de Builds, Exportação e Entrega Contínua._

---

## 🧩 [Bônus: Arquitetura de Modding & DLCs](./Bonus-Modding_DLCs/README.md)

_Extensibilidade do Produto Final e Carregamento Dinâmico._

---

## 📐 [Bônus: Matemática para Engenheiros de Jogos](./Bonus-Math_Engineers/README.md)

_Vetores, Matrizes, Quaternions e o que acontece "debaixo do capô"._

---

> **Nota:** Para conceitos avançados de arquitetura, consulte a seção de **Pós-Graduação** na raiz do repositório.
