# Módulo 14: Singletons & Autoloads – Os Gerentes Globais do seu Jogo

## 1. Introdução: O Problema do Acesso Global e o Anti-Padrão do "Objeto Deus"

À medida que seu jogo cresce, surge uma necessidade inevitável: ter sistemas que possam ser acessados de qualquer lugar. Como o script do player pode tocar um som sem ter uma referência direta ao nó de áudio? Como um inimigo informa ao HUD que a pontuação aumentou? A resposta comum é um "objeto global".

Na Godot, a ferramenta para isso é o **Autoload**. No entanto, essa facilidade de acesso é uma faca de dois gumes. Se mal utilizada, ela leva ao anti-padrão mais temido da arquitetura de software: o **"God Object"** (Objeto Deus). Este é um único Autoload, frequentemente chamado `Global.gd`, que acaba fazendo tudo: controla o som, o save, a UI, o estado do jogador, as quests... Ele sabe tudo e gerencia tudo.

O "God Object" é o inimigo da manutenção. Mudar uma pequena coisa no sistema de áudio pode quebrar o sistema de save, porque ambos vivem no mesmo arquivo gigante e acoplado. O "Machi Class" ensina a usar o poder dos singletons com a disciplina de um arquiteto, criando múltiplos gerentes especializados em vez de um único "faz-tudo".

## 2. Autoloads: A Solução Padrão da Godot

Um Autoload é a implementação do padrão Singleton na Godot. Tecnicamente, é um nó (com ou sem cena) que a Godot garante que será carregado no início do jogo e adicionado à raiz da árvore de cenas (`root`), tornando-o persistente entre as trocas de cena e acessível globalmente.

- **Como Criar**: Vá em `Project -> Project Settings -> Autoload`. Adicione o caminho para o seu script (`.gd`) ou cena (`.tscn`) e dê a ele um nome global. É por este nome que você o acessará.

- **Benefícios Arquiteturais**:
  - **Acesso Global Confiável**: De qualquer script, você pode chamar `SaveMachine.save()` ou `AudioManager.play_music()`. Simples e direto.
  - **Persistência de Estado**: Como o Autoload não é destruído com a troca de cenas, ele é o lugar perfeito para guardar informações que precisam sobreviver entre as fases, como a pontuação do jogador ou o inventário.
  - **Ponto de Entrada Único (API do Sistema)**: Em vez de ter lógica de save espalhada, todo o código do jogo interage com um único ponto: o `SaveMachine`. Isso centraliza a responsabilidade e facilita a depuração.

## 3. Estudo de Caso: A `GameMachine` – Um Controlador de Estado Global

Um dos usos mais poderosos de um Autoload é criar uma "Máquina de Estado do Jogo" (`GameMachine`). Sua única responsabilidade é saber "o que está acontecendo no jogo em um nível macro?".

- **Propósito**: Evitar que a lógica de "pausado", "no menu" ou "em loading" se espalhe por todos os seus scripts. O inimigo não precisa saber se o jogo está pausado; ele simplesmente não deve ser processado se o estado global não for `IN_GAME`.

- **Implementação (Exemplo de Código)**:

  ```gdscript
  # GameMachine.gd (Configurado como Autoload com o nome "GameMachine")
  extends Node

  # Enum para os estados possíveis do jogo. Claro e legível.
  enum State { MAIN_MENU, LOADING, IN_GAME, PAUSED, SAVING, GAME_OVER }

  # Sinal emitido sempre que o estado muda. A UI pode ouvir isso.
  signal state_changed(new_state)

  private var current_state: State = State.MAIN_MENU

  func _ready():
      # Garante que o jogo comece pausado se estivermos no menu
      get_tree().paused = (current_state == State.MAIN_MENU or current_state == State.PAUSED)

  # O método central que controla todas as transições de estado
  func change_state(new_state: State):
      if new_state == current_state: return

      current_state = new_state
      # A propriedade 'paused' da SceneTree para o processamento de nós (exceto os que têm process_mode = ALWAYS)
      get_tree().paused = (current_state == State.PAUSED or current_state == State.MAIN_MENU)
      state_changed.emit(current_state)
      print("Game state changed to: ", State.keys()[new_state])

  # --- API Pública do nosso sistema ---

  func start_new_game():
      change_state(State.LOADING)
      # ... lógica para carregar a cena do jogo ...
      # (ex: await get_tree().change_scene_to_file("res://level1.tscn"))
      # Ao terminar o loading:
      change_state(State.IN_GAME)

  func pause_game():
      if current_state == State.IN_GAME:
          change_state(State.PAUSED)

  func resume_game():
      if current_state == State.PAUSED:
          change_state(State.IN_GAME)

  func get_current_state() -> State:
      return current_state
  ```

- **Como outros sistemas o usam**:

  - **UI de Pausa**: Um script no menu de pausa, ao ser aberto, chama `GameMachine.pause_game()`. O botão "Continuar" chama `GameMachine.resume_game()`.
  - **Scripts de Inimigos/Player**: No topo do `_physics_process`, eles podem ter uma guarda:

    ```gdscript
    func _physics_process(delta):
        if GameMachine.get_current_state() != GameMachine.State.IN_GAME:
            return # Não faz nada se não estivermos no estado de jogo ativo

        # ... resto da lógica de movimento e IA ...
    ```

## 4. Os Singletons Ocultos da Godot: Os "Servers"

A Godot possui seus próprios singletons nativos, globais e extremamente poderosos, mas que ficam "escondidos" da maioria dos usuários: os **Servers**.

- **O que são?** São os singletons de mais baixo nível da engine. Existem para cada subsistema principal: `RenderingServer`, `PhysicsServer2D`, `PhysicsServer3D`, `AudioServer`, `NavigationServer2D`, etc.

- **Por que existem?** **PERFORMANCE.** Quando você cria um nó `Sprite2D` e muda sua posição, você está usando uma "fachada" amigável. Por baixo dos panos, o `Sprite2D` está fazendo chamadas complexas para o `RenderingServer`, que é quem realmente desenha as coisas na tela. Para 100 sprites, usar nós é ótimo. Para 10.000 sprites, o custo de gerenciar cada nó na árvore de cenas se torna um gargalo.

- **O Trade-off:** Usar os Servers diretamente é ordens de magnitude mais rápido, mas também muito mais complexo e verboso. Você perde todas as conveniências dos nós (sinais, grupos, `_ready`, etc.) e passa a trabalhar com IDs numéricos (RIDs - Resource IDs).

  _Exemplo: Mover um objeto_

  **O jeito fácil (Nós):**

  ```gdscript
  $Sprite2D.position = Vector2(100, 100)
  ```

  **O jeito rápido (Servers):**

  ```gdscript
  # 'canvas_item' é um RID (um número) que você obteve ao criar o item
  var new_transform = Transform2D(0, Vector2(100, 100))
  RenderingServer.canvas_item_set_transform(canvas_item, new_transform)
  ```

- **A Lição do MBA:** Você não precisa usar os Servers no seu primeiro jogo. Mas um arquiteto de software **precisa saber que eles existem**. Eles são sua rota de fuga, sua "arma secreta" para quando a performance se torna um problema crítico. Saber quando e por que descer para este nível de abstração é o que diferencia um hobbyista de um engenheiro profissional.

## 5. Quando NÃO Usar um Singleton Global (Autoload)

Singletons são uma ferramenta poderosa, mas para problemas específicos. Usá-los para tudo é um sinal de má arquitetura.

- **Lógica Específica de Cena**: Se um "manager" só faz sentido dentro de uma cena específica (ex: um `MinigameManager` para um puzzle), ele **não** deve ser um Autoload. Ele deve ser um nó normal naquela cena. Se outros nós na mesma cena precisam acessá-lo, use o padrão de "Singleton Manual" (um nó com uma variável `static var instance`).
- **Dados de Configuração**: Stats de armas, definições de itens, diálogos, etc., não devem ser hard-coded em um Autoload. Eles pertencem a `Resources` (`.tres` arquivos), pois são dados, não sistemas. Isso permite que designers de jogo os editem sem tocar em código.
- **Simples "Saco de Variáveis"**: Se seu Autoload é apenas uma lista de variáveis globais (`var jogador_ganhou = false`), isso é um forte indício de "código espaguete". O estado deve ser gerenciado e encapsulado dentro do sistema que é responsável por ele.
