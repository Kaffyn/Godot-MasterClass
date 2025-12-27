# Machi Class: Gameplay Ability System (GAS)

> **Instrutor:** Machi
> **Nível:** Arquiteto (Level 5)
> **Objetivo:** Abandonar o "Script do Player que faz tudo" e adotar um sistema onde Habilidades, Buffs e Atributos são dados modulares. Entender o fluxo: Intenção -> Custo -> Execução.

---

## 1. O Que é o GAS?

Em jogos simples, quando você aperta "Espaço", o script do player faz `velocity.y = jump_force`.
Em jogos complexos (RPGs, MOBAs, Hero Shooters), isso não escala.

E se o player estiver "Stunned"? E se estiver "Silenced"? E se o pulo custar Stamina? E se um item passivo fizer o pulo causar dano de fogo em área?

O **Gameplay Ability System (GAS)** é uma arquitetura que desacopla a **Regra** da **Ação**.

- **Ability (Habilidade):** Um Resource que define _o que_ pode ser feito (Bola de Fogo, Pulo, Dash).
- **Effect (Efeito):** Um Resource que define _o que acontece_ (Dano, Cura, Stun, Buff de Velocidade).
- **AttributeSet (Atributos):** Os números vivos (Vida, Mana, Força).
- **GameplayTags:** O vocabulário do sistema (ex: `State.Stunned`, `Damage.Fire`).

---

## 2. Gameplay Tags: O Vocabulário do Jogo

Antes de codar, precisamos falar. Booleanos (`is_stunned`, `is_burning`) são o inferno da manutenção. Usamos **Tags**.
No Zyris e em sistemas robustos, Tags são identificadores hierárquicos.

Exemplo de Tags:

- `State.Alive`
- `State.Disabled.Stun`
- `Element.Fire`
- `Ability.Melee.Slash`

Em vez de verificar `if not is_stunned and not is_frozen and not is_dead`, você verifica:
`if not container.has_tag_matching("State.Disabled")`.

---

## 3. A Arquitetura: Node vs Resource

Seguindo o padrão Zyris, dividimos as responsabilidades:

### 3.1. O Executor (`AbilityContainer` - Node)

Este é o cérebro anexado ao personagem. Ele não sabe o que é uma "Bola de Fogo". Ele sabe gerenciar execução.

```gdscript
class_name AbilityContainer extends Node

# O estado atual do personagem (Tags Ativas)
var active_tags: Dictionary = {} # Tag: int (Count)
var attributes: Dictionary = {"health": 100, "mana": 50}

func try_activate_ability(ability: AbilityResource) -> bool:
    # 1. Checa Cooldown
    if not ability.can_activate(self):
        return false

    # 2. Checa Custos (Mana/Stamina)
    if not ability.check_cost(self):
        return false

    # 3. Commit (Paga o custo e inicia)
    ability.commit_cost(self)
    ability.activate(self)
    return true
```

### 3.2. A Habilidade (`AbilityResource` - Resource)

Aqui vive a lógica. Graças ao ROP, podemos criar 50 magias diferentes apenas criando arquivos `.tres`.

```gdscript
class_name AbilityResource extends Resource

@export_group("Config")
@export var id: String
@export var cooldown: float
@export var tags_required: Array[String] # Ex: ["Weapon.Sword"]
@export var tags_blocking: Array[String] # Ex: ["State.Disabled"]

@export_group("Cost")
@export var cost_attribute: String = "mana"
@export var cost_value: float = 10.0

# Função virtual que pode ser sobrescrita por scripts específicos (FireballAbility.gd)
func activate(container: AbilityContainer):
    print("Habilidade genérica ativada!")
```

---

## 4. Effects (Buffs e Debuffs)

No GAS, você nunca altera `hp -= 10` diretamente. Você aplica um **GameplayEffect**.
Por que? Porque efeitos têm **Origem**, **Duração** e **Contexto**.

### O `EffectResource`

```gdscript
class_name EffectResource extends Resource

enum Type { INSTANT, DURATION, INFINITE }
@export var type: Type = Type.INSTANT
@export var duration: float = 0.0

@export var attribute_name: String = "health"
@export var value: float = -10.0 # Dano
@export var operation: String = "add" # add, multiply, override

@export var grants_tags: Array[String] # Ex: ["State.Burning"]
```

Quando você aplica um efeito de "Veneno" (Duration, -2 HP a cada seg, grants `State.Poisoned`), o Container do alvo:

1. Adiciona a Tag `State.Poisoned`.
2. Inicia um Timer interno.
3. A cada tick, aplica o modificador de atributo.

Se o jogador tentar usar uma cura que requer `not State.Poisoned`, o sistema bloqueia automaticamente.

---

## 5. O Fluxo de Trabalho (Workflow)

1. **Design:** Crie um `AbilityResource` chamado `HeavyStrike.tres`.
   - Custo: 20 Stamina.
   - Blocking Tags: `State.Stunned`.
2. **Implementação:** O script do `HeavyStrike` instancia uma `Hitbox` que aplica um `EffectResource` de Dano.
3. **Input:** O PlayerController apenas chama:
   `$AbilityContainer.try_activate_ability(heavy_strike_res)`

O Controller não sabe se o player tem mana, se está stunnado ou se a habilidade está em cooldown. O **Sistema** sabe.

## 6. Conclusão: Por que usar isso?

1. **Multiplayer:** É muito mais fácil sincronizar "Ativou Habilidade X" e "Aplicou Efeito Y" do que sincronizar variáveis soltas.
2. **Escalabilidade:** Adicionar uma nova mecânica (ex: "Imunidade a Fogo") é apenas criar uma Tag e colocá-la na lista de "Blocking Tags" das habilidades de fogo. Não exige reescrever código de dano.
3. **Design:** Game Designers podem criar classes inteiras (Paladino, Necromante) apenas combinando Resources de Habilidades e Efeitos existentes.

Bem-vindo à engenharia de sistemas de RPG de alto nível.
