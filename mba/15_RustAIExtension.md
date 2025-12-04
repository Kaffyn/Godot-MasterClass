# ColonyCore: Arquitetura de IA com Rust (Deep Dive)

> **Instrutor:** Machi
> **Objetivo:** Projetar e implementar um "cérebro" de IA de alta performance em Rust, capaz de gerenciar centenas de entidades com comportamentos complexos (estilo _RimWorld_ / _Dwarf Fortress_). Esta aula é o ponto de virada do "arrastar nós" para a verdadeira engenharia de sistemas de jogo.

---

## Módulo 1: A Arquitetura - O Cérebro Desacoplado

Antes de escrever uma linha de código, precisamos entender a filosofia. A arquitetura padrão da Godot, baseada em Nós que guardam seu próprio estado, quebra em escala massiva.

### 1.1. O Problema: A Crise de Identidade do Nó

Em uma simulação complexa, um `Pawn.tscn` (nosso colono) precisa ser muitas coisas:

- Ele **tem** fome.
- Ele **tem** uma posição.
- Ele **tem** um trabalho.
- Ele **tem** relacionamentos.

A abordagem ingênua é colocar tudo isso no script `Pawn.gd`. Para 500 Pawns, isso significa 500 `_process` loops, 500 timers, e uma teia de aranha de `get_node()` e sinais para fazer um Pawn conversar com o outro. É um pesadelo de performance e manutenção.

### 1.2. A Solução: Data-Oriented Design & ECS

Abandonamos a ideia de que o Nó "é" o Pawn. Adotamos uma arquitetura de "Cérebro Desacoplado":

```
         [ RUST (O Cérebro) ]                     [ GODOT (O Teatro) ]
+-------------------------------------+      +-----------------------------+
|                                     |      |                             |
|  struct World {                     |      |   SceneTree                 |
|      Vec<Position>  positions;      |      |    └─ Pawn.tscn (Sprite2D)  |
|      Vec<Hunger>    hunger_levels;  |      |    └─ Pawn.tscn (Sprite2D)  |
|      Vec<Energy>    energy_levels;  |      |    └─ ... (500x)            |
|  }                                  |      |                             |
|                                     |      |                             |
|  // Lógica pura em dados:           |      |   // Lógica visual:         |
|  fn process_hunger(world) { ... }   | <--- |   Pawn.gd:                  |
|                                     |      |     func _process():        |
|                                     |      |       var data = Brain.get()|
|                                     |      |       position = data.pos   |
+-------------------------------------+      +-----------------------------+
```

- **Rust (O Cérebro):** Gerencia todos os dados em arrays compactos e eficientes. A lógica (Sistemas) itera sobre esses arrays. Rust não sabe o que é um `Sprite2D`.
- **Godot (O Teatro):** É um "renderizador burro". Os nós `Pawn.tscn` são apenas marionetes. Eles não pensam; eles apenas se posicionam e animam com base nos dados que recebem do Cérebro.

Para implementar essa arquitetura em Rust, usaremos um **Entity Component System (ECS)**.

---

## Módulo 2: Configurando o Ecossistema (Rust + Godot)

Vamos fazer as duas ferramentas se falarem.

1. **Crie a Biblioteca Rust:**
    No diretório raiz do seu projeto Godot, rode:

    ```sh
    cargo new --lib colony_brain
    ```

2. **Configure o `Cargo.toml`:**
    Abra `colony_brain/Cargo.toml` e adicione:

    ```toml
    [package]
    name = "colony_brain"
    version = "0.1.0"
    edition = "2021"

    [lib]
    crate-type = ["cdylib"] # Compila como biblioteca dinâmica

    [dependencies]
    # A ponte oficial para a Godot
    godot = { git = "https://github.com/godot-rust/gdext.git", branch = "master" }

    # A biblioteca ECS que usaremos como cérebro
    bevy_ecs = "0.12"
    ```

3. **Crie a Ponte (`.gdextension`):**
    Dentro da pasta do projeto **Godot**, crie o arquivo `colony_brain.gdextension` com o seguinte conteúdo. Ele diz à Godot como carregar sua biblioteca.

    ```ini
    [configuration]
    entry_symbol = "gdext_rust_init"
    compatibility_minimum = "4.2"

    [libraries]
    windows.64 = "res://../colony_brain/target/release/colony_brain.dll"
    linux.64 = "res://../colony_brain/target/release/libcolony_brain.so"
    macos.64 = "res://../colony_brain/target/release/libcolony_brain.dylib"
    ```

---

## Módulo 3: O Cérebro ECS (Entidades, Componentes, Sistemas)

Agora, a lógica em Rust. Dentro de `colony_brain/src/lib.rs`.

### 3.1. Definindo os Componentes

Componentes são **apenas dados**. Structs puras.

```rust
use godot::prelude::*;
use bevy_ecs::prelude::*;

// Componente para Posição no mundo 2D
#[derive(Component)]
struct Position(Vector2);

// Componente para a necessidade "Fome"
#[derive(Component)]
struct Hunger {
    /// Nível de 0.0 (faminto) a 1.0 (saciado)
    level: f32,
    /// Taxa de perda por segundo
    rate: f32,
}

// Marcador para um objeto que pode ser comido
#[derive(Component)]
struct Food;
```

### 3.2. Definindo os Sistemas

Sistemas são **apenas lógica**. Funções que operam sobre os componentes.

```rust
// Sistema que atualiza a fome de todas as entidades que têm o componente Hunger
fn apply_hunger_system(mut query: Query<&mut Hunger>, time: Res<Time>) {
    let delta = time.delta_seconds();
    for mut hunger in query.iter_mut() {
        hunger.level -= hunger.rate * delta;
        if hunger.level < 0.0 {
            hunger.level = 0.0;
        }
    }
}

// Sistema de IA: entidades famintas buscam a comida mais próxima
fn find_food_system(
    mut pawn_query: Query<(Entity, &Position, &mut Target), With<Hunger>>,
    food_query: Query<(Entity, &Position), With<Food>>
) {
    for (pawn_entity, pawn_pos, mut pawn_target) in pawn_query.iter_mut() {
        // Se já tem um alvo, não faz nada
        if pawn_target.entity.is_some() { continue; }

        // Encontra a comida mais próxima
        let mut closest_food = None;
        let mut min_dist_sq = f32::MAX;

        for (food_entity, food_pos) in food_query.iter() {
            let dist_sq = pawn_pos.0.distance_squared_to(food_pos.0);
            if dist_sq < min_dist_sq {
                min_dist_sq = dist_sq;
                closest_food = Some(food_entity);
            }
        }

        if let Some(entity) = closest_food {
            pawn_target.entity = Some(entity);
        }
    }
}

#[derive(Component)]
struct Target {
    entity: Option<Entity>
}
```

### 3.3. Montando o Cérebro (BrainServer)

Esta é a classe que será exposta à Godot. Ela gerencia o `Mundo ECS`.

```rust
#[derive(GodotClass)]
#[class(base=Node)]
struct BrainServer {
    world: World,
    schedule: Schedule,
}

#[godot_api]
impl BrainServer {
    #[func]
    fn process_tick(&mut self, delta: f64) {
        // Atualiza o tempo no mundo ECS
        self.world.insert_resource(Time { delta_seconds: delta as f32 });
        // Roda todos os sistemas
        self.schedule.run(&mut self.world);
    }

    // ... outras funções para Godot interagir
}

#[godot_api]
impl INode for BrainServer {
    fn new_alloc() -> Gd<Self> {
        let mut world = World::new();
        // Adicionar recursos e entidades iniciais aqui...

        let mut schedule = Schedule::new();
        schedule.add_systems((apply_hunger_system, find_food_system));

        Gd::from_init_fn(|base| {
            Self { world, schedule }
        })
    }
}
```

---

## Módulo 4: Compilando o Cérebro

Com o código Rust pronto, voltamos ao terminal, na pasta `colony_brain`, e compilamos para produção:

```sh
cargo build --release
```

Este comando gera o arquivo `.dll`/`.so` na pasta `target/release/`, que a Godot irá carregar automaticamente na próxima vez que você abrir o editor.

---

## Módulo 5: Dando Vida ao Teatro (Godot)

Agora, na Godot, o trabalho é mínimo.

1. **Configure o Singleton:**
    Vá em `Project -> Project Settings -> Autoload`. Adicione um novo Autoload. Em `Path`, selecione `colony_brain.gdextension`. Em `Node Name`, coloque `BrainServer`. Marque "Enable".

2. **Crie o `Pawn.tscn`:**
    Uma cena simples com um `Sprite2D`. O script `Pawn.gd` é surpreendentemente simples:

    ```gdscript
    extends Sprite2D

    # O ID deste Pawn no mundo ECS do Rust.
    var brain_id: int

    func _ready():
        # Registra este Pawn no Cérebro e guarda seu ID.
        brain_id = BrainServer.create_pawn(self.position)

    func _process(delta):
        # Apenas busca os dados atualizados do Cérebro.
        var latest_data = BrainServer.get_pawn_data(brain_id)
        if latest_data.is_empty():
            return

        # O Nó não pensa, apenas obedece.
        self.position = latest_data["position"]

        var new_color = latest_data["hunger_color"]
        self.modulate = Color(new_color[0], new_color[1], new_color[2])
    ```

3. **Crie a Comida (`Food.tscn`):**
    Um sprite simples que, no `_ready()`, se registra no `BrainServer.create_food(position)`.

---

## Módulo 6: O Benchmark - A Prova do Poder

A cena final de teste: um script que instancia 500 `Pawn.tscn` e 50 `Food.tscn` em posições aleatórias.

Em um `Label` na tela, mostramos o resultado de uma função do `BrainServer`:

```gdscript
# ui.gd
func _process(delta):
    var stats = BrainServer.get_performance_stats()
    $Label.text = "Tick Time (500 Pawns): %.4f ms" % stats["last_tick_ms"]
```

**Resultado Esperado:**
O tempo de processamento para a IA de 500 peões se manterá na casa dos **0.1 a 0.5 milissegundos**, provando que o Cérebro Rust pode escalar para milhares de entidades sem impactar o FPS da Godot.

Este é o poder da arquitetura desacoplada. Godot faz o que faz de melhor (renderizar e iterar rápido no gameplay), e Rust faz o que faz de melhor (processar números e gerenciar memória com segurança e velocidade máxima).
