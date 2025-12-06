# Bônus: Rust & GDExtension (Performance Extrema)

> **Foco:** Segurança de Memória, Performance Nativa e Integração de Baixo Nível.

Quando o GDScript não é suficiente (ou quando você quer a robustez do Rust), a GDExtension permite escrever código nativo que conversa diretamente com a engine, sem recompilar o Godot.

## Ementa

1. **Por que Rust?:**

   - Safety (Zero Null Pointers, Zero Data Races).
   - Performance similar a C++.
   - Ecossistema Cargo.

2. **Configurando o Ambiente:**

   - A binding `godot-rust` (gdext).
   - Estrutura de um projeto GDExtension.
   - Compilando bibliotecas dinâmicas (`.dll` / `.so`).

3. **Expondo para Godot:**

   - Criando Classes e Nodes em Rust.
   - Exportando propriedades e sinais para o Editor.
   - Hot-reloading (com limitações).

4. **Integração Híbrida:**
   - Chamando Rust do GDScript e vice-versa.
   - Otimizando algoritmos pesados (Pathfinding customizado, PCG).
