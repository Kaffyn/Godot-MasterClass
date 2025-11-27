# Godot MBA: Animação & Motion

> **Instrutor:** Machi
> **Objetivo:** Dominar as ferramentas de movimento da Godot. Entender a diferença entre `AnimationPlayer` e `Tweens` e quando usar cada um.

---

## 1. AnimationPlayer vs Tweens

| Ferramenta          | Melhor Uso                                                                                  | Exemplo                                                                              |
| :------------------ | :------------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| **AnimationPlayer** | Animações complexas, visuais, com keyframes fixos, sincronia de áudio e chamadas de função. | Walk cycles, Ataques, Cutscenes, UI complexa de entrada.                             |
| **Tweens**          | Animações procedurais, interpolações numéricas, movimentos simples "Fire-and-forget".       | Fade in/out de um item coletado, Tremer a câmera, Barra de vida descendo suavemente. |

---

## 2. O Poder do AnimationPlayer

O `AnimationPlayer` é o editor de vídeo da Godot. Ele anima QUALQUER propriedade exportada.

### Call Method Track (A arma secreta)

Além de animar Posição e Rotação, você pode animar Funções.

- **Cenário:** Um ataque de espada.
- **Problema:** O dano deve ocorrer exatamente no frame 12, quando a espada atinge o ápice.
- **Solução:** Adicione uma "Call Method Track" no AnimationPlayer, aponte para o script do Player e insira uma chave no tempo 0.4s chamando a função `apply_damage()`.

### AnimationTree (Estados de Animação)

Para personagens, não controle o AnimationPlayer via código (`play("run")`). Use uma `AnimationTree`.

- **StateMachine:** Cria um grafo visual (Idle <-> Run <-> Jump).
- **BlendSpace1D/2D:** Mistura animações baseado em vetores (Ex: Idle mistura com Walk que mistura com Run dependendo da velocidade 0 a 100).

---

## 3. Tweens (Scripted Animation)

Tweens são leves e poderosos para "Game Feel".
Em Godot 4, Tweens não são mais Nodes. São objetos criados on-the-fly.

```gdscript
func collect_item():
    # Animação de "pulo" do item antes de sumir
    var tween = create_tween()

    # Sobe e Escala
    tween.tween_property(self, "position:y", position.y - 50, 0.3).set_trans(Tween.TRANS_SINE).set_ease(Tween.EASE_OUT)
    tween.parallel().tween_property(self, "scale", Vector2(0, 0), 0.3).set_delay(0.2)

    # Callback ao terminar
    tween.tween_callback(queue_free)
```

### Easing e Transições

O segredo do polimento ("Juice") está nas curvas.

- **Linear:** Chato, robótico.
- **Sine/Cubic:** Suave, natural.
- **Elastic/Bounce:** Cartunesco, divertido.

Use `set_trans()` e `set_ease()` em todo Tween.

---

## 4. Screen Shake (A Arte de Tremer)

Screen Shake não deve mover a Câmera diretamente, ou você perde o controle do "offset" do jogador.
Use um `Camera2D` com um nó filho ou propriedade dedicada para "Shake Offset".

```gdscript
# camera_shaker.gd
extends Camera2D

var shake_strength: float = 0.0
@export var fade_speed: float = 5.0

func apply_shake(strength: float):
    shake_strength = strength

func _process(delta):
    if shake_strength > 0:
        # Reduz a força com o tempo
        shake_strength = lerpf(shake_strength, 0, fade_speed * delta)

        # Aplica offset aleatório
        offset = Vector2(
            randf_range(-shake_strength, shake_strength),
            randf_range(-shake_strength, shake_strength)
        )
```

---

## 5. Transições de Cena

Nunca troque de cena seco (`change_scene`). Use um singleton `TransitionLayer`.

1. Crie uma cena com um `ColorRect` (preto) cobrindo a tela.
2. Use `AnimationPlayer` para animar a propriedade `modulate:a` de 0 a 1 (Fade Out) e 1 a 0 (Fade In).
3. Use `await` no seu SceneLoader.

```gdscript
func change_scene(path):
    await TransitionLayer.fade_out()
    get_tree().change_scene_to_file(path)
    await TransitionLayer.fade_in()
```
