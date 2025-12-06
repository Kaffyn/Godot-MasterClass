# Machi Class: A Arte do Hash Map (De O(N) a O(1))

> **De:** Machi
> **Nível:** Arquiteto (Level 4)
> **Assunto:** Estruturas de Dados de Alta Performance

Se você quer deixar de ser um "scripter" e se tornar um Engenheiro de Software, você precisa entender como os dados são organizados na memória. Hoje vamos falar sobre a estrutura de dados mais importante para performance em jogos: o **Hash Map** (ou Dicionário).

---

## 1. O Que é um Hash Map? (Fundamentos)

Imagine que você é um bibliotecário.

- **A Abordagem da Lista (Array) - O(N):**
  Alguém pede o livro "O Senhor dos Anéis". Você tem uma pilha de 10.000 livros no chão. Você precisa pegar um por um, ler o título, e ver se é o que você quer.

  - Se o livro for o último da pilha, você leu 10.000 títulos.
  - Se a pilha dobrar de tamanho, seu trabalho dobra. Isso é **O(N)** (Linear).

- **A Abordagem do Hash Map - O(1):**
  Você tem uma estante mágica com 10.000 nichos numerados.
  Quando alguém pede "O Senhor dos Anéis", você joga o título numa calculadora mágica (Função de Hash) e ela cospe o número: `42`.
  Você vai direto no nicho 42 e o livro está lá.
  - Não importa se tem 10 livros ou 10 milhões. Você sempre vai direto ao lugar certo. Isso é **O(1)** (Constante).

### 1.1. O Conceito de "Hash" (Impressão Digital)

Um "Hash" é um número inteiro gerado a partir de qualquer dado.

- `hash("Machi")` -> `849201`
- `hash("Godot")` -> `112358`

A regra de ouro é: **O mesmo dado sempre gera o mesmo Hash.**

### 1.2. Tipos de Hash

- **Hash Criptográfico (SHA-256, MD5):** Lento, seguro, impossível de reverter. Usado para senhas.
- **Hash Rápido (MurmurHash, FNV-1a):** Extremamente rápido, focado em distribuição uniforme. Usado em **Dictionaries** e **Hash Maps** de jogos.

---

## 2. Dictionaries em Godot (A Ferramenta)

Em GDScript, o `Dictionary` é a implementação de um Hash Map.

```gdscript
var inventory = {
    "potion": 5,
    "sword": 1
}
```

Quando você escreve `inventory["potion"]`, a Godot não procura a string "potion" na lista. Ela:

1. Calcula `hash("potion")`.
2. Usa esse número para pular direto para o endereço de memória onde o valor `5` está guardado.

### Performance e Boas Práticas

- **Chaves:** Prefira usar `Integers`, `Strings` curtas ou `Resources` como chaves.
- **Resources como Chaves:** Sim, você pode usar um Resource como chave! A Godot usa o `Object ID` (o endereço de memória) como hash, o que é ultra-rápido.

```gdscript
var loot_table = {
    preload("res://items/sword.tres"): 0.1, # 10% de chance
    preload("res://items/potion.tres"): 0.5  # 50% de chance
}
```

---

## 3. O Padrão "Query Hash Map"

Agora que entendemos a ferramenta, vamos falar de **Arquitetura**.

O problema comum em jogos é buscar algo baseado em múltiplos critérios.
Exemplo: _"Quero um item que seja do tipo ESPADA e de nível 5."_

A abordagem ingênua (O(N)):

```gdscript
# Lento! Itera sobre todos os itens do jogo.
for item in all_items:
    if item.type == SWORD and item.level == 5:
        return item
```

A abordagem **Query Hash Map**:
Nós pré-calculamos as chaves combinando os critérios.

```gdscript
# No _ready() ou no Editor (@tool)
var item_lookup = {
    "SWORD_5": sword_lvl_5_resource,
    "AXE_10": axe_lvl_10_resource
}

# No Runtime (O(1))
func get_item(type: String, level: int) -> Item:
    var key = type + "_" + str(level)
    return item_lookup.get(key)
```

Transformamos uma busca complexa em uma concatenação de string e um acesso direto à memória.

---

## 4. Query Hash Map Invertido (A Arquitetura do Plugin)

Aqui é onde separamos os amadores dos profissionais. Esta é a arquitetura usada no **Behavior Engineering** (Plugin) para gerenciar milhares de ações sem perder performance.

### 4.1. O Problema da Escolha

Em uma State Machine complexa, você não quer um item específico. Você quer **"Qualquer coisa que sirva"**.
_"Estou caindo (AIR) e segurando uma lança (SPEAR). O que posso fazer?"_

Não existe uma chave única "AIR_SPEAR" que retorne uma única ação. Podem existir 10 ataques aéreos diferentes.

### 4.2. A Inversão (O "Balde")

Em vez de mapear `Chave -> Valor Único`, nós mapeamos `Contexto -> Lista de Candidatos`.
Chamamos isso de **Indexação Invertida** (similar a como o Google busca sites por palavras-chave).

**Estrutura de Dados (O Mapa):**

```gdscript
# Mapeia uma TAG de Contexto para uma LISTA de Ações possíveis
var rules = {
    BehaviorTags.Physics.AIR: [AirSlash, DiveKick, Glide],
    BehaviorTags.Physics.GROUND: [Walk, Run, Idle],
    BehaviorTags.Weapon.SPEAR: [SpearThrust, SpearThrow]
}
```

### 4.3. O Algoritmo de Query (Runtime)

Quando o jogador aperta "Ataque", nós não checamos todas as ações.
Nós olhamos para o contexto atual do personagem.

1. **Contexto:** Estou no `AIR`.
2. **Query O(1):** `rules[AIR]` retorna `[AirSlash, DiveKick, Glide]`.
3. **Redução:** De 500 ações possíveis, reduzimos para 3 candidatos instantaneamente.

### 4.4. O Desempate (Scoring System)

Agora que temos 3 candidatos no "balde", quem vence?
Rodamos um algoritmo de **Score** simples nesses 3 itens:

- `Glide`: Requer `AIR`. (Match: 1 ponto)
- `AirSlash`: Requer `AIR` + `SWORD`. (Tenho Spear. Match: Inválido ❌)
- `DiveKick`: Requer `AIR` + `SPEAR`. (Tenho Spear. Match: 2 pontos ✅)

**Vencedor:** `DiveKick`.

### Conclusão

O **Query Hash Map Invertido** permite que seu jogo tenha milhares de habilidades, itens e reações, mantendo a lógica de decisão extremamente rápida e desacoplada. Você não escreve `if/else`. Você apenas joga dados nos baldes certos e deixa o Hash Map fazer a mágica.
