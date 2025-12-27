# Guia de Planejamento de Jogos e Sistemas

Este guia foca em transformar uma ideia em um plano de execução concreto. Abrange o planejamento de alto nível (o jogo como um todo) e o de baixo nível (os sistemas e plugins que o compõem), sempre com a arquitetura Zyris em mente.

---

## Parte 1: Planejamento de Alto Nível (O Jogo)

Antes de escrever uma linha de código, um bom planejamento evita meses de trabalho desperdiçado.

### 1.1. O Documento de Design de Jogo (Game Design Document - GDD)

O GDD é a "bíblia" do projeto. É um documento vivo que descreve todos os aspectos do jogo.

**Seções Essenciais de um GDD:**

1. **Visão Geral (One-Page Pitch):**

   - **Conceito Central:** Uma única frase que resume o jogo. (Ex: "Um jogo de sobrevivência e criação em um mundo pós-apocalíptico subaquático.").
   - **Público-Alvo:** Para quem é este jogo? (Ex: "Fãs de jogos de sobrevivência e exploração, como Subnautica e The Long Dark.").
   - **Diferenciais (USP - Unique Selling Points):** O que torna seu jogo único? (Ex: "Sistema de criação baseado em pressão, ciclo dia/noite que afeta a fauna abissal.").
   - **Plataforma(s):** PC, Console, Mobile?

2. **Mecânicas de Jogo (Core Gameplay Loop):**

   - **Loop Principal:** O ciclo de ações que o jogador repetirá. (Ex: Explorar -> Coletar Recursos -> Criar Ferramentas -> Explorar áreas mais profundas -> Repetir).
   - **Mecânicas Detalhadas:** Descreva cada ação do jogador. (Pular, nadar, construir, lutar, etc.).
   - **Sistemas Principais:** Liste e descreva os sistemas que suportam as mecânicas (Sistema de Inventário, Sistema de Habilidades, IA dos Inimigos, etc.).

3. **Mundo e Narrativa:**

   - **História:** Qual é o enredo? Quem é o protagonista?
   - **Universo:** Descreva o mundo, suas regras, facções e locais de interesse.
   - **Tom e Atmosfera:** O jogo é sombrio, cômico, sério?

4. **Arte e Áudio:**

   - **Direção de Arte:** Estilo visual (realista, cartunesco, pixel art), paleta de cores, referências visuais.
   - **Direção de Áudio:** Estilo da trilha sonora, tipos de efeitos sonoros.

5. **Design de Interface (UI/UX):**
   - **Fluxo de Telas:** Esboce a navegação entre menus (Menu Principal -> Jogo -> Menu de Pause -> Inventário).
   - **HUD (Heads-Up Display):** O que o jogador vê na tela durante o jogo? (Vida, mapa, etc.).

### 1.2. O "Vertical Slice": A Prova dos Nove

Um _Vertical Slice_ (Fatia Vertical) não é uma demo. É uma porção pequena e **altamente polida** do jogo que contém exemplos de **todas** as suas funcionalidades principais.

- **Propósito:** Provar que o _Core Gameplay Loop_ é divertido e que a tecnologia e a arte funcionam juntas. É uma ferramenta para validar a visão do projeto e atrair investimento.
- **Exemplo para nosso jogo subaquático:** Uma única área explorável que inclua: 1-2 tipos de inimigos, 3-4 recursos para coletar, 2-3 itens para criar, e um pequeno objetivo de quest. Tudo com a qualidade visual e de áudio final.

---

## Parte 2: Planejamento de Baixo Nível (Sistemas e Plugins)

Aqui aplicamos os princípios do Zyris para projetar sistemas modulares e escaláveis.

### 2.1. Princípios de Design de Sistemas

1. **Modularidade:** O sistema deve funcionar de forma independente. Um `Sistema de Inventário` não deve depender diretamente de um `Sistema de Quests`. Eles se comunicam através de sinais ou `Servers`, mas não são acoplados.
2. **Escalabilidade:** O sistema deve suportar o crescimento do jogo. Um sistema de combate deve funcionar bem com 10 habilidades ou com 100.
3. **Abstração:** Exponha uma API simples e clara. O resto do jogo não precisa saber _como_ o `InventoryServer` organiza os itens internamente, apenas como `adicionar_item()` e `remover_item()`.
4. **Orientação a Dados (Zyris):** Separe a lógica dos dados. A lógica de "o que acontece quando uso esta poção" deve estar no `Resource` da poção, não codificada no `InventorySystem`.

### 2.2. Workflow de Planejamento de um Novo Sistema (Ex: Sistema de Reputação)

Vamos planejar um sistema de reputação para o nosso jogo, onde as ações do jogador afetam sua relação com diferentes facções (ex: "Cientistas da Cúpula", "Rebeldes do Abismo").

**Passo 1: Definir os Requisitos**

- A reputação é um valor numérico (ex: de -100 a 100).
- Existem múltiplas facções. O jogador tem um valor de reputação para cada uma.
- Certas ações (completar quests, atacar membros) modificam a reputação.
- A reputação deve ser persistente (salva no save do jogo).
- NPCs no mundo devem reagir visualmente à reputação do jogador (ex: um ícone sobre a cabeça).
- Diálogos e preços de lojas podem mudar com base na reputação.

**Passo 2: Aplicar a Arquitetura Zyris**

Como vamos dividir as responsabilidades entre `Server`, `Resource` e `Node`?

1. **`ReputationServer` (Server - C++):**

   - **Propósito:** O cérebro global e autoritativo do sistema.
   - **Responsabilidades:**
     - Armazenar o estado atual da reputação do jogador com todas as facções (ex: `std::map<FactionID, int>`).
     - Fornecer a API principal do sistema: `adjust_reputation(faction, amount)`, `get_reputation(faction)`.
     - Lidar com a serialização (salvar/carregar) dos dados de reputação.
     - Emitir um sinal global quando uma reputação muda (ex: `reputation_changed(faction, new_value)`).

2. **`ReputationModifier` (Resource):**

   - **Propósito:** Um pacote de dados que descreve uma mudança de reputação.
   - **Responsabilidades:**
     - Conter os dados da modificação: `@export var faccao: FactionEnum`, `@export var quantidade: int`.
     - Pode ser anexado a recompensas de quests, a itens, ou a eventos de diálogo.
   - **Exemplo:** Uma `Quest` (outro `Resource`) teria uma lista de `ReputationModifier` em suas recompensas. Ao completar a quest, o `QuestServer` lê esses modificadores e chama `ReputationServer.adjust_reputation()` para cada um.

3. **`ReputationObserver` (Node - C++ ou GDScript):**
   - **Propósito:** A manifestação do sistema na cena.
   - **Responsabilidades:**
     - Ser adicionado a um NPC no mundo do jogo.
     - No `_ready()`, conectar-se ao sinal `reputation_changed` do `ReputationServer`.
     - Quando o sinal é recebido, ele verifica se a facção que mudou é a sua.
     - Se for, ele atualiza sua aparência (ex: mostra um ícone de "feliz" ou "irritado") ou altera a árvore de diálogo que ele usará.

**Passo 3: Desenhar o Fluxo de Dados**

1. O jogador completa uma quest.
2. O `QuestSystem` (Node ou Server) lê o `Quest` (Resource) e encontra um `ReputationModifier` (Resource) nas recompensas.
3. O `QuestSystem` chama `ReputationServer.adjust_reputation("RebeldesDoAbismo", 20)`.
4. O `ReputationServer` atualiza seu mapa interno e emite o sinal `reputation_changed("RebeldesDoAbismo", 80)`.
5. Todos os `ReputationObserver` (Nodes) na cena recebem o sinal. O observador em um NPC Rebelde vê que a reputação de sua facção mudou e atualiza seu ícone para um rosto sorridente.

Este método de planejamento resulta em um sistema **desacoplado**: o `QuestSystem` não precisa saber nada sobre o `ReputationSystem` além da existência do `Server`. Os NPCs não gerenciam estado, apenas reagem a mudanças. E os designers podem criar novas quests e modificadores de reputação inteiramente no editor, sem escrever código.
