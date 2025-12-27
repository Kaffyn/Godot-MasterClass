# Treinamento de IA para Assistente de Desenvolvimento de Jogos

Este repositório contém um conjunto de documentos projetados para treinar um assistente de IA (como o Gemini CLI) nos fundamentos e práticas do desenvolvimento de jogos.

O objetivo é criar uma base de conhecimento que permita à IA responder a perguntas e auxiliar em tarefas relacionadas não apenas à programação, mas também à teoria de design, arquitetura de software para jogos e uso de ferramentas padrão da indústria.

## Estrutura dos Documentos

O conhecimento é organizado de forma modular para facilitar o aprendizado e a consulta.

- **`GEMINI.md`**: O documento principal e ponto de entrada. Ele funciona como um índice, fornecendo uma visão geral de cada tópico e links para os guias especializados.

- **`game_design.md`**: Foca nos conceitos teóricos e artísticos do desenvolvimento de jogos. Essencial para que a IA possa discutir e aconselhar sobre a experiência do jogador.

  - Teoria das Cores
  - Game Design (Loops, Curvas de Dificuldade)
  - Level Design

- **`programming_paradigms.md`**: Aborda as diferentes maneiras de estruturar o código de um jogo, uma decisão crítica que afeta a performance e a escalabilidade.

  - Programação Orientada a Objetos (OOP)
  - Design Orientado a Dados (DOD)

- **`godot_development.md`**: Um guia prático para a Godot Engine, uma engine de código aberto popular. O objetivo é que a IA possa gerar código, explicar conceitos e ajudar com problemas específicos da Godot.

  - GDScript
  - Sistema de Cenas e Nós
  - Uso de Resources
  - GDExtension (C++)

- **`python_ai_for_gamedev.md`**: Descreve o uso avançado de Python para a criação de Inteligência Artificial e Machine Learning em jogos.
  - Algoritmos de IA (A\*, FSMs, Behavior Trees)
  - Técnicas de Machine Learning (Reinforcement Learning, Player Modeling)
  - Fluxo de trabalho para treinamento e integração de modelos de ML.

## Como Usar

Esses documentos devem ser usados como material de base para o treinamento ou `fine-tuning` de um modelo de linguagem. Ao processar esses arquivos, a IA deve ser capaz de:

1. **Responder a perguntas conceituais:** "Qual a diferença entre uma Máquina de Estado e uma Árvore de Comportamento?"
2. **Gerar código relevante:** "Crie um script em GDScript para um personagem que se move e pula."
3. **Auxiliar em tarefas de desenvolvimento:** "Descreva o processo para treinar um modelo de ML em Python e usá-lo para inferência na minha engine de jogo."
4. **Estruturar projetos:** "Como devo organizar os dados de itens do meu inventário em Godot?" (A resposta ideal seria sugerir o uso de `Resources`).
