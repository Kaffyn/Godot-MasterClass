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

*   **Declaração (`var`)**: Você escolhe uma gaveta e cola a etiqueta.
*   **Tipagem (`:`)**: Você instala divisórias na gaveta. Agora ela só aceita um tipo de coisa (ex: só cabe `int`).
*   **Atribuição (`=`)**: Você abre a gaveta e coloca o valor lá dentro.
*   **Acesso**: Você chama o nome, o computador abre e te entrega o valor.
*   **Mutabilidade**: Você pode trocar o valor da gaveta, desde que respeite o tipo.

### Tipagem Estrita vs Dinâmica

A regra número 1 do MBA: **Sempre tipe suas variáveis.**

*   **Dinâmica (Amador):** A gaveta é um saco sem fundo. Cabe tudo.
    *   `var vida = 100` (Hoje é número, amanhã pode virar texto "Cem").
    *   *Problema:* Lento e inseguro. Se você somar "Banana" com 10, o jogo crasha.

*   **Estrita (Engenheiro):** A gaveta tem formato específico.
    *   `var vida: int = 100` (Só cabe número inteiro).
    *   *Vantagem:* Rápido e seguro. O compilador te avisa antes do jogo rodar se você tentar guardar a coisa errada.

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

### 1.3. Tipos de Dados

**Primitivos (Básicos):**

- `int`: Números inteiros (1, -5, 42).
- `float`: Números quebrados (3.14, -0.01). Use sempre o ponto `.`.
- `bool`: Verdadeiro ou Falso (`true`, `false`).
- `String`: Texto ("Olá Mundo").

**Tipos de Game Dev (Matemáticos):**
Aqui a Godot brilha. Esses tipos são cidadãos de primeira classe.

- `Vector2`: Uma posição ou direção 2D `(x, y)`.
- `Vector3`: Uma posição ou direção 3D `(x, y, z)`.
- `Color`: Uma cor RGBA.

### 1.4. Enums (Estados Nomeados)

Enums são um tipo especial de variável que só aceita valores de uma lista pré-definida.
Nunca use "números mágicos" ou strings para definir estados.

**Errado:**

```gdscript
if estado == 1: # O que é 1? Ninguém sabe.
    atacar()
```

**Certo:**

```gdscript
enum State { IDLE, RUN, ATTACK, DEAD }

var current_state: State = State.IDLE

func update_state():
    if current_state == State.ATTACK:
        atacar()
```

### 1.5. Constantes (`const`)

Uma constante é uma variável que, uma vez definida, **nunca mais muda**.
Use para valores fixos de configuração.

```gdscript
const MAX_SPEED: float = 500.0
const GRAVITY: float = 980.0

# MAX_SPEED = 600.0 # ERRO! O computador não deixa você mudar.
```

---

## 2. Coleções (Arrays e Dictionaries)

### Arrays (Listas)

Listas ordenadas de coisas.
**Regra:** Use Arrays Tipados sempre que possível.

```gdscript
# Ruim (Array Genérico - cabe tudo, lento)
var mochila = ["Espada", 10, true]

# Bom (Array Tipado - só cabe String, rápido)
var inventario: Array[String] = ["Espada", "Poção", "Mapa"]
```

### Dictionaries (Mapas / HashTables)

Pares de Chave-Valor. Ótimo para inventários complexos ou dados de save.

```gdscript
var player_data: Dictionary = {
    "name": "Machi",
    "level": 55,
    "is_alive": true
}

# Acessando
print(player_data["name"]) # Machi
```

---

## 3. Funções

Funções são contratos. Elas prometem receber algo e devolver algo.
Sempre defina o tipo de retorno com `->`.

```gdscript
# Recebe dois inteiros, devolve um inteiro
func somar(a: int, b: int) -> int:
    return a + b

# Não devolve nada (void)
func morrer() -> void:
    queue_free()
```

---

## 4. Controle de Fluxo (Loops e Condicionais)

### If / Else

O básico.

```gdscript
if health > 0:
    print("Vivo")
else:
    print("Morto")
```

### Match (O Switch Case da Godot)

O `match` é incrivelmente poderoso. Ele compara padrões.

```gdscript
match current_state:
    State.IDLE:
        play_anim("idle")
    State.RUN:
        play_anim("run")
    State.ATTACK:
        play_anim("attack")
    _: # Default (qualquer outra coisa)
        print("Estado inválido")
```

### Loops (For e While)

**For:** Para percorrer listas ou contar.

```gdscript
# Conta de 0 a 9
for i in 10:
    print(i)

# Percorre lista
for item in inventario:
    print("Tenho: " + item)
```

**While:** Enquanto uma condição for verdadeira. Cuidado com loops infinitos!

```gdscript
while health < 100:
    health += 1 # Regenera até encher
```

---

## 5. O "Machi Way" de GDScript

1. **PascalCase** para Classes (`Enemy`, `LevelManager`).
2. **snake_case** para variáveis e funções (`player_health`, `get_input()`).
3. **CONSTANTES** em maiúsculo (`MAX_SPEED`).
4. **Tipagem Estrita** sempre.
5. **Use `as`** para casting seguro: `var enemy = body as Enemy`. Se `body` não for `Enemy`, vira `null` (evita crash).
