# Controle Total: O Sistema de Input

Esqueça `Input.is_key_pressed(KEY_SPACE)`. Isso é hardcoding e impede que o jogador reconfigure os controles.
Engenheiros usam **Actions** (Ações).

---

## 1. Input Map (Mapa de Ações)

Vá em `Project > Project Settings > Input Map`.
Aqui você define **O QUE** o jogador quer fazer, não **COMO**.

- Crie uma ação: `jump`
- Associe teclas: `Space`, `Gamepad Bottom Button` (X/A).

Agora seu código responde a `jump`, não importa se veio do teclado ou do controle de Xbox.

---

## 2. Polling vs Eventos

Existem duas formas de ler Input. Cada uma tem seu lugar.

### Polling (Perguntar a todo frame)

Você pergunta no `_process` ou `_physics_process`: "O botão está apertado agora?"

- **Função:** `Input.is_action_pressed("jump")` (Segurando) ou `Input.is_action_just_pressed("jump")` (Apertou agora).
- **Uso:** Movimento contínuo (andar, acelerar carro), metralhadoras.

```gdscript
func _physics_process(delta):
    if Input.is_action_pressed("move_right"):
        velocity.x += speed
```

### Eventos (Reagir quando acontece)

A Godot te avisa quando um input chega. Isso acontece na função `_unhandled_input(event)`.

- **Uso:** Menus, Inventário, Pulo (ações únicas), Digitação.
- **Vantagem:** Não roda a cada frame (economia de CPU).

```gdscript
func _unhandled_input(event):
    if event.is_action_pressed("jump"):
        jump()
```

---

## 3. Input Buffering (Conceito Avançado)

Jogos de ação precisos (como Celeste ou Hollow Knight) não confiam apenas no input do frame exato. O jogador humano erra por milissegundos.

**Input Buffering** é a técnica de "lembrar" que o jogador apertou o botão um pouco antes de poder usar.

- _Cenário:_ O jogador aperta Pulo 0.1s antes de tocar no chão.
- _Sem Buffer:_ O input é ignorado. O jogador acha que o jogo falhou.
- _Com Buffer:_ O jogo guarda o "Pulo" na memória por 0.2s. Assim que toca no chão, o personagem pula.

> **Machi Way:** Implemente Input Buffering usando um `Timer` simples. Quando apertar Pulo, inicie o timer. Se tocar no chão e o timer > 0, pule.
