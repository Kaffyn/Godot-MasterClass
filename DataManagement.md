# Arquitetura de Dados: A Espinha Dorsal do seu Jogo

> **De:** Machi
> **Para:** Você, que está prestes a parar de tratar dados como um detalhe.
>
> A lógica do seu jogo é o músculo. Os gráficos são a pele. Mas os dados são a espinha dorsal. Uma espinha dorsal fraca ou malformada e todo o corpo desmorona no primeiro sinal de estresse. Aprenda a arquitetar seus dados e você construirá sistemas que não apenas funcionam, mas que duram e escalam.

## 1. A Separação Sagrada de Dados e Lógica

Um dos saltos mais importantes para um desenvolvedor amadurecer é entender a **separação entre dados e lógica**.

- **Dados** são a informação, o "quê". _Exemplo: A vida máxima de um personagem é 100, sua velocidade é 50, o dano da sua espada é 15._
- **Lógica** é o comportamento, o "como". _Exemplo: A função `take_damage(amount)` que subtrai vida, o código que move o `CharacterBody2D`._

Um projeto onde os dados estão misturados com a lógica (ex: `var max_hp = 100` dentro do `player.gd`) é um pesadelo de manutenção. Se um game designer quiser balancear o jogo e mudar o HP de 100 para 120, ele precisará editar um script, com grande risco de quebrar a lógica.

A arquitetura profissional isola os dados em arquivos próprios, permitindo que a equipe de design itere e balanceie o jogo sem precisar de um programador.

## 2. O Arsenal de Dados da Godot: Um Guia Comparativo

A Godot oferece múltiplas ferramentas para gerenciar dados. Saber quando usar cada uma é crucial.

| Ferramenta              | O que é?                                                  | Quando Usar (O Jeito Machi)                                                                                                                                               | Quando NÃO Usar                                                                                                       |
| :---------------------- | :-------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------- |
| **Resource (`.tres`)**  | Container de dados nativo, tipado e visível no Inspector. | **A ferramenta padrão para 90% dos casos.** Definição de itens, stats de inimigos, skills, configurações de níveis. O coração da Programação Orientada a Resources (ROP). | Dados de runtime que mudam a todo instante (use Dicionários). Dados para ferramentas externas (use JSON).             |
| **ConfigFile (`.cfg`)** | Arquivo de texto simples no formato INI (chave=valor).    | **Configurações do usuário.** Volume de áudio, qualidade gráfica, mapeamento de teclas. É simples e legível por humanos.                                                  | Estruturas de dados complexas ou aninhadas. Definição de conteúdo de jogo (use Resources).                            |
| **JSON (`.json`)**      | Formato de texto universal para troca de dados.           | **Comunicação externa.** APIs web, importação/exportação de dados com ferramentas que não são a Godot (ex: uma planilha de diálogos, um editor de níveis externo).        | Dados que só a Godot usa. JSON não entende tipos nativos como `Vector2`, `Color` ou referências a outros `Resource`s. |
| **Dictionary**          | Estrutura chave-valor dinâmica, em memória.               | **Estado de Runtime.** O inventário _atual_ do jogador, quests ativas, estado do mundo. É o formato perfeito para ser serializado em um arquivo de save.                  | Para definir conteúdo de jogo (falta de tipagem, não é amigável para designers no Inspector).                         |
| **Array**               | Lista ordenada de itens, em memória.                      | Quando a **ordem importa**. Caminhos de patrulha, sequências de diálogo, um baralho de cartas. Use `Array[Tipo]` para segurança.                                          | Quando você precisa de acesso rápido por uma chave única (use Dictionary).                                            |

## 3. Uma Analogia de Banco de Dados: Resource (Schema) vs. JSON (Tabela)

Para solidificar o entendimento, vamos usar uma analogia do mundo de banco de dados.

Pense em um `JSON` ou um `Dictionary` como uma **tabela de dados** ou uma planilha. Ele é excelente para armazenar e transportar linhas e colunas de informação de forma universal. Qual o propósito original do JSON? Estruturar dados para JavaScript. Sua função é ser facilmente convertido em um objeto genérico em outra linguagem. Ele é um formato de intercâmbio, um "passaporte de dados".

Agora, pense em um `Resource` da Godot como um **Schema de Banco de Dados** completo. Ele é arquiteturalmente superior por várias razões:

1.  **Ele Contém as Tabelas**: Um `Resource` pode, obviamente, conter todos os dados que uma tabela/JSON conteria.
2.  **Ele Define as Regras entre Elas**: Mais importante, ele define a **estrutura**, os **tipos de dados** (`int`, `String`, `Vector2`), as **regras de validação** (via setters/getters) e os **relacionamentos** entre diferentes "tabelas" (um `Resource` contendo referências a outros `Resource`s).
3.  **Ele é um Objeto Inteligente, não um Saco de Dados**: Quando a Godot carrega um `JSON`, você recebe um `Dictionary` genérico. Quando a Godot carrega um `.tres`, ela instancia um **objeto de uma classe específica**, com seus próprios métodos, herança e funcionalidades. Ele não é apenas "parseado", ele é "instanciado".

Enquanto o `JSON` foi criado para ser transformado em objetos JavaScript genéricos, o `Resource` foi criado para ser a espinha dorsal de um ecossistema de objetos de jogo ricos e interconectados, em um padrão de excelência muito superior para o design de games.

## 4. O Padrão Ouro: ROP (Resource-Oriented Programming) em Ação

A filosofia do Godot MBA e da SoftEngine é clara: **use `Resource`s para tudo que for conteúdo de design**. Isso significa que, em vez de apenas guardar dados, os `Resource`s se tornam blocos de construção.

O poder real vem da **composição**: `Resource`s que contêm outros `Resource`s.

**Exemplo Prático:**

1.  Você cria um `WeaponData.gd` que herda de `Resource`. Ele tem `@export var damage: int`.
2.  Você cria um `PlayerProfile.gd` que também herda de `Resource`. Dentro dele, você declara: `@export var equipped_weapon: WeaponData`.

No editor, você agora pode ter vários arquivos:

- `sword.tres` (dano = 15)
- `axe.tres` (dano = 25)
- `player_warrior.tres` (com o `axe.tres` no slot `equipped_weapon`)
- `player_rogue.tres` (com uma `dagger.tres` no slot `equipped_weapon`)

Um designer pode criar centenas de combinações de personagens e armas **sem escrever uma linha de código**, apenas criando e arrastando esses arquivos `.tres` no Inspector. Isso é ROP em sua forma mais pura.

## 5. O Ciclo de Vida do Dado em um Jogo Real

Vamos visualizar como esses formatos interagem durante uma sessão de jogo:

1.  **Design-time (No Editor)**

    - O Game Designer cria `GoblinStats.tres` (um `Resource`) e ajusta o `max_hp` para 50.
    - O Level Designer abre um `LevelConfig.tres` (outro `Resource`) e adiciona o `GoblinStats.tres` a uma lista de inimigos que podem aparecer na fase.
    - O programador edita `settings.cfg` (`ConfigFile`) para adicionar uma nova opção de volume.

2.  **Load-time (Jogo Inicia)**

    - O jogo carrega `settings.cfg` para ajustar o áudio.
    - A cena da floresta é carregada. Um `EnemySpawner` (nó de lógica) lê o `LevelConfig.tres` e, ao instanciar um inimigo, injeta nele o `GoblinStats.tres`. O nó do Goblin, em seu `_ready()`, lê `current_hp = stats.max_hp`.

3.  **Runtime (Gameplay)**

    - O jogador ataca o Goblin. A vida dele (uma `var current_hp: int` no script do nó) diminui.
    - O jogador coleta uma poção. O `InventoryManager` (um Autoload) adiciona a `id` do item e a quantidade a um `Dictionary` que representa o inventário. O estado mudou, mas o `Resource` original da poção (`potion.tres`) permanece intocado.

4.  **Save-time (Persistência)**

    - O jogador ativa um checkpoint. O `SaveMachine` (Autoload) é acionado.
    - Ele pede os dados de runtime a cada sistema:
      - O `Player` retorna um `Dictionary`: `{"hp": 85, "position": Vector2(x, y)}`.
      - O `InventoryManager` retorna seu `Dictionary` de itens.
      - O `QuestManager` retorna um `Dictionary` de quests ativas.
    - O `SaveMachine` junta tudo em um grande dicionário e o serializa para um arquivo `savegame.dat` usando `JSON` ou o formato binário da Godot.

5.  **Load-time (Continuar Jogo)**
    - O processo se inverte. O `SaveMachine` lê `savegame.dat`, o transforma de volta em um `Dictionary`, e entrega a cada sistema sua "fatia" dos dados para que eles restaurem seu estado de runtime.

## Conclusão: A Ferramenta Certa para o Trabalho Certo

- **Para definir "o que é" um item/inimigo/skill (dados de design):** Use `Resource` (`.tres`).
- **Para guardar as configurações do jogador (volume, teclas):** Use `ConfigFile` (`.cfg`).
- **Para falar com o mundo exterior (APIs, web):** Use `JSON`.
- **Para guardar o estado "vivo" e mutável do seu jogo (o que está acontecendo agora):** Use `Dictionary` e `Array` em memória, e serialize-os para o seu arquivo de save.

Dominar esse fluxo é dominar a arquitetura de dados.
