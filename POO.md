# Curso Kaffyn: POO & A Filosofia dos Nós

> **Instrutor:** Machi
> **Objetivo:** Entender como a Programação Orientada a Objetos (POO) se traduz no sistema de Nós da Godot. Parar de ver "Scripts" e começar a ver "Classes" e "Objetos".

---

## 1. O Mito do Script

Muitos iniciantes dizem: *"Vou colocar um script nesse objeto"*.
A mentalidade correta é: *"Vou transformar esse objeto em uma instância da minha Classe"*.

Quando você anexa um arquivo `.gd` a um `Node`, você está **estendendo** a funcionalidade base daquele nó.

```gdscript
# meu_player.gd
extends CharacterBody2D
```

Isso significa que `MeuPlayer` **É UM** `CharacterBody2D`. Ele herda tudo: `move_and_slide()`, `velocity`, `position`.

---

## 2. Os 4 Pilares da POO na Godot

### A. Abstração
Esconder a complexidade.
Um `Carro` não precisa saber como o `Motor` queima combustível. Ele apenas chama `Motor.ligar()`.

**Na Godot:**
Seu nó `Player` não deve saber como a `Gun` atira.
Errado: `Player` instancia a bala e calcula a física.
Certo: `Player` chama `$Gun.shoot()`. A `Gun` resolve o resto.

### B. Encapsulamento
Proteger os dados. Variáveis internas não devem ser tocadas de fora.

```gdscript
# health_component.gd
var _hp: int = 100 # Privado (convenção _)

func damage(amount: int):
    _hp -= amount
    if _hp <= 0:
        die()
```

Se alguém fizer `player.hp = -50` diretamente, pode quebrar a lógica de morte. Obrigue o uso de funções (`damage()`).

### C. Herança
Criar versões especializadas de algo genérico.

**Exemplo Clássico:**
1. `Enemy` (Base): Tem vida, toma dano, segue o player.
2. `Goblin` (extends Enemy): Corre rápido.
3. `Orc` (extends Enemy): Bate forte e tem muita vida.

```gdscript
# enemy.gd
class_name Enemy extends CharacterBody2D

func take_damage(amount):
    print("Ai!")

# orc.gd
extends Enemy

func take_damage(amount):
    super(amount) # Chama o código do Enemy base
    play_angry_sound() # Adiciona comportamento extra
```

### D. Polimorfismo
Tratar objetos diferentes da mesma maneira.

Se você tem uma lista de inimigos (`Goblin`, `Orc`, `Dragon`), você não quer saber qual é qual. Você só quer causar dano.

```gdscript
# explosion.gd
func _on_body_entered(body):
    # Não importa se é Orc ou Goblin. Se é um Inimigo, ele sangra.
    if body is Enemy:
        body.take_damage(50)
```

---

## 3. Composição sobre Herança (O Jeito Godot)

Embora Herança seja útil, **Composição** é o superpoder da Godot.
Em vez de criar uma árvore genealógica gigante (`Enemy -> FlyingEnemy -> ShooterFlyingEnemy`), nós montamos o objeto com **Componentes**.

**A Regra:** "Tem um" é melhor que "É um".

- O Player **TEM UM** `HealthComponent`.
- O Player **TEM UM** `InventoryComponent`.
- O Player **TEM UM** `InputHandler`.

**Na SceneTree:**
```
Player (CharacterBody2D)
├── HealthComponent (Node)
├── HitboxComponent (Area2D)
└── Gun (Node2D)
```

Se você quiser que uma `Caixa` tenha vida, você não faz `Caixa extends Player`. Você apenas arrasta o `HealthComponent` para dentro da cena da Caixa.

---

## 4. Classes e Tipagem

Use `class_name` para elevar seus scripts a cidadãos de primeira classe.

```gdscript
# interactable.gd
class_name Interactable extends Area2D

func interact(user):
    pass
```

Agora, em qualquer lugar do projeto:

```gdscript
# player.gd
func _input(event):
    if event.is_action_pressed("interact"):
        var bodies = get_overlapping_bodies()
        for body in bodies:
            if body is Interactable:
                body.interact(self)
```

Isso elimina a necessidade de checar strings, grupos ou caminhos de nós. O sistema de tipos da linguagem trabalha para você.

---

## 5. Nodes, Scenes e a SceneTree

Muitos confundem estes termos. Vamos definir a hierarquia da existência na Godot.

### Nodes (Nós)
O bloco fundamental. Um nó é uma **Classe** que herda de `Node`. Ele tem:
- Nome.
- Propriedades (x, y, rotation).
- Callbacks (`_ready`, `_process`).
- Capacidade de ter filhos.

### Scenes (Cenas - PackedScene)
Uma Cena (`.tscn`) é um **Blueprint** (Planta Baixa) de uma árvore de nós.
Ela não "existe" no jogo até ser instanciada.
- `Player.tscn` é o arquivo no disco.
- Quando você faz `instantiate()`, você cria os nós na memória RAM.

### SceneTree (Árvore de Cena)
É o "Mundo Ativo". O gerenciador principal do loop do jogo.
- Contém a `root` (Window).
- Contém a `current_scene` (o nível atual).
- Contém os `Autoloads` (Singletons).

**Hierarquia Visual:**
```
SceneTree (Main Loop)
├── root (Window)
│   ├── Global (Autoload)
│   ├── SoundManager (Autoload)
│   └── Level01 (Current Scene)
│       ├── Player
│       └── Enemies
```

---

## 6. Sinais (Observer Pattern)

Como os nós conversam sem criar dependências "Spaghetti"?
**Regra de Ouro:** *Call Down, Signal Up.*
- **Pai chama Filho:** O Pai sabe quem é o filho (`$Filho`). Pode chamar funções dele.
- **Filho NÃO conhece Pai:** O Filho nunca deve fazer `get_parent().funcao()`. Se o pai mudar, o filho quebra.

### O Que é um Sinal?
É um evento. O nó grita: *"Aconteceu algo!"*. Quem estiver ouvindo (conectado) reage. O nó que gritou não sabe quem (ou se alguém) ouviu.

### Exemplo Prático: Barra de Vida (HUD)

1. **O Player (Emissor):**
Ele não sabe que existe uma barra de vida na tela. Ele só avisa que a vida mudou.

```gdscript
# player.gd
extends CharacterBody2D

signal health_changed(new_value: int, max_value: int) # Declaração
signal died

var hp: int = 100
var max_hp: int = 100

func take_damage(amount):
    hp -= amount
    # Emite o sinal avisando o novo valor
    health_changed.emit(hp, max_hp)
    
    if hp <= 0:
        died.emit()
```

2. **A UI (Receptor):**
A UI sabe quem é o Player (provavelmente foi injetado nela ou ela o buscou).

```gdscript
# health_bar.gd (HUD)
extends ProgressBar

func _ready():
    # Busca o player (exemplo simples)
    var player = get_tree().get_first_node_in_group("Player")
    if player:
        # CONECTA o sinal do player à função local 'update_bar'
        player.health_changed.connect(update_bar)

# Essa função só roda quando o sinal é emitido
func update_bar(new_hp, max_hp):
    value = new_hp
    max_value = max_hp
```

### Sinais com `await` (Corrotinas)

Sinais também servem para pausar a execução até algo acontecer.

```gdscript
func _on_button_pressed():
    print("Iniciando cutscene...")
    $AnimationPlayer.play("intro_cutscene")
    
    # Espera a animação terminar antes de continuar
    await $AnimationPlayer.animation_finished
    
    print("Cutscene acabou. Iniciando jogo.")
    get_tree().change_scene_to_file("res://level_1.tscn")
```