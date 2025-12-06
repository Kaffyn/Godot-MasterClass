# Machi Class: Testes e Qualidade (QA) - Garantindo a Robustez do seu Jogo

> **De:** Machi
> **Para:** Você, que busca excelência e evita surpresas desagradáveis.
>
> Em engenharia, não basta que algo funcione. Precisa funcionar _sempre_. E funcionar _bem_. No desenvolvimento de jogos, onde a complexidade é intrínseca, a disciplina de testes é o seu escudo contra o caos. Este módulo não é sobre "achar bugs", mas sobre construir sistemas inerentemente robustos e confiáveis.

---

## 1. Por que Testar? A Mentalidade do Arquiteto

O custo de um bug aumenta exponencialmente com o tempo. Um erro encontrado na fase de design custa 1. Na implementação, 10. Na QA, 100. No cliente final, 1000. Testar é um investimento, não um gasto.

- **Validação de Design**: Testes forçam você a pensar sobre o comportamento esperado de cada componente.
- **Garantia de Manutenção**: Mudar uma parte do código se torna menos arriscado, pois os testes existentes avisarão se algo quebrou em outro lugar.
- **Confiança no Código**: A equipe tem certeza de que o que foi construído realmente funciona, liberando tempo para novas features em vez de caça a bugs antigos.

## 2. O Que Testar? Tipos de Testes no Contexto Godot

### 2.1. Testes Unitários

- **Foco**: A menor parte testável de um sistema (uma função, um método, uma classe). Testados isoladamente.
- **No Godot**: Perfeitos para:
  - **Funções de utilidade e matemática pura**.
  - **A Lógica em `Resources`**: Graças à ROP, Resources são puramente dados com lógica mínima e autocontida. Testá-los é fácil e eficiente, pois não dependem da `SceneTree`, input ou renderização.
  - **Componentes sem estado visual/físico**.
- **Ferramenta Principal**: GUT (Godot Unit Test framework).

_Exemplo: Testando uma função de cálculo de dano crítico de um `WeaponData` (Resource):_

```gdscript
# test_weapon_data.gd (usando GUT)
extends "res://addons/gut/test.gd"

var weapon_data_instance: WeaponData

func before_each():
    weapon_data_instance = WeaponData.new()
    weapon_data_instance.base_damage = 10

func test_critical_hit_damage():
    var critical_damage = weapon_data_instance.calculate_critical_hit()
    assert_eq(critical_damage, 20, "Dano crítico deve ser o dobro do base.")

func test_normal_hit_damage():
    var normal_damage = weapon_data_instance.calculate_normal_hit()
    assert_eq(normal_damage, 10, "Dano normal deve ser igual ao base.")
```

### 2.2. Testes de Integração

- **Foco**: Como diferentes unidades trabalham juntas. Testar a interação entre dois ou mais componentes ou subsistemas.
- **No Godot**:
  - Interação entre `Player` e `HealthComponent`.
  - Um `Enemy` detectando um `Player` via `Area2D`.
  - Um `Autoload` recebendo sinais de múltiplas fontes.
- **Ferramenta Principal**: GUT, mas orquestrando múltiplos nós em uma cena de teste.

_Exemplo: Testando se o `Player` toma dano corretamente de uma `SpikeTrap`:_

```gdscript
# test_player_damage.gd (usando GUT)
extends "res://addons/gut/test.gd"

var player_scene: PackedScene = preload("res://player.tscn")
var spike_trap_scene: PackedScene = preload("res://spike_trap.tscn")
var player_instance: Node
var spike_trap_instance: Node

func before_each():
    player_instance = player_scene.instantiate()
    get_tree().root.add_child(player_instance)
    spike_trap_instance = spike_trap_scene.instantiate()
    get_tree().root.add_child(spike_trap_instance)

func after_each():
    player_instance.queue_free()
    spike_trap_instance.queue_free()

func test_player_takes_damage_from_spike_trap():
    var initial_health = player_instance.health_component.current_health
    # Posiciona player na armadilha
    player_instance.global_position = spike_trap_instance.global_position

    # Simula o processamento físico para a colisão ocorrer
    get_tree().physics_process(0.1)

    var final_health = player_instance.health_component.current_health
    assert_lt(final_health, initial_health, "Player deveria ter tomado dano da armadilha.")
```

### 2.3. Testes Funcionais / End-to-End (E2E)

- **Foco**: Testar o sistema como um todo do ponto de vista do usuário. Simular interações e verificar o resultado final.
- **No Godot**: Geralmente são mais manuais ou feitos com scripts que simulam inputs e navegam por cenas.

### 2.4. Testes de Performance

- **Foco**: Identificar gargalos e garantir que o jogo roda dentro das especificações de FPS.
- **No Godot**: Usar o **Profiler** e os **Monitores de Performance** nativos da Godot.

## 3. Escrevendo Código Testável (O Jeito Machi)

A testabilidade não é um "extra" que você adiciona no final; ela é um subproduto de uma boa arquitetura.

- **3.1. Desacoplamento Radical**: Minimizar dependências diretas. Seu `Player` não deve chamar `get_node("/root/HUD/HealthBar")`. Em vez disso:

  - Use **Sinais**: `player.health_changed.emit(new_hp)` e a `HealthBar` se conecta.
  - Use **Autoloads**: `GameMachine.current_state` para lógica global.
  - Use **Interfaces Implícitas**: Se um objeto precisa "atacar", o invés de `if body is Enemy: body.take_damage()`, use `if body.has_method("take_damage"): body.take_damage()`.

- **3.2. Funções Puras**: Sempre que possível, escreva funções que, para os mesmos inputs, retornam sempre os mesmos outputs, sem efeitos colaterais (sem mudar variáveis globais ou de classe).
  _Exemplo:_ Uma função que calcula o dano final baseado em ataque e defesa.

- **3.3. ROP e Dados Puros**: A Programação Orientada a Resources (ROP) é a maior aliada da testabilidade. `Resources` são intrinsecamente fáceis de testar em isolamento porque são apenas dados (e lógica autocontida sobre esses dados), sem dependências de `SceneTree` ou `Input`.

## 4. Ferramentas de Teste em Godot

### 4.1. GUT (Godot Unit Test)

É o framework de testes mais popular e robusto para Godot.

- **Instalação**: Adicione o `gut` addon ao seu projeto.
- **Uso Básico**: Escreva scripts de teste que herdam de `test.gd`. Use `assert_eq()`, `assert_true()`, `assert_false()`, etc.
- **Rodar Testes**: Pelo terminal (`gut.sh`) ou diretamente pelo editor Godot.

### 4.2. Godot's Debugger e Profiler

Ferramentas integradas para encontrar bugs em runtime e identificar gargalos de performance.

- **Debugger**: `print_debug()`, breakpoints, inspeção de variáveis.
- **Profiler**: Identifica quais funções estão consumindo mais CPU/GPU por frame.
- **Monitores de Performance**: Visualize variáveis customizadas em tempo real.

### 4.3. Linter (GDScript)

A análise estática do seu código. O linter identifica erros comuns, inconsistências de estilo e potenciais bugs antes mesmo de você rodar o jogo. Mantenha seu projeto com **Zero Warnings**.

## Conclusão: Testar é Construir Confiança

Testes não são uma barreira para a criatividade, mas um facilitador. Eles libertam você para experimentar, refatorar e inovar, sabendo que sua base está sólida. Um jogo bem testado é um jogo que você pode evoluir com confiança.
