# O Coração da Máquina: Game Loop e Ciclo de Vida

Um jogo não é um software comum que espera você clicar. Ele é um loop infinito rodando 60 (ou mais) vezes por segundo. Entender esse loop é a diferença entre um jogo fluido e um jogo travado.

---

## 1. O Ciclo de Vida de um Node

Todo Node passa por fases desde o nascimento até a morte.

1. **`_init()`**: O Construtor. O objeto foi alocado na memória RAM.

   - _Uso:_ Inicializar variáveis que não dependem de outros nós.
   - _Cuidado:_ O nó ainda não está na SceneTree. Não tente acessar `$Sprite` aqui.

2. **`_enter_tree()`**: O nó foi conectado à SceneTree.

   - _Uso:_ Raro. Útil para ferramentas que precisam saber quando entraram no mundo.

3. **`_ready()`**: O nó E todos os seus filhos estão prontos.

   - _Uso:_ A inicialização real do jogo. Buscar nós (`$Node`), conectar sinais, configurar UI.
   - _Ordem:_ Os filhos rodam `_ready()` antes do pai. O pai é o último a ficar pronto.

4. **`_exit_tree()`**: O nó foi removido da SceneTree (mas ainda pode estar na RAM).
   - _Uso:_ Limpar conexões manuais ou salvar dados antes de morrer.

---

## 2. O Game Loop: Process vs Physics Process

A Godot tem dois relógios rodando ao mesmo tempo. Confundi-los causa "jitter" (tremedeira) e bugs de física.

### `_process(delta)`: O Relógio Visual

Roda o mais rápido possível (ex: 60, 144, 300 FPS). Varia conforme a potência do PC.

- **Uso:** Animações de UI, rotação de câmera, interpolação visual, inputs contínuos.
- **Nunca use para:** Física (mover CharacterBody, aplicar forças).

### `_physics_process(delta)`: O Relógio Físico

Roda em intervalo fixo (padrão: 60 vezes/segundo). É determinístico.

- **Uso:** `move_and_slide()`, detecção de colisão, lógica de IA.
- **Por que?** Se a física rodasse no `_process`, um PC rápido faria o personagem correr mais rápido que um PC lento.

---

## 3. O Mistério do Delta

O parâmetro `delta` (float) que você vê nas funções é o **Tempo (em segundos) que passou desde o último frame**.

Se o jogo roda a 60 FPS, o delta é `0.016` (1/60).
Se o jogo cai para 30 FPS, o delta sobe para `0.033` (1/30).

**A Regra de Ouro:**
Sempre multiplique movimento e tempo por `delta` no `_process`.

```gdscript
# Errado (Depende do FPS)
position.x += 10 # A 60 FPS anda 600px. A 120 FPS anda 1200px.

# Certo (Independe do FPS)
position.x += 100 * delta # Anda 100 pixels por segundo, sempre.
```

> **Nota:** `move_and_slide()` já aplica o delta internamente para a velocidade. Não multiplique `velocity` por delta antes de chamar `move_and_slide()`.

---

## 4. A Morte: `queue_free()`

Para destruir um objeto, nunca use `free()` (é perigoso e pode crashar se algo ainda estiver usando o objeto).

Use **`queue_free()`**.
Isso diz à Godot: "Quando terminar este frame e for seguro, delete este nó da memória". É a coleta de lixo segura e controlada.
