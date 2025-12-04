# Machi MBA: Capstone - A Ficha de RPG Suprema

> **Instrutor:** Machi
> **Nível:** Avançado (Integração de Sistemas)
> **Objetivo:** Criar uma tela de criação de personagens completa. Vamos misturar ROP (para os dados), UI (para a interação), i18n (para tradução), Tooltips (UX) e Save System.
>
> **A Filosofia:** A UI não guarda dados. A UI apenas **visualiza** e **modifica** um Resource temporário. Quando o jogador clica em "Salvar", persistimos esse Resource no disco.

---

## 1. A Fundação: Dados (ROP)

Antes de colocar um único botão na tela, precisamos definir **o que** é um personagem. Não use variáveis soltas no script da UI. Use um `Resource`.

### 1.1. A Classe (Job)

Primeiro, definimos o que é uma "Classe" (Guerreiro, Mago). Isso é dado estático (Design).

```gdscript
# class_data.gd
class_name ClassData extends Resource

@export var id: String = "warrior"
@export var display_name: String = "CLASS_WARRIOR" # Chave i18n
@export_multiline var description: String = "DESC_WARRIOR" # Chave i18n
@export var icon: Texture2D
@export var base_strength: int = 5
@export var base_magic: int = 0
```

### 1.2. O Personagem (Character Sheet)

Agora, o container que será preenchido pelo jogador.

```gdscript
# character_sheet.gd
class_name CharacterSheet extends Resource

@export var name: String = "Hero"
@export var level: int = 1
@export var job: ClassData # Injeção da classe escolhida

# Atributos distribuídos
@export var strength: int = 0
@export var dexterity: int = 0
@export var intelligence: int = 0

# Visual
@export var skin_color: Color = Color.WHITE

# Lógica de validação
func get_total_strength() -> int:
    return strength + (job.base_strength if job else 0)
```

---

## 2. A Interface (UI) e Tooltips

Vamos montar uma cena `CharacterCreation.tscn`.
Estrutura sugerida:

- `HBoxContainer`
  - `VBoxContainer` (Esquerda: Inputs)
    - `LineEdit` (Nome)
    - `OptionButton` (Seleção de Classe)
    - `SpinBox` (Força, Dex, Int)
  - `CenterContainer` (Direita: Preview)
    - `TextureRect` (Sprite do Personagem)
  - `PanelContainer` (Rodapé: Tooltip Area)
    - `Label` (TooltipText)

### 2.1. O Sistema de Tooltip (UX)

Em vez de usar o tooltip nativo (que demora a aparecer), vamos criar uma área de informação dinâmica.

**O Segredo:** Conectar os sinais `mouse_entered` e `mouse_exited` de **cada** componente interativo.

```gdscript
# creation_screen.gd
extends Control

@onready var tooltip_label: Label = %TooltipLabel
@onready var name_input: LineEdit = %NameInput
@onready var class_selector: OptionButton = %ClassSelector

# Textos i18n para as dicas
const TIPS = {
    "name": "TIP_ENTER_NAME",
    "class": "TIP_CHOOSE_CLASS",
    "str": "TIP_STAT_STR"
}

func _ready():
    # Conexão manual (ou via editor)
    _connect_tooltip(name_input, TIPS["name"])
    _connect_tooltip(class_selector, TIPS["class"])

func _connect_tooltip(node: Control, text_key: String):
    # Usamos lambda (função anônima) para passar o texto
    node.mouse_entered.connect(func(): tooltip_label.text = tr(text_key))
    node.mouse_exited.connect(func(): tooltip_label.text = "")
```

---

## 3. Lógica de Controle e Pontos

O jogador tem um "pool" de pontos para gastar. A UI deve impedir que ele gaste mais do que tem.

```gdscript
# creation_screen.gd (continuação)

var temp_sheet: CharacterSheet # O Resource que estamos editando
var available_points: int = 10

func _ready():
    temp_sheet = CharacterSheet.new() # Começa em branco
    _update_ui()

func _on_strength_changed(new_value: float):
    var current = temp_sheet.strength
    var diff = int(new_value) - current

    if available_points - diff >= 0:
        temp_sheet.strength = int(new_value)
        available_points -= diff
    else:
        # Bloqueia o aumento se não tiver pontos
        %StrengthSpinBox.value = current
        flash_error("MSG_NO_POINTS")

    _update_ui()

func _update_ui():
    %PointsLabel.text = str(available_points)
    # Atualiza preview baseado na classe e cor
    if temp_sheet.job:
        %PreviewSprite.texture = temp_sheet.job.icon
    %PreviewSprite.modulate = temp_sheet.skin_color
```

---

## 4. Internacionalização (i18n)

Note que em nenhum momento escrevemos "Guerreiro" ou "Força" no código.
No seu arquivo `pt.csv` (tradução), você teria:

```csv
keys,en,pt
CLASS_WARRIOR,Warrior,Guerreiro
DESC_WARRIOR,"Hits hard with swords.","Bate forte com espadas."
TIP_STAT_STR,"Increases physical damage.","Aumenta dano físico."
MSG_NO_POINTS,"Not enough points!","Pontos insuficientes!"
```

O Godot faz a troca automaticamente se você usar `text = tr("CHAVE")` ou configurar no Inspector.

---

## 5. Persistência: Salvando a Ficha

O jogador clicou em "Start Game". O que fazemos?
Transformamos nosso `temp_sheet` (Resource) em um arquivo no disco.

**Machi Rule:** Não salve na pasta `res://` (é read-only no jogo exportado). Salve em `user://`.

```gdscript
func _on_save_pressed():
    if name_input.text.is_empty():
        flash_error("MSG_NAME_REQUIRED")
        return

    temp_sheet.name = name_input.text

    # Opcional: Converter para Dictionary se preferir JSON
    # Mas como estamos usando ROP puro, podemos salvar o Resource direto!

    var save_path = "user://save_slot_1.tres"
    var error = ResourceSaver.save(temp_sheet, save_path)

    if error == OK:
        print("Personagem salvo com sucesso!")
        change_scene_to_game()
    else:
        push_error("Falha ao salvar: " + str(error))
```

### Carregando depois (No jogo)

```gdscript
# game_manager.gd (Autoload)

var current_player_data: CharacterSheet

func load_game():
    if FileAccess.file_exists("user://save_slot_1.tres"):
        # Safe Cast é crucial aqui!
        current_player_data = load("user://save_slot_1.tres") as CharacterSheet

        if current_player_data:
            spawn_player(current_player_data)
```

---

## Resumo da Aula

1. **Dados:** Criamos `CharacterSheet` e `ClassData` como Resources.
2. **UI:** Criamos um Control que manipula uma instância temporária de `CharacterSheet`.
3. **UX:** Usamos sinais `mouse_entered` para Tooltips informativos.
4. **Logica:** Validamos a distribuição de pontos antes de aplicar ao Resource.
5. **Save:** Usamos `ResourceSaver` para persistir o objeto final em `user://`.

Você acabou de criar um sistema profissional, traduzível e escalável. Se quiser adicionar uma nova classe "Necromante" amanhã, basta criar um novo `.tres` de `ClassData` e adicionar na lista do `OptionButton`. Zero código alterado.
