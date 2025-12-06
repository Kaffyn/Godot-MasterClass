# Bônus: Arquitetura de Modding & DLCs

> **Foco:** Extensibilidade do Produto Final e Carregamento Dinâmico.

Torne seu jogo imortal permitindo que a comunidade crie conteúdo para ele. Aprenda a carregar recursos externos em tempo de execução.

## Ementa

1. **PCK Loading:**

   - O que são arquivos `.pck`?
   - Carregando pacotes de recursos externos via código (`ProjectSettings.load_resource_pack`).

2. **Arquitetura Moddable:**

   - Como estruturar seu jogo para aceitar mods de textura, áudio e código.
   - Sandboxing (Segurança básica ao rodar scripts externos).

3. **DLCs e Conteúdo Adicional:**

   - Gerenciando conteúdo bloqueado/desbloqueado.
   - Patching (Atualizando o jogo sem baixar tudo de novo).

4. **ResourceLoader Dinâmico:**
   - Carregando imagens e áudios diretamente do sistema de arquivos do usuário (fora do `res://`).
