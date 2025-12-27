# Guia de Design de Jogos

Este documento aborda os princípios fundamentais do design de jogos, incluindo teoria das cores, game design e level design.

## 1. Teoria das Cores

A teoria das cores é um pilar do design visual em jogos, influenciando a estetica, a atmosfera e a jogabilidade.

### Círculo Cromático e Harmonia

- **Cores Primárias:** Vermelho, Azul, Amarelo.
- **Cores Secundárias:** Verde, Laranja, Roxo (mistura de primárias).
- **Cores Terciárias:** Mistura de primárias e secundárias.

**Esquemas de Harmonia:**

- **Monocromático:** Usa variações de uma única cor. Cria uma atmosfera coesa e unificada.
- **Análogo:** Cores vizinhas no círculo cromático. Baixo contraste, agradável aos olhos.
- **Complementar:** Cores opostas no círculo cromático (ex: Vermelho e Verde). Alto contraste, ideal para destacar elementos importantes.
- **Triádico:** Três cores equidistantes. Oferece um bom equilíbrio entre contraste e harmonia.
- **Tetrádico:** Quatro cores em dois pares complementares. Rico em variedade, mas difícil de equilibrar.

### Psicologia e Aplicação

- **Vermelho:** Ação, perigo, paixão, urgência. Usado para inimigos, explosões, ou botões de "ação".
- **Azul:** Calma, serenidade, frio, tecnologia. Usado para cenários noturnos, gelo, ou interfaces de ficção científica.
- **Verde:** Natureza, vida, saúde, veneno. Usado para itens de cura, florestas, ou inimigos tóxicos.
- **Amarelo:** Energia, alegria, atenção, riqueza. Usado para coletáveis (moedas), eletricidade, ou áreas de alerta.
- **Contraste e Saturação:** Use alto contraste para guiar o olhar do jogador para elementos importantes (ex: inimigo, plataforma, item). A saturação pode ser usada para indicar a "vida" ou "energia" de um ambiente.

## 2. Game Design

Game design é a arte de criar sistemas e experiências interativas.

### O Core Loop (Loop de Gameplay)

O ciclo fundamental de ações que o jogador executa repetidamente. Um bom loop é viciante e recompensador.

- **Exemplo (Jogo de Plataforma):** Ver um inimigo -> Pular sobre ele -> Coletar a moeda que ele dropa -> Repetir.
- **Componentes:**
  1. **Ação do Jogador:** A escolha ou input do jogador.
  2. **Sistema do Jogo:** A reação do mundo do jogo à ação.
  3. **Feedback:** A resposta sensorial (visual, sonora) que informa o jogador sobre o resultado.

### Curva de Dificuldade

A forma como o desafio do jogo aumenta ao longo do tempo.

- **Ideal:** Deve ter picos e vales. Momentos de desafio intenso seguidos por momentos de calmaria para o jogador aprender e se preparar.
- **Flow State:** O estado de imersão total acontece quando o nível de habilidade do jogador corresponde ao nível de desafio do jogo.

### Mecânicas, Dinâmicas e Estética (MDA Framework)

- **Mecânicas (Mechanics):** As regras e componentes do jogo. As ações que o jogador pode tomar. (Pular, Atirar, etc).
- **Dinâmicas (Dynamics):** O comportamento do sistema que emerge da interação das mecânicas. (Estratégias de combate, exploração de mapa).
- **Estética (Aesthetics):** A resposta emocional do jogador à dinâmica. (Desafio, Descoberta, Narrativa, Fantasia).

## 3. Level Design

Level design é a aplicação prática do game design na construção dos espaços do jogo.

### Princípios de Level Design

- **Guiando o Jogador (Guiding):** Usar luz, cor, composição e "migalhas de pão" (ex: moedas) para mostrar o caminho.
- **Pacing e Ritmo:** Alternar entre momentos de alta e baixa tensão. Um corredor longo e seguro pode preceder uma arena de combate.
- **Landmarking:** Criar pontos de referência únicos (uma estátua, uma torre) para ajudar o jogador a se orientar no mapa.
- **Risco e Recompensa:** Oferecer caminhos alternativos mais difíceis que levam a recompensas melhores.
- **Espaço Positivo vs. Negativo:** O espaço acessível (positivo) e inacessível (negativo) definem os limites e os caminhos do nível.
- **Composição e Linhas de Visão:** Arranjar elementos para criar cenas visualmente interessantes e dar ao jogador informações sobre o que está por vir. Use "leading lines" (linhas guia) para direcionar o olhar.

### Tipos de Level Design

- **Linear:** Um caminho único do início ao fim (ex: _Uncharted_). Foco na narrativa e em momentos controlados.
- **Mundo Aberto (Open World):** Um grande mapa com liberdade de exploração (ex: _The Witcher 3_). Foco na descoberta e na emergência.
- **Hub-and-Spoke:** Um mundo central (hub) que conecta a vários níveis distintos (ex: _Super Mario 64_).
- **Metroidvania:** Um grande mapa interconectado que se abre gradualmente à medida que o jogador ganha novas habilidades (ex: _Hollow Knight_).
