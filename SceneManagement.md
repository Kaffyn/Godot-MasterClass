# Godot MBA: Gerenciamento de Cenas (Scene Flow)

> **Instrutor:** Machi
> **Objetivo:** Controlar o fluxo do jogo. Loading Screens, transições suaves e carregamento em background.

---

## 1. O Básico: `change_scene_to_file`

Para protótipos, `get_tree().change_scene_to_file("res://level2.tscn")` funciona.
**Problema:** O jogo trava (congela) enquanto carrega a nova cena. Se for pesada, o jogador acha que crashou.

---

## 2. Carregamento Assíncrono (Background Loading)

A Godot tem a classe `ResourceLoader` que permite carregar em uma thread separada.

### O `SceneLoader` (Singleton)

Vamos criar um Autoload robusto.

```gdscript
# scene_loader.gd
extends Node

var _loading_path: String = ""
var _loading_status: int = 0
var _progress: Array = []

func load_scene(path: String):
    _loading_path = path
    # Inicia o carregamento em background
    ResourceLoader.load_threaded_request(path)

    # Troca para uma cena de "Loading..." temporária
    get_tree().change_scene_to_file("res://ui/loading_screen.tscn")

    # Liga o processamento para checar o progresso a cada frame
    set_process(true)

func _process(delta):
    if _loading_path == "":
        set_process(false)
        return

    # Consulta o status atual
    _loading_status = ResourceLoader.load_threaded_get_status(_loading_path, _progress)

    # Atualiza a barra de progresso (se existir na cena atual)
    # _progress[0] vai de 0.0 a 1.0
    var loading_screen = get_tree().current_scene
    if loading_screen.has_method("update_bar"):
        loading_screen.update_bar(_progress[0])

    if _loading_status == ResourceLoader.THREAD_LOAD_LOADED:
        # Terminou!
        set_process(false)

        # Pega o recurso carregado
        var new_scene_resource = ResourceLoader.load_threaded_get(_loading_path)

        # Troca a cena manualmente
        get_tree().change_scene_to_packed(new_scene_resource)
        _loading_path = ""
```

---

## 3. Estrutura de Mundos (World vs UI)

Em jogos complexos, você não quer destruir TUDO ao trocar de fase.
Exemplo: O HUD e o Player podem persistir.

**Estrutura da Main.tscn:**

```
Main (Node)
├── UILayer (CanvasLayer) -> Nunca é destruído
├── MusicLayer (Node) -> Nunca é destruído
└── WorldContainer (Node) -> Aqui trocamos as fases
    └── Level01 (Node3D)
```

### Script de Troca Local

```gdscript
func change_level(level_packed: PackedScene):
    # Remove fase antiga
    var old_level = $WorldContainer.get_child(0)
    old_level.queue_free()

    # Adiciona nova
    var new_level = level_packed.instantiate()
    $WorldContainer.add_child(new_level)
```

Isso mantém a música tocando sem cortes e o HUD intacto.

---

## 4. Passando Dados entre Cenas

Como dizer para o "Level 2" que o player deve spawnar na "Porta B" e não na "Porta A"?
Não use globais (`Global.next_spawn_point`).

Use um **Payload** ou **Contexto**.

Se estiver usando o `SceneLoader` acima, adicione um parâmetro `params: Dictionary`.
Após instanciar a nova cena, injete os dados.

```gdscript
# No SceneLoader
func _on_load_complete(resource):
    var new_scene = resource.instantiate()

    # Injeção de Dependência
    if new_scene.has_method("setup"):
        new_scene.setup(_saved_params)

    get_tree().root.add_child(new_scene)
    get_tree().current_scene = new_scene
```

**No Level Script:**

```gdscript
func setup(params):
    var spawn_id = params.get("spawn_point", 0)
    var point = get_node("Spawns/Point_" + str(spawn_id))
    $Player.global_position = point.global_position
```
