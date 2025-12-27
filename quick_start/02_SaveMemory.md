# Aula 2: SaveMemory - Persistência Real com Servers

## 🎯 Objetivo

Criar um Server (Object) para gerenciar dados e Resources para estruturar o que deve ser salvo.

---

## 🟢 [INTRODUÇÃO]

- "Nós morrem. Cenas morrem. Mas a memória precisa ser eterna."
- "Hoje vamos criar o SaveMemoryServer."

## 🟢 [O PADRÃO SERVER]

- "Um Server no nosso estilo não é um Node. Ele estende 'Object' ou 'RefCounted'."
- "Ele é um Singleton que a Yggdrasil consulta."

## 🟢 [PASSO A PASSO]

1. **O Resource de Dados:**
   - Crie `SaveData.gd` (extends Resource).
   - `@export var player_pos: Vector2`.
   - `@export var current_level: String`.
2. **O Server:**
   - Crie `SaveServer.gd` (Object).
   - Implemente `save_to_disk()` e `load_from_disk()`.
3. **Integração Yggdrasil:**
   - "Na Yggdrasil, ao carregar o estado 'GAMEPLAY', injetamos o Resource carregado pelo Server."

## 🟢 [DICA DO ARQUITETO]

- "Nunca salve a SceneTree. Salve DADOS estruturados em Resources."
- "Isso evita bugs de compatibilidade e torna o save leve e seguro."

---
