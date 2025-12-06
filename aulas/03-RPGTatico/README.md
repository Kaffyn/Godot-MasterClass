# Módulo 03: Sistemas de Dados e UI (RPG Tático)

> **Foco:** Resource-Oriented Programming (ROP) Profundo e Interfaces Complexas.

Aqui o foco sai da ação frenética e vai para a arquitetura de dados. Como gerenciar inventários, skills e saves?

## Ementa

1. **Custom Resources Avançados:**

   - `ItemData`, `SkillData`, `CharacterSheet`.
   - Resources referenciando outros Resources.

2. **Sistema de Inventário:**

   - Separação total entre Dados (`InventoryComponent`) e Visual (`InventoryUI`).
   - Drag and Drop.

3. **Arquitetura de UI:**

   - Padrão MVC (Model-View-Controller) adaptado para Godot.
   - Temas e Estilização global.

4. **Persistência (Save/Load):**
   - Serializando Resources para disco.
   - Salvando o estado do mundo.
