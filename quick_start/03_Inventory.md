# Aula 3: Inventory - Sistema de Itens Data-Driven

## 🎯 Objetivo

Implementar um sistema de inventário profissional com separação entre Definição e Instância.

---

## 🟢 [INTRODUÇÃO]

- "Esqueça listas de Strings para itens."
- "Vamos usar Resources para definir o DNA dos itens e um Server para as transações."

## 🟢 [PASSO A PASSO]

1. **ItemData (Definição):**
   - `@export var icon: Texture`.
   - `@export var max_stack: int`.
2. **ItemInstance (O Item Vivo):**
   - Refere-se ao `ItemData`.
   - Guarda `durability` e `quantity`.
3. **InventoryServer:**
   - Gerencia um `Array[ItemInstance]`.
   - Implementa `add_item()` e `swap_slots()`.
4. **A HUD (UI):**
   - Implemente o `_get_drag_data` e `_can_drop_data`.
   - Mostre o cooldown e durabilidade lendo direto da Instância.

## 🟢 [DICA DO ARQUITETO]

- "O Inventário não sabe 'o que' o item faz. Ele apenas gerencia 'onde' o item está."
- "A separação de responsabilidades aqui é o que permite escalabilidade."

---
