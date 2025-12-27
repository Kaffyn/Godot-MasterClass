# Princípios Arquiteturais do Framework Zyris (Refinado)

Este documento refina e expande os princípios arquiteturais apresentados no `zyris.md`, servindo como a base de conhecimento para o desenvolvimento e treinamento da IA assistente.

## A Tríade Arquitetural: Server, Resource, Node

A escalabilidade e performance do Zyris vêm de uma divisão de responsabilidades estrita entre três tipos de elementos.

### 1. Servers: A Verdade Global e Persistente

- **Extende:** `Object` (geralmente)
- **Propósito:** Gerenciar o estado **global, persistente e autoritativo**. São os "deuses" do framework, operando em um nível acima de qualquer cena ou objeto.
- **Características:**
  - **Singleton:** Existe uma única instância de cada `Server` durante toda a execução do jogo (ex: `GaiaServer`, `InventoryServer`).
  - **Fora da SceneTree:** Não são `Nodes`. Isso os protege de serem destruídos em trocas de cena e remove o _overhead_ do processamento da árvore de cena.
  - **Lógica Pesada:** Escritos em C++, são ideais para executar algoritmos complexos, gerenciar _pools_ de memória e processar grandes volumes de dados (ex: `SynapseServer` para física, `SoundServer` para mixagem de áudio).
- **Analogia:** Um `Server` é como o servidor de um MMO. Ele detém o estado verdadeiro do mundo, e os jogadores (Nodes) apenas enviam requisições e recebem atualizações.

### 2. Resources: O DNA do Gameplay (Dados e Lógica Pura)

- **Extende:** `Resource`
- **Propósito:** Definir **"o quê"** e **"como"** de forma passiva, portátil e orientada a dados. São o DNA que descreve cada elemento do jogo.
- **Características:**
  - **Data-Driven:** Contêm principalmente dados (`@export var dano: int`), mas também podem conter lógica pura e sem estado (funções que calculam algo com base nos dados que eles mesmos contêm).
  - **Reutilizáveis:** Um mesmo `Resource` de uma "Poção de Cura" pode ser usado por centenas de itens no mundo do jogo. Alterar o `Resource` altera o comportamento de todos eles.
  - **Editáveis:** A principal vantagem é serem editáveis diretamente no Inspetor da Godot. Isso permite que _designers_ criem e modifiquem o gameplay sem tocar no código C++.
  - **Agnósticos à Cena:** Um `Resource` não sabe em que cena está ou a qual `Node` pertence. Ele é uma unidade de informação autocontida.
- **Analogia:** Um `Resource` é como uma carta em um jogo de cartas colecionáveis (TCG). A carta descreve o que a criatura faz, seu poder, seu custo. Ela não _faz_ nada por si só, apenas contém as regras.

### 3. Nodes: Os Atores em Cena (Orquestradores e Visualizadores)

- **Extende:** `Node`, `Node3D`, etc.
- **Propósito:** Atuar como a **manifestação de um sistema ou objeto dentro de uma cena específica**. São os "atores" que usam o "roteiro" (Resources) no "palco" (SceneTree).
- **Dois Papéis Principais:**

  1. **Visualizadores Puros:** Este é o papel mais simples. Um `Sprite2D` que mostra um ícone de item, ou um `MeshInstance3D` que representa um personagem. Eles leem dados de `Resources` ou `Servers` e os exibem. A regra de ouro do Zyris é que, sempre que possível, os `Nodes` devem se limitar a este papel.

  2. **Controladores/Motores de Instância:** Este é o papel mais complexo e crucial, e a razão pela qual `Nodes` são mais do que meros visualizadores. Um `Node` customizado como `AbilitySystem` ou `BehaviorTreePlayer` age como o **orquestrador de uma instância de um sistema**.
     - **Gestão de Estado de Cena:** O `Node` `AbilitySystem` em um inimigo gerencia os _cooldowns_ das habilidades _daquele_ inimigo. O `Node` `BehaviorTreePlayer` em outro inimigo gerencia qual nó da árvore está `RUNNING` _para ele_. Esse estado é local à cena e não precisa de persistência global.
     - **Ponte para a Engine:** São o ponto de entrada para o `_process` e `_physics_process`. O `Node` `AbilitySystem` usa esses _callbacks_ para atualizar seus _cooldowns_ internos.
     - **Executor da Lógica dos Resources:** O `Node` carrega o `Resource` (a "carta") e é responsável por "jogá-lo", ou seja, executar a lógica descrita nele no momento certo. O `Node` `AbilitySystem` carrega um `Skill` (Resource) e executa suas fases (Custo, Execução, etc.).

- **Analogia:** Um `Node` é o jogador que segura a carta (Resource) e a coloca na mesa, pagando seu custo e ativando seu efeito no jogo. O `Node` `AbilitySystem` é o jogador, e os `Skill` Resources são as cartas em sua mão.

---

## Fluxo de Interação: Exemplo Prático

1. **O Inimigo (`Inimigo.tscn`):** Um `CharacterBody3D` que contém, entre outros, um **`BehaviorTreePlayer` (Node)** e um **`AbilitySystem` (Node)**.
2. **A Lógica (Resources):** O `BehaviorTreePlayer` tem um `BehaviorTree` (Resource) assignado a ele, que dita sua estratégia. O `AbilitySystem` tem acesso aos `Skill` e `State` (Resources) do personagem.
3. **A Decisão (Node -> Node):** A cada _tick_, o `BehaviorTreePlayer` (Node) executa sua árvore (`Resource`). A árvore decide: "é hora de atacar". O nó de ação da árvore não ataca, ele chama uma função no `Node` irmão, o `AbilitySystem`.
4. **A Execução (Node -> Resource -> Node):** O `AbilitySystem` (Node) recebe o comando. Ele consulta seus `Skill` (Resources) de ataque, escolhe o melhor com base no contexto, paga o custo, e aplica o efeito (outro `Resource`) no alvo. O `AbilitySystem` (Node) então gerencia o _cooldown_ daquela habilidade.
5. **O Estado Global (Node -> Server):** Se o ataque mata o jogador, o `AbilitySystem` (Node) pode notificar um `GameStateServer` (Server) de que o jogo terminou.

Esta arquitetura permite um desacoplamento poderoso, onde designers podem criar lógicas de IA (`BehaviorTree` Resource) e habilidades (`Skill` Resource) visualmente, enquanto os programadores criam os `Nodes` (motores C++) e `Servers` (sistemas globais C++) de alta performance que executam essas lógicas no jogo.
