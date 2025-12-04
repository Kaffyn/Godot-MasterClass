# Leis da Física: Bodies e Areas

A física da Godot é poderosa, mas exige que você escolha a ferramenta certa para o trabalho. Não tente martelar um prego com uma chave de fenda.

---

## 1. Os Três Corpos (Bodies)

### StaticBody (A Parede)

- **Comportamento:** Imóvel. Não se move, não sofre gravidade, não é empurrado.
- **Uso:** Chão, Paredes, Plataformas fixas.
- **Performance:** Extremamente leve.
- **Variação:** `AnimatableBody` (Filho de StaticBody). Use para plataformas móveis ou portas que se movem via código/animação, mas que devem empurrar o jogador sem serem empurradas.

### RigidBody (A Bola de Futebol)

- **Comportamento:** Controlado pela Física (Gravidade, Inércia, Forças). Você não define a posição dele; você aplica forças (`apply_force`).
- **Uso:** Caixas, Pedras rolando, Ragdolls.
- **Variação:** `PhysicalBone` (Filho de RigidBody). Usado especificamente para simular ossos em um esqueleto 3D (Ragdolls complexos).
- **Cuidado:** Difícil de controlar para personagens (o jogador sente que está "escorregando").

### CharacterBody (O Herói)

- **Comportamento:** Kinematic (Cinemático). Você controla tudo via código (`velocity`, `move_and_slide`). Ele colide com paredes e desliza, mas não sofre forças externas a menos que você as programe.
- **Uso:** Player, Inimigos, NPCs. É o padrão para jogos de plataforma e RPG.

---

## 2. Area2D: O Sensor Invisível

`Area2D` não bloqueia movimento. Ela detecta presença.
É a ferramenta mais importante para lógica de jogo.

- **Sinais Principais:**
  - `body_entered(body)`: Alguém entrou na área?
  - `area_entered(area)`: Outra área entrou aqui? (Ótimo para Hitbox vs Hurtbox).

**Exemplo: Moeda**
A moeda é uma `Area2D`. Quando o Player (`CharacterBody`) entra nela, a moeda emite `body_entered`, dá pontos e se deleta.

---

## 3. Layers e Masks (Quem vê quem?)

O sistema de colisão pode virar um caos se tudo colidir com tudo. Use Layers e Masks para filtrar.

- **Collision Layer (Onde eu estou):** "Eu sou um Inimigo".
- **Collision Mask (O que eu vejo):** "Eu colido com Paredes e Players".

**Cenário:**

- O Inimigo está na Layer `Enemy`.
- A Mask do Inimigo marca `Player` e `World`.
- Isso significa que o Inimigo bate no Player e na Parede, mas **atravessa outros Inimigos** (porque `Enemy` não está na Mask dele).

> **Dica:** Nomeie suas Layers em `Project Settings > Layer Names`. Nunca confie em "Layer 1" ou "Layer 2".

---

## 4. Move and Slide

Para `CharacterBody`, a função mágica é `move_and_slide()`.

1. Você define a `velocity` (Vector2).
2. Chama `move_and_slide()`.
3. A Godot move o corpo, detecta colisões, desliza pelas paredes e **atualiza a velocity** (se bater no chão, a velocidade Y zera).

```gdscript
func _physics_process(delta):
    velocity.y += gravity * delta # Aplica gravidade

    if Input.is_action_pressed("ui_right"):
        velocity.x = speed
    else:
        velocity.x = 0

    move_and_slide() # A mágica acontece aqui
```
