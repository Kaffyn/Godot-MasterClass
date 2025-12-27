# Guia de Desenvolvimento com Godot Engine

Este guia foca nas ferramentas e técnicas específicas para o desenvolvimento de jogos com a Godot Engine.

## 1. A Estrutura da Godot: Cenas e Nós

O conceito mais fundamental da Godot é a **árvore de cena**.

- **Cena:** Uma coleção de **Nós (Nodes)** organizados em uma estrutura de árvore (com pais e filhos). Uma cena pode ser um personagem, uma arma, um nível inteiro, ou o menu principal.
- **Nó:** O bloco de construção básico em Godot. Cada nó tem uma função específica: um `Sprite2D` exibe uma textura, um `Camera2D` controla a visão do jogo, um `CharacterBody2D` é usado para personagens que colidem com o ambiente.
- **Composição sobre Herança:** Você constrói objetos complexos **compondo** nós simples. Para criar um jogador, você pode usar um `CharacterBody2D` (para física e movimento) e adicionar como filhos um `Sprite2D` (para a aparência) e uma `CollisionShape2D` (para definir a área de colisão).

## 2. GDScript: A Linguagem da Godot

GDScript é a linguagem de script nativa da Godot, projetada para ser concisa e integrada à engine. Sua sintaxe é fortemente inspirada em Python.

### Características Principais

- **Tipagem Opcional:** Você pode declarar variáveis com ou sem tipo.

  ```gdscript
  var vida = 100 # Tipagem dinâmica
  var energia: int = 100 # Tipagem estática
  ```

  Usar tipagem estática é **altamente recomendado**, pois melhora o autocompletar, reduz erros e pode aumentar a performance.

- **Sinais (Signals):** O sistema de comunicação preferido em Godot. Um nó pode "emitir" um sinal quando um evento ocorre, e outros nós podem "conectar-se" a esse sinal para reagir.

  - **Exemplo:** O nó `Button` emite o sinal `pressed` quando clicado.

  ```gdscript
  # Conecta o sinal "pressed" do botão à função "_on_button_pressed"
  $MyButton.pressed.connect(_on_button_pressed)

  func _on_button_pressed():
      print("O botão foi pressionado!")
  ```

  Isso desacopla o código: o botão não precisa saber quem está ouvindo, ele apenas anuncia o que aconteceu.

- **Palavra-chave `@export`:** Expõe uma variável no Inspetor da Godot, permitindo que você ou um game designer alterem seu valor sem tocar no código.

  ```gdscript
  @export var velocidade: float = 200.0
  @export var mensagem_secreta: String
  ```

- **Ciclo de Vida de um Script (`_process` e `_physics_process`):**
  - `_process(delta)`: Chamado a cada frame de renderização. Ideal para lógica que não envolve física, como atualizar a UI ou checar input. `delta` é o tempo desde o último frame.
  - `_physics_process(delta)`: Chamado a uma taxa de atualização fixa (padrão de 60 vezes por segundo). **Use sempre este para lógica de física e movimento** para garantir que o comportamento seja consistente independentemente do framerate.

## 3. Resources: Armazenando Dados

Resources são um dos recursos mais poderosos da Godot. Eles são objetos de dados que podem ser salvos em arquivos (`.tres` ou `.res`) e carregados na memória.

- **Tipos Nativos:** `Texture`, `AudioStream`, `PackedScene` (suas cenas `.tscn` são resources!).
- **Criando seus próprios Resources:** Você pode criar scripts que herdam de `Resource` para definir suas próprias estruturas de dados.

  ```gdscript
  # scripts/ItemData.gd
  class_name ItemData
  extends Resource

  @export var nome: String
  @export var descricao: String
  @export var icone: Texture
  @export var dano: int = 0
  @export var valor: int = 1
  ```

- **Como Usar:**

  1. No "Sistema de arquivos" do editor, clique com o botão direito -> Novo -> Resource... -> `ItemData`.
  2. Salve o arquivo como `espada.tres`.
  3. Clique no arquivo `espada.tres` e preencha os campos no Inspetor (Nome: "Espada Épica", Dano: 25, etc).
  4. No seu código, carregue e use este resource:

  ```gdscript
  # Em qualquer script
  const ESPADA_DATA = preload("res://items/espada.tres")

  func _ready():
      print("Equipado:", ESPADA_DATA.nome)
      aplicar_dano(ESPADA_DATA.dano)
  ```

- **Por que usar Resources?**
  - **Separação de Dados e Código:** Mantém sua lógica limpa.
  - **Facilidade de Manutenção:** Designers podem balancear o jogo (alterar dano, valor, etc.) editando arquivos `.tres` sem precisar mexer no código.
  - **Reutilização:** O mesmo resource pode ser compartilhado por múltiplas instâncias de cena.

## 4. GDExtension: Usando C++ para Performance

Quando o GDScript não é rápido o suficiente, você pode recorrer ao C++ usando o sistema **GDExtension**.

- **O que é?** Uma interface que permite que a Godot se comunique com bibliotecas compiladas (C++, Rust, etc.). Você pode criar classes em C++ e usá-las diretamente no editor da Godot como se fossem nós ou classes nativas.

- **Quando usar?**

  - **Algoritmos Pesados:** IA complexa, geração procedural de mundos, simulações físicas customizadas.
  - **Processamento de Dados em Massa:** Qualquer tarefa que siga os princípios do Design Orientado a Dados (DOD) e precise iterar sobre milhares de elementos.
  - **Integração de Bibliotecas:** Para usar bibliotecas de terceiros escritas em C ou C++.

- **Como Funciona (Visão Geral):**
  1. **Setup do Projeto:** Você precisa clonar o repositório `godot-cpp` que contém os "bindings" (a ponte) entre Godot e C++.
  2. **Definição da Classe:** Em C++, você cria uma classe que herda de uma classe base da Godot (ex: `Godot::Sprite2D`).
  3. **Compilação:** Você compila seu código C++ em uma biblioteca dinâmica (.dll para Windows, .so para Linux, .dylib para macOS).
  4. **Arquivo `.gdextension`:** No seu projeto Godot, você cria um arquivo de configuração (`.gdextension`) que diz à engine onde encontrar sua biblioteca compilada e quais classes ela contém.
  5. **Uso:** Uma vez carregada, sua classe C++ aparecerá na lista "Adicionar Nó" e poderá ser usada como qualquer outro nó da Godot, com seus métodos e propriedades expostos ao GDScript e ao editor.
