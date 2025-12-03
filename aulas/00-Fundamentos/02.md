# GDScript: A Linguagem do Engenheiro

Muitos tratam GDScript como uma linguagem de "scripting" (como Lua ou Python), feita para colar coisas com cuspe. **Errado.**
GDScript é uma linguagem orientada a objetos, gradualmente tipada e otimizada para a engine.

Se você escreve GDScript como Python, seu jogo será lento e cheio de bugs.
Se você escreve GDScript como C#, seu jogo será robusto e escalável.

---

## 1. Variáveis e Tipagem Estrita (Static Typing)

A regra número 1 do MBA: **Sempre tipe suas variáveis.**

A tipagem dinâmica (`var vida = 100`) é para protótipos de 5 minutos.
A tipagem estrita (`var vida: int = 100`) é para engenharia.

### Por que tipar?

1. **Performance:** A Godot não precisa "adivinhar" o tipo a cada frame.
2. **Segurança:** O compilador te avisa se você tentar somar "Banana" com 10.
3. **Autocomplete:** A IDE sabe o que o objeto é e te mostra as funções disponíveis.

```gdscript
# AMADOR (Dinâmico)
var health = 100
var player_name = "Machi"

# ENGENHEIRO (Estrito)
var health: int = 100
var player_name: String = "Machi"

# INFERÊNCIA DE TIPO (O atalho seguro)
# O operador := diz: "O tipo dessa variável é o tipo do valor que ela recebe"
var speed := 300.0 # A Godot sabe que é float e TRAVA como float.
```

---

## 2. Tipos de Dados

### Primitivos (Básicos)

- `int`: Números inteiros (1, -5, 42).
- `float`: Números quebrados (3.14, -0.01). Use sempre o ponto `.`.
- `bool`: Verdadeiro ou Falso (`true`, `false`).
- `String`: Texto ("Olá Mundo").

### Tipos de Game Dev (Matemáticos)

Aqui a Godot brilha. Esses tipos são cidadãos de primeira classe.

- `Vector2`: Uma posição ou direção 2D `(x, y)`.
  - `var pos := Vector2(100, 200)`
- `Vector3`: Uma posição ou direção 3D `(x, y, z)`.
- `Color`: Uma cor RGBA.
  - `var red := Color.RED`

---

## 3. Coleções (Arrays e Dictionaries)

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

## 4. Enums (Estados Nomeados)

Nunca use "números mágicos" ou strings para definir estados. Use Enums.

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

Enums transformam números em palavras legíveis. Internamente `IDLE` é `0`, `RUN` é `1`, etc.

---

## 5. Funções

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

## 6. Controle de Fluxo (Loops e Condicionais)

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

## 7. O "Machi Way" de GDScript

1. **PascalCase** para Classes (`Enemy`, `LevelManager`).
2. **snake_case** para variáveis e funções (`player_health`, `get_input()`).
3. **CONSTANTES** em maiúsculo (`MAX_SPEED`).
4. **Tipagem Estrita** sempre.
5. **Use `as`** para casting seguro: `var enemy = body as Enemy`. Se `body` não for `Enemy`, vira `null` (evita crash).
