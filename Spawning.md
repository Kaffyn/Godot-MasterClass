# Curso Kaffyn: Sistemas de Spawn e Fábricas

> **Instrutor:** Machi
> **Objetivo:** Criar inimigos, balas e itens dinamicamente. Entender `instantiate()`, `add_child` e como evitar problemas de hierarquia.

---

## 1. O Básico: Instanciando Cenas

Para criar algo que não existe no editor (ex: uma bala ao apertar o gatilho), precisamos de uma referência ao seu arquivo `.tscn` (PackedScene).

```gdscript
# weapon.gd
@export var bullet_scene: PackedScene # Arraste bullet.tscn aqui

func shoot():
    if not bullet_scene: return
    
    # 1. Cria a instância na memória (ainda não visível)
    var bullet = bullet_scene.instantiate()
    
    # 2. Configura posição e rotação
    bullet.global_position = global_position
    bullet.rotation = rotation
    
    # 3. CRÍTICO: Onde adicionar?
    # Errado: add_child(bullet) -> A bala vai ser filha da Arma. Se a arma girar, a bala gira junto!
    
    # Certo: Adicionar na "Raiz do Mundo"
    get_tree().current_scene.add_child(bullet)
```

---

## 2. O Problema do `owner`

Quando você instancia algo via código, o `owner` (propriedade usada para salvar cenas) vem nulo.
Isso é normal em runtime. Só se preocupe com `owner` se estiver criando ferramentas de editor (`@tool`).

---

## 3. Spawners Inteligentes (Factory Pattern)

Não deixe a lógica de spawn espalhada. Crie nós dedicados: `EnemySpawner`.

### Exemplo: Spawner com ROP

Imagine um Spawner que pode criar Goblins ou Orcs dependendo de um Resource de configuração.

```gdscript
# enemy_spawner.gd
extends Marker2D

@export var spawn_list: Array[PackedScene]
@export var spawn_interval: float = 2.0
@export var limit: int = 5

var _active_count: int = 0

func _ready():
    $Timer.wait_time = spawn_interval
    $Timer.timeout.connect(_on_timer_timeout)
    $Timer.start()

func _on_timer_timeout():
    if _active_count >= limit: return
    
    spawn_random()

func spawn_random():
    var scene = spawn_list.pick_random()
    var enemy = scene.instantiate()
    
    enemy.global_position = global_position
    
    # Conectar sinal de morte para liberar o slot
    enemy.tree_exited.connect(func(): _active_count -= 1)
    
    get_parent().add_child(enemy)
    _active_count += 1
```

---

## 4. Pooling (Otimização)

Instanciar (`.new()`) e Destruir (`queue_free()`) custa caro para a CPU.
Se você tem um jogo "Bullet Hell" com 1000 tiros por segundo, o jogo vai travar (Garbage Collection).

**Solução: Object Pooling**
Em vez de destruir a bala, você a esconde e desliga a física.
Quando precisar atirar de novo, você pega aquela bala escondida, move para a arma e liga de novo.

**Implementação Kaffyn (Simplificada):**

```gdscript
# bullet_pool.gd
var _pool: Array[Node2D] = []
var _scene: PackedScene

func get_bullet() -> Node2D:
    if _pool.is_empty():
        return _scene.instantiate() # Cria nova se faltar
    else:
        var b = _pool.pop_back()
        b.visible = true
        b.set_physics_process(true)
        return b

func return_bullet(b: Node2D):
    b.visible = false
    b.set_physics_process(false)
    _pool.append(b)
```
*Nota: Em Godot 4, a instanciação ficou muito mais rápida, então Pooling só é obrigatório em casos extremos (Mobile ou Bullet Hells massivos).*

---

## 5. Spawning de Itens (Loot Tables)

Ao matar um inimigo, ele deve dropar algo?
Use **Resources** para definir tabelas de loot.

```gdscript
# loot_table.gd
class_name LootTable extends Resource

@export var items: Array[LootEntry] # Array de outro Resource customizado

func roll_item() -> PackedScene:
    var roll = randf() # 0.0 a 1.0
    var current = 0.0
    
    for entry in items:
        current += entry.chance
        if roll <= current:
            return entry.item_scene
            
    return null # Nada dropou
```
O Inimigo tem um `export var loot: LootTable`. Ao morrer, chama `loot.roll_item()` e instancia o resultado.
