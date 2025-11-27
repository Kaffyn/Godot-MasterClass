# Godot MBA: Áudio Dinâmico e Imersivo

> **Instrutor:** Machi
> **Objetivo:** Parar de usar sons repetitivos e estáticos. Aprender a usar o `AudioStreamRandomizer` (antigo AudioStreamRandomPitch) e `AudioBuses` para criar uma paisagem sonora profissional.

---

## 1. O Problema do "Efeito Metralhadora"

Em jogos de tiro ou com passos rápidos, tocar o EXATO mesmo arquivo `.wav` repetidamente cria um efeito artificial e irritante chamado "Machine Gun Effect".
O cérebro humano detecta padrões rapidamente.

**A Solução Antiga:**
Criar um script que sorteia um pitch aleatório antes de dar play.

**A Solução Godot 4:**
Usar `AudioStreamRandomizer`.

---

## 2. AudioStreamRandomizer

Este é um **Resource** especial que embrulha seus sons. Ele permite variações automáticas de Pitch (tom) e Volume sem escrever uma linha de código.

### Como Criar:

1. No FileSystem, crie um novo Resource -> `AudioStreamRandomizer`.
2. Salve como `footstep_grass.tres`.
3. No Inspector do Resource:
   - **Streams:** Adicione seus arquivos de áudio (pode ser 1 ou vários variações de passos).
   - **Random Pitch:** Defina a escala (ex: `1.1`). Isso significa que o som pode tocar 10% mais grave ou mais agudo a cada play.
   - **Random Volume:** Defina uma variação (ex: `2.0` dB).

### Como Usar:

No seu `AudioStreamPlayer`, em vez de arrastar o `.wav`, arraste o `footstep_grass.tres`. Pronto.

---

## 3. Sistema de Áudio Global (SoundManager)

Não coloque `AudioStreamPlayer` dentro de balas ou objetos que morrem rápido. Se o objeto for deletado (`queue_free`), o som corta.

Crie um Autoload `SoundManager`.

```gdscript
# sound_manager.gd
extends Node

# Pool de players para sons 2D (espaciais) e UI (globais)
func play_sfx(stream: AudioStream, pitch_scale: float = 1.0):
    var player = AudioStreamPlayer.new()
    player.stream = stream
    player.pitch_scale = pitch_scale
    player.bus = "SFX"
    add_child(player)
    player.play()
    # Auto-destruição após tocar
    player.finished.connect(player.queue_free)

func play_sfx_2d(stream: AudioStream, position: Vector2):
    var player = AudioStreamPlayer2D.new()
    player.stream = stream
    player.global_position = position
    player.bus = "SFX"
    # Configurar atenuação (distância máxima)
    player.max_distance = 500
    get_tree().current_scene.add_child(player)
    player.play()
    player.finished.connect(player.queue_free)
```

---

## 4. Audio Buses (Mixagem)

Nunca deixe todos os sons no Master.
Vá na aba **Audio** (parte inferior do editor) e crie Buses:

- **Master**
  - **Music** (Para trilha sonora)
  - **SFX** (Efeitos gerais)
    - **Ambience** (Vento, chuva)
    - **UI** (Cliques)
  - **Voice** (Diálogos)

**Por que?**

1. **Menu de Opções:** Permite que o jogador baixe só a música.
2. **Ducking (Sidechain):** Você pode adicionar um efeito "Compressor" no Bus da Música e fazer ele baixar o volume automaticamente quando o Bus de Voz tiver sinal. (Estilo rádio locutor).
3. **Reverb Local:** Crie um Bus "ReverbCave". Quando o player entrar na caverna, redirecione os sons dele para esse Bus via script (`player_audio.bus = "ReverbCave"`).

---

## 5. Música Dinâmica (Interativa)

Para trocar de música (Exploração -> Combate) sem cortes bruscos, use `AnimationPlayer` ou `Tweens` para controlar o volume dos Buses ou dos Players.

```gdscript
func enter_combat_mode():
    var tween = create_tween()
    # Fade out exploração
    tween.tween_property($MusicExploration, "volume_db", -80.0, 2.0)
    # Fade in combate
    tween.parallel().tween_property($MusicCombat, "volume_db", 0.0, 2.0)
```

Ou use o recurso **Interactive Music** se estiver usando ferramentas de middleware como FMOD/Wwise (avançado), mas para Godot puro, Crossfading é a chave.
