# Módulo 10: Plugins & Tooling (Extensibilidade Nativa)

> **Foco:** Criando suas próprias ferramentas usando GDScript e GDShaders.

O Godot é uma engine feita com Godot. Isso significa que você pode estender o editor para criar fluxos de trabalho personalizados para sua equipe, sem sair da linguagem que você já ama.

## Ementa

1. **Ferramentas de Editor (`@tool`):**

   - Rodando código no editor.
   - Ciclo de vida de scripts de ferramenta.
   - Criando Gizmos customizados na Viewport.

2. **EditorPlugins:**

   - Criando docks (painéis) personalizados.
   - Adicionando botões na toolbar.
   - Inspetores customizados (`EditorInspectorPlugin`).

3. **GDShaders em Plugins:**

   - Empacotando shaders complexos como addons reutilizáveis.
   - Visual Shader Nodes customizados.
   - Gerando texturas via Compute Shaders (Godot 4+).

4. **Distribuição:**
   - Estrutura de pastas de um Addon (`addons/`).
   - `plugin.cfg`.
   - Publicando na Asset Library.
