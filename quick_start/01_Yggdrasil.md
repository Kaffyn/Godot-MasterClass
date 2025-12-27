# Aula 1: Yggdrasil - A Raiz de Tudo (Main Scene)

## 🎯 Objetivo

Criar a cena mestra que orquestra o ciclo de vida do jogo usando uma Máquina de Estados simples.

---

## 🟢 [INTRODUÇÃO]

- "Bem-vindos. Hoje vamos construir a Yggdrasil."
- "Não é apenas uma cena; é o ponto de entrada único do seu jogo."
- "O erro comum é trocar de cena solta. O jeito profissional é ter um Orquestrador."

## 🟢 [O CONCEITO]

- "A Yggdrasil não possui dados ainda. Ela possui PODER de decisão."
- "Ela vai decidir se estamos no Menu, no Loading ou no Gameplay."

## 🟢 [PASSO A PASSO]

1. **Criação do Nó:** "Crie um Node simples chamado 'Yggdrasil'. Salve como 'Main.tscn'."
2. **Máquina de Estados (Enum):**
   ```gdscript
   enum GameState { MENU, LOADING, GAMEPLAY }
   var current_state: GameState = GameState.MENU
   ```
3. **Gerenciador de Cenas:**
   - Explique o `@export var menu_scene: PackedScene`.
   - Explique o `@export var world_container: Node`.
4. **Lógica de Troca:**
   ```gdscript
   func change_state(new_state: GameState):
       # Limpeza da cena anterior
       # Instanciamento da nova cena dentro do container
   ```

## 🟢 [DICA DO ARQUITETO]

- "Por que não trocar de cena direto no root?"
- "Porque se você faz isso, você perde o controle global. A Yggdrasil permite manter a música e o fundo persistentes enquanto o resto do mundo muda."

---
