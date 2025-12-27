# Aula 5: Ability System - Estados e Habilidades

## 🎯 Objetivo

Unificar Habilidades, Efeitos e Estados em um sistema modular baseado em Resources.

---

## 🟢 [INTRODUÇÃO]

- "Habilidades não são apenas 'poderzinhos'. São modificadores do estado do personagem."
- "Vamos construir o cérebro das Skills hoje."

## 🟢 [A ESTRUTURA]

- "Uma Skill é um Resource que contém uma lista de Efeitos."
- "O personagem possui um 'AbilityComponent' (StateMachine)."

## 🟢 [PASSO A PASSO]

1. **Effect (Resource):**
   - Classe pura com `apply(target)`.
   - Ex: `DamageEffect`, `SpeedBuffEffect`.
2. **Skill (Resource):**
   - `@export var duration: float`.
   - `@export var effects: Array[Effect]`.
3. **AbilityComponent (Node):**
   - Gerencia os estados: `IDLE`, `CASTING`, `ACTIVE`, `COOLDOWN`.
   - Quando a Skill ativa, ele processa os efeitos.

## 🟢 [DICA DO ARQUITETO]

- "Não code a lógica do fogo no script do player."
- "Code no 'BurnEffect'. Assim, qualquer coisa no mundo (inimigo, mato, barril) pode queimar usando o mesmo dado."

---
