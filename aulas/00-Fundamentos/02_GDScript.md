# GDScript: A Linguagem do Engenheiro

Muitos tratam GDScript como uma linguagem de "scripting" (como Lua ou Python), feita para colar coisas com cuspe. **Errado.**
GDScript é uma linguagem orientada a objetos, gradualmente tipada e otimizada para a engine.

Se você escreve GDScript como Python, seu jogo será lento e cheio de bugs.
Se você escreve GDScript como C#, seu jogo será robusto e escalável.

---

## 1. Variáveis: A Memória do Jogo

Imagine que a memória do computador é um armário gigante cheio de gavetas.
Uma **Variável** é uma etiqueta que você cola em uma dessas gavetas para guardar algo dentro.

Mas na Engenharia de Software, não basta jogar coisas na gaveta. Você precisa definir **o que cabe nela**.

### O Ciclo de Vida da Gaveta

- **Declaração (`var`)**: Você escolhe uma gaveta e cola a etiqueta.
- **Tipagem (`:`)**: Você instala divisórias na gaveta. Agora ela só aceita um tipo de coisa (ex: só cabe `int`).
- **Atribuição (`=`)**: Você abre a gaveta e coloca o valor lá dentro.
- **Acesso**: Você chama o nome, o computador abre e te entrega o valor.
- **Mutabilidade**: Você pode trocar o valor da gaveta, desde que respeite o tipo.

### Tipagem Estrita vs Dinâmica

A regra número 1 do MBA: **Sempre tipe suas variáveis.**

- **Dinâmica (Amador):** A gaveta é um saco sem fundo. Cabe tudo.

  - `var vida = 100` (Hoje é número, amanhã pode virar texto "Cem").
  - _Problema:_ Lento e inseguro. Se você somar "Banana" com 10, o jogo crasha.

- **Estrita (Engenheiro):** A gaveta tem formato específico.
  - `var vida: int = 100` (Só cabe número inteiro).
  - _Vantagem:_ Rápido e seguro. O compilador te avisa antes do jogo rodar se você tentar guardar a coisa errada.

```gdscript
# AMADOR (Dinâmico - Perigoso)
var health = 100
var player_name = "Machi"

# ENGENHEIRO (Estrito - Seguro)
var health: int = 100
var player_name: String = "Machi"

# INFERÊNCIA DE TIPO (O atalho inteligente)
# O operador := diz: "Crie uma gaveta do formato exato desse valor inicial"
var speed := 300.0 # A Godot vê 300.0 (float) e tranca a gaveta como float.
```

### Tipos de Dados Essenciais

**Primitivos:**

- `int`: Números inteiros (1, -5, 42).
- `float`: Números quebrados (3.14, -0.01). Use sempre o ponto `.`.
- `bool`: Verdadeiro ou Falso (`true`, `false`).
- `String`: Texto ("Olá Mundo").

**Matemáticos (Godot):**

- `Vector2`: Posição ou direção 2D `(x, y)`.
- `Vector3`: Posição ou direção 3D `(x, y, z)`.
- `Color`: Cor RGBA.

**Especiais:**

- **Enums:** Lista de opções fixas. Evita "números mágicos".

```gdscript
enum State { IDLE, RUN, ATTACK }
var current: State = State.IDLE
```

- **Constantes (`const`):** Variáveis que nunca mudam.

```gdscript
const MAX_SPEED: float = 500.0
```

---

## 2. Funções: Os Verbos do Jogo

Se variáveis são "Substantivos" (Vida, Velocidade), Funções são "Verbos" (Correr, Atacar, Morrer).
Uma função é um bloco de código que faz uma tarefa específica.

### A Função mais famosa: `print()`

Antes de criar as nossas, saiba que a Godot já vem com várias prontas.
`print("Texto")` é uma função que escreve algo no console de saída. É vital para debugar.

### Anatomia de uma Função

```gdscript
# func [nome]([parâmetros]) -> [tipo_retorno]:
func take_damage(amount: int) -> bool:
    health -= amount

    if health <= 0:
        return true # Morreu
    return false # Sobreviveu
```

1. **Parâmetros:** O que a função precisa receber para trabalhar (`amount`).
2. **Retorno (`->`):** O que a função devolve para quem chamou. Se não devolver nada, use `-> void`.
3. **Escopo:** Variáveis criadas dentro da função morrem quando a função termina.

---

## 3. Controle de Fluxo: Tomando Decisões

O código não precisa rodar linha por linha. Ele pode bifurcar.

### If / Elif / Else (Se / Senão Se / Senão)

Pense nisso como uma conversa em português:

- **`if` (Se):** "Se a vida for maior que 50..."
- **`elif` (Senão Se):** "Senão, se a vida for maior que 10..."
- **`else` (Senão):** "Se não for nada disso..."

```gdscript
if health > 50:
    print("Estou bem!") # Só roda se health > 50
elif health > 10:
    print("Estou ferido...") # Só roda se health <= 50 E health > 10
else:
    print("Vou morrer!") # Roda se nenhuma das anteriores for verdade
```

### Match (O Switch Case)

Pense no `match` como um **App de Namoro**.
Dar "Match" significa encontrar a combinação perfeita.

O código pega o valor da sua variável (o "perfil") e compara com várias opções. Quando ele encontra uma que combina, ele executa aquele bloco.

- **`match current_state`:** "Quem combina com o meu estado atual?"
- **`State.IDLE`:** "Eu combino!" -> Toca animação Idle.
- **`_` (Underscore):** É o "Tinder Gold". Combina com todo mundo. Se ninguém mais der match, ele assume.

```gdscript
match current_state:
    State.IDLE:
        play_anim("idle")
    State.RUN:
        play_anim("run")
    _: # Default (Se não for nem IDLE nem RUN)
        print("Estado desconhecido")
```

### Loops (Repetição)

- **For:** Quando você sabe quantas vezes quer repetir ou quer percorrer uma lista.

```gdscript
for i in 5: # 0, 1, 2, 3, 4
    print(i)
```

- **While:** Repete enquanto uma condição for verdade (Cuidado com loops infinitos!).

```gdscript
while health < 100:
    health += 1
```

---

## 4. Coleções: Organizando Dados

### Arrays (Listas)

Uma gaveta com várias divisórias numeradas (0, 1, 2...).

```gdscript
# Array Tipado (Só aceita Strings) - RÁPIDO
var inventory: Array[String] = ["Espada", "Poção"]

# Adicionar
inventory.append("Mapa")

# Acessar
print(inventory[0]) # Espada
```

### Dictionaries (Mapas)

Uma gaveta onde cada divisória tem um nome (Chave) em vez de um número.

```gdscript
var stats: Dictionary = {
    "str": 10,
    "dex": 5,
    "int": 2
}

print(stats["str"]) # 10
```

---

## 5. Orientação a Objetos (POO): O Poder da Godot

Aqui é onde deixamos de ser "scripters" e viramos engenheiros.
Na Godot, **cada arquivo `.gd` é uma Classe**.

### Definindo uma Classe

```gdscript
# hero.gd
class_name Hero extends CharacterBody2D

var hp: int = 100

func attack():
    print("Heroi atacou!")
```

- **`class_name Hero`**: Agora `Hero` é um tipo global no projeto, igual a `int` ou `Sprite2D`.
- **`extends CharacterBody2D`**: O Herói herda tudo que um CharacterBody tem (física, colisão, movimento).

### Instanciando (Criando Objetos)

```gdscript
# Em outro script...
var my_hero = Hero.new() # Cria um novo herói na memória
my_hero.hp = 50
my_hero.attack()
```

### Casting (`as`) e Type Check (`is`)

Quando você colide com algo, a Godot diz que é um `Node` ou `Object`. Você precisa verificar o que é.

```gdscript
func _on_body_entered(body: Node):
    # Verificação (É um inimigo?)
    if body is Enemy:
        # Casting (Trate esse corpo como Inimigo)
        var enemy = body as Enemy
        enemy.take_damage(10)
```

> **Machi Way:** Use `class_name` para tudo. Isso permite que o autocomplete te ajude e evita erros de digitação.
