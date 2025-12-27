# Guia de Desenvolvimento de IA para Jogos com Zyris

Este documento é um curso intensivo sobre a criação de Inteligência Artificial para inimigos em jogos, começando dos princípios fundamentais e culminando na integração com o framework **Zyris**. O objetivo é educar uma IA assistente (como o Gemini) para que ela compreenda o _workflow_ completo: da teoria à implementação prática em C++/Python, e finalmente, à utilização de ferramentas avançadas como o **Sonhar**.

**Filosofia Central:** A IA do jogo (o "cérebro" do inimigo) deve ser performática e rodar em C++. O treinamento e a otimização dessa IA, no entanto, são feitos em um ambiente de desenvolvimento separado (Python), onde modelos de linguagem como o Gemini podem atuar como "técnicos" ou "treinadores" (_coaches_), analisando dados e sugerindo melhorias, mas **nunca** como o modelo final em produção.

---

## Parte 1: Fundamentos Teóricos da IA em Jogos

Antes de escrever qualquer código, precisamos entender os modelos conceituais que governam o comportamento de uma IA.

### 1.1. O Agente e o Mundo

- **Agente:** A entidade controlada pela IA (um inimigo, um NPC).
- **Mundo:** O estado do jogo (posições, vida do jogador, etc.).
- **Objetivo da IA:** Perceber o mundo e agir sobre ele para cumprir um objetivo (derrotar o jogador, sobreviver, etc.).

### 1.2. Máquinas de Estado Finito (Finite State Machines - FSMs)

A abordagem mais clássica e intuitiva.

- **Conceito:** Um agente só pode estar em um "estado" por vez. Transições entre estados ocorrem baseadas em eventos.
- **Exemplo de Estados:** `PATRULHANDO`, `INVESTIGANDO`, `PERSEGUINDO`, `ATACANDO`.
- **Exemplo de Transições:**
  - Do estado `PATRULHANDO`, se `ouvir um barulho` -> vá para `INVESTIGANDO`.
  - Do estado `INVESTIGANDO`, se `ver o jogador` -> vá para `PERSEGUINDO`.
  - Do estado `PERSEGUINDO`, se `jogador sair de vista` -> vá para `INVESTIGANDO`.
- **Prós:** Simples de entender e implementar para IAs simples.
- **Contras (O "Spaghetti Code"):** Para comportamentos complexos, as transições se tornam uma teia de aranha incontrolável. Adicionar um novo estado pode exigir a modificação de todos os outros. A lógica não é reutilizável.

### 1.3. Árvores de Comportamento (Behavior Trees - BTs)

A evolução natural das FSMs, focada em modularidade e reutilização. É o padrão da indústria moderna e a base do sistema de IA do Zyris.

- **Conceito:** Uma árvore hierárquica de tarefas. A IA "percorre" a árvore do topo para o fundo a cada _tick_ (frame de atualização) para decidir o que fazer.
- **Analogia:** É como uma lista de verificação de um piloto. "Tentar pousar o avião. Se não for possível, tentar arremeter. Se não for possível, declarar emergência."

**Os Três Tipos de Nós Fundamentais:**

1. **Nó de Ação (Action / Leaf):**

   - **Função:** Executa uma ação concreta no mundo do jogo. É a "folha" da árvore.
   - **Exemplos:** `MoverParaPosição`, `AtacarAlvo`, `TocarAnimação`.
   - **Retorno:** Retorna um de três status:
     - `SUCCESS`: A ação foi completada com sucesso.
     - `FAILURE`: A ação não pôde ser completada.
     - `RUNNING`: A ação ainda está em andamento (ex: `MoverParaPosição` leva tempo).

2. **Nó Composto (Composite):**

   - **Função:** Controla o fluxo de execução de seus nós filhos. São os "galhos" da árvore.
   - **Tipos Principais:**
     - **Sequence (`->`):** Executa seus filhos em ordem. Se um filho retorna `FAILURE`, a sequência inteira para e retorna `FAILURE`. Se um filho retorna `RUNNING`, a sequência para e retorna `RUNNING` (e continuará daquele filho no próximo tick). Só retorna `SUCCESS` se **todos** os filhos retornarem `SUCCESS`.
       - _Uso:_ Para uma sequência de passos que **devem** ocorrer em ordem (Ex: `Sequence` -> `MoverParaAtrásDoJogador` -> `ExecutarAtaqueFurtivo`).
     - **Selector (`?` ou Fallback):** Executa seus filhos em ordem até que um retorne `SUCCESS` ou `RUNNING`. Se um filho retorna `SUCCESS`, o seletor para e retorna `SUCCESS`. Só retorna `FAILURE` se **todos** os filhos retornarem `FAILURE`.
       - _Uso:_ Para tentar uma série de ações em ordem de prioridade (Ex: `Selector` -> `TentarAtaqueEspecial` -> `TentarAtaqueNormal` -> `Fugir`).

3. **Nó Decorador (Decorator):**
   - **Função:** Modifica o comportamento de um único nó filho.
   - **Exemplos:**
     - **Inverter:** Inverte o resultado do filho (`SUCCESS` vira `FAILURE` e vice-versa).
     - **Succeeder:** Sempre retorna `SUCCESS`, não importa o que o filho retorne.
     - **Condition:** Só executa o filho se uma condição for verdadeira (Ex: `InimigoComVidaAbaixoDe(25%)`).

- **O Blackboard:** É a "memória" da IA. Um dicionário de dados (chave-valor) onde a árvore pode ler e escrever informações, como `alvoAtual`, `ultimaPosicaoVistaDoJogador`, etc.

---

## Parte 2: Implementação Prática (C++ e Python)

Vamos esboçar a arquitetura de uma Behavior Tree em C++ (para o _runtime_ do jogo) e um script de treinamento em Python.

### 2.1. Arquitetura da Behavior Tree em C++ (Runtime)

Esta implementação será a base do que roda dentro do `BehaviorTreePlayer` do Zyris. É performática e não tem dependências externas.

```cpp
// behavior_tree.h
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <map>

// O Blackboard (memória da IA)
class Blackboard {
public:
    template<typename T>
    void set(const std::string& key, const T& value) {
        // Em uma implementação real, usaria std::any ou similar
        // Aqui, simplificamos para strings para demonstração.
        data_[key] = std::to_string(value);
        std::cout << "Blackboard: Set '" << key << "' to '" << data_[key] << "'" << std::endl;
    }

    std::string get(const std::string& key) {
        if (data_.find(key) == data_.end()) {
            return "";
        }
        return data_[key];
    }
private:
    std::map<std::string, std::string> data_;
};

// Enum para o status de retorno de um nó
enum class NodeStatus {
    SUCCESS,
    FAILURE,
    RUNNING
};

// Classe base abstrata para todos os nós da árvore
class BTNode {
public:
    virtual ~BTNode() = default;
    virtual NodeStatus tick(Blackboard& blackboard) = 0;
};

// --- Nós Compostos ---

class Sequence : public BTNode {
public:
    explicit Sequence(std::vector<std::unique_ptr<BTNode>> children) : children_(std::move(children)) {}

    NodeStatus tick(Blackboard& blackboard) override {
        for (auto& child : children_) {
            NodeStatus status = child->tick(blackboard);
            if (status != NodeStatus::SUCCESS) {
                return status; // Retorna FAILURE ou RUNNING imediatamente
            }
        }
        return NodeStatus::SUCCESS; // Todos os filhos tiveram sucesso
    }

private:
    std::vector<std::unique_ptr<BTNode>> children_;
};

class Selector : public BTNode {
public:
    explicit Selector(std::vector<std::unique_ptr<BTNode>> children) : children_(std::move(children)) {}

    NodeStatus tick(Blackboard& blackboard) override {
        for (auto& child : children_) {
            NodeStatus status = child->tick(blackboard);
            if (status != NodeStatus::FAILURE) {
                return status; // Retorna SUCCESS ou RUNNING imediatamente
            }
        }
        return NodeStatus::FAILURE; // Todos os filhos falharam
    }

private:
    std::vector<std::unique_ptr<BTNode>> children_;
};

// --- Nós de Ação (Folhas) ---

class CheckHealth : public BTNode {
public:
    explicit CheckHealth(int threshold) : threshold_(threshold) {}

    NodeStatus tick(Blackboard& blackboard) override {
        int health = std::stoi(blackboard.get("health"));
        std::cout << "Action: Checking health (" << health << ") against threshold (" << threshold_ << ")" << std::endl;
        if (health < threshold_) {
            blackboard.set("is_low_health", true);
            return NodeStatus::SUCCESS;
        }
        return NodeStatus::FAILURE;
    }
private:
    int threshold_;
};

class Flee : public BTNode {
public:
    NodeStatus tick(Blackboard& blackboard) override {
        if (blackboard.get("is_fleeing") == "1") {
            std::cout << "Action: Still fleeing..." << std::endl;
            return NodeStatus::RUNNING;
        }
        std::cout << "Action: Started to Flee!" << std::endl;
        blackboard.set("is_fleeing", true);
        return NodeStatus::RUNNING; // A fuga leva tempo
    }
};

class Attack : public BTNode {
public:
    NodeStatus tick(Blackboard& blackboard) override {
        std::cout << "Action: Attacking the player!" << std::endl;
        return NodeStatus::SUCCESS;
    }
};

// Exemplo de uso
void run_cpp_example() {
    Blackboard blackboard;
    blackboard.set("health", 15); // Vida atual do inimigo

    // Construir a árvore de comportamento
    // Lógica:
    // Se a vida estiver baixa (< 20), fuja.
    // Senão, ataque.
    auto tree = std::make_unique<Selector>(std::vector<std::unique_ptr<BTNode>>{
        // Ramo 1: Lógica de fuga
        std::make_unique<Sequence>(std::vector<std::unique_ptr<BTNode>>{
            std::make_unique<CheckHealth>(20),
            std::make_unique<Flee>()
        }),
        // Ramo 2: Lógica de ataque (fallback)
        std::make_unique<Attack>()
    });

    std::cout << "--- C++ BEHAVIOR TREE TICK 1 ---" << std::endl;
    tree->tick(blackboard); // Deverá começar a fugir

    std::cout << "\n--- C++ BEHAVIOR TREE TICK 2 ---" << std::endl;
    tree->tick(blackboard); // Deverá continuar a fugir
}

```

### 2.2. Script de Treinamento e "Coaching" com Python

Este script **NÃO** roda no jogo. Ele roda offline, no ambiente de desenvolvimento. Ele simula um loop de treinamento de Reinforcement Learning e mostra como a API do Gemini pode ser usada para analisar os logs e dar sugestões.

```python
# training_coach.py
import os
import numpy as np
# from google.generativeai import GenerativeModel # Conceitual

# --- Ambiente de Simulação Simples ---
class SimpleGameEnv:
    """Um ambiente de jogo simulado para treinar a IA."""
    def __init__(self):
        self.player_pos = 5
        self.ai_pos = 0
        self.ai_health = 100
        self.done = False
        print("SimEnv: Jogo iniciado. Jogador na posição 5.")

    def get_state(self):
        """O estado é a distância até o jogador."""
        return np.array([self.player_pos - self.ai_pos])

    def step(self, action):
        """Executa uma ação e retorna (novo_estado, recompensa, fim_de_jogo)."""
        if self.done:
            return self.get_state(), 0, self.done

        # 0: Mover para perto, 1: Mover para longe, 2: Atacar
        if action == 0:
            self.ai_pos += 1
        elif action == 1:
            self.ai_pos -= 1
        elif action == 2:
            if abs(self.player_pos - self.ai_pos) <= 1:
                print("SimEnv: IA acertou o jogador!")
                self.done = True
                return self.get_state(), 100, self.done # Grande recompensa
            else:
                print("SimEnv: IA errou o ataque.")
                return self.get_state(), -10, self.done # Penalidade por errar

        # Recompensa baseada na distância
        reward = -abs(self.player_pos - self.ai_pos) # Penalidade por estar longe
        return self.get_state(), reward, self.done

# --- Agente de RL Simples (Q-learning) ---
class RLAgent:
    def __init__(self):
        # Tabela Q: estados (distância) x ações
        self.q_table = np.zeros((10, 3)) # 10 distâncias possíveis, 3 ações
        self.learning_rate = 0.1
        self.discount_factor = 0.9
        self.epsilon = 0.1 # 10% de chance de tomar uma ação aleatória (exploração)

    def choose_action(self, state):
        if np.random.uniform(0, 1) < self.epsilon:
            return np.random.choice([0, 1, 2]) # Ação aleatória
        else:
            # Ação baseada no maior valor Q para o estado atual
            distance = state[0]
            if 0 <= distance < 10:
                return np.argmax(self.q_table[distance, :])
            return 0 # Ação padrão

    def update_q_table(self, state, action, reward, next_state):
        distance = state[0]
        next_distance = next_state[0]

        if not (0 <= distance < 10 and 0 <= next_distance < 10):
            return

        old_value = self.q_table[distance, action]
        next_max = np.max(self.q_table[next_distance, :])

        # Fórmula do Q-learning
        new_value = old_value + self.learning_rate * (reward + self.discount_factor * next_max - old_value)
        self.q_table[distance, action] = new_value

def train_ai():
    """Loop principal de treinamento."""
    agent = RLAgent()
    env = SimpleGameEnv()
    training_logs = []

    for episode in range(100): # Rodar 100 "partidas"
        state = env.get_state()
        done = False
        total_reward = 0

        log_entry = f"\n--- Episódio {episode + 1} ---\n"

        while not done:
            action = agent.choose_action(state)
            next_state, reward, done = env.step(action)
            agent.update_q_table(state, action, reward, next_state)

            log_entry += f"Estado: {state[0]}, Ação: {action}, Recompensa: {reward}\n"

            state = next_state
            total_reward += reward

        log_entry += f"Recompensa Total: {total_reward}\n"
        training_logs.append(log_entry)

        env = SimpleGameEnv() # Reinicia o jogo

    print("\n--- Tabela Q Final (Cérebro da IA) ---")
    print(agent.q_table)

    return "".join(training_logs), agent.q_table


def get_coaching_from_gemini(logs, api_key):
    """
    Função CONCEITUAL para enviar os logs de treino para a API do Gemini
    e pedir uma análise.
    """
    # Em um cenário real:
    # os.environ["GEMINI_API_KEY"] = api_key
    # model = GenerativeModel("gemini-1.5-pro")

    prompt = f"""
    Você é um especialista em IA para jogos. Analise os seguintes logs de treinamento de um agente de Reinforcement Learning.
    O objetivo do agente é atacar um jogador. O estado é a distância, as ações são (0: perto, 1: longe, 2: atacar).

    Logs:
    {logs}
    Com base nesses logs, forneça um relatório com:
    1. Uma avaliação do comportamento geral da IA.
    2. Sugestões para melhorar a função de recompensa (reward function) para incentivar um comportamento mais inteligente.
    3. Ideias para novos "estados" ou "sensores" que a IA poderia usar para tomar decisões melhores.
    """

    print("\n--- ANÁLISE DO COACH (GEMINI) ---")
    print("Enviando logs para a API do Gemini...")
    # response = model.generate_content(prompt)
    # print(response.text)

    # Resposta simulada para este exemplo:
    simulated_response = """
    **Relatório de Análise do Coach de IA**

    1. **Avaliação do Comportamento:** O agente aprendeu a associação básica entre estar perto (distância <= 1) e atacar (ação 2) para obter a recompensa máxima. No entanto, ele parece oscilar muito quando está longe, pois a penalidade é a mesma para qualquer distância.

    2. **Melhora na Recompensa:** A função de recompensa atual `-abs(distância)` é linear. Considere uma penalidade exponencial (`-distância^2`). Isso o puniria muito mais por estar longe, incentivando-o a se aproximar mais agressivamente. Adicione também uma pequena recompensa por se mover na direção certa (ação 0 quando a distância > 1).

    3. **Novos Estados/Sensores:** O estado atual é muito simples. Para um `Behavior Tree`, o `Blackboard` poderia ser enriquecido com:
        *   `player_is_attacking` (bool): A IA deveria aprender a se esquivar.
        *   `low_health` (bool): Deveria aprender a recuar (ação 1) se a vida estiver baixa.
        *   `obstacle_between_us` (bool): Deveria procurar um caminho alternativo em vez de apenas se aproximar.
    """
    print(simulated_response)


# --- Execução ---
training_logs, final_q_table = train_ai()
# get_coaching_from_gemini(training_logs, "SUA_GEMINI_API_KEY_AQUI")

```

---

## Parte 3: Conectando Tudo ao Framework Zyris

Agora que entendemos a teoria e a prática isoladamente, vamos ver como elas se encaixam na arquitetura profissional do Zyris.

### 3.1. Behavior Tree e Ability System: A Dupla Dinâmica

O `zyris.md` afirma:

> | Sistema            | Papel          | Responsabilidade                                              |
> | :----------------- | :------------- | :------------------------------------------------------------ |
> | **Behavior Tree**  | **Estratégia** | Decide _o que_ fazer (ex: "Inimigo perto -> Atacar").         |
> | **Ability System** | **Execução**   | Decide _como_ fazer (ex: "Qual ataque usar? Tenho stamina?"). |

**Tradução Prática:**

- Nossa **Behavior Tree** (implementada em C++ como o `BehaviorTreePlayer` do Zyris) toma a decisão de alto nível. Um nó folha (Action) na árvore não seria `Atacar()`, mas sim `TentarExecutarHabilidadeComTag("Ataque.CorpoACorpo")`.
- Este nó de ação **NÃO** contém a lógica do ataque. Ele apenas faz uma chamada para o **Ability System** daquele personagem.
- O **Ability System**, que é um sistema orientado a dados (`Resources`), recebe essa "intenção". Ele então:
  1. Constrói o `Contexto` (Onde estou? Quem é o alvo? Estou com algum efeito negativo?).
  2. Consulta todas as habilidades (`State` Resources) que ele possui com a tag `Ataque.CorpoACorpo`.
  3. Valida os requisitos de cada uma (Tenho mana? A arma está equipada?).
  4. Pontua (`Scoring`) as habilidades válidas (Um ataque forte pode ter um score base maior, mas um ataque rápido pode ganhar pontos se o inimigo estiver quase morto).
  5. Seleciona a melhor e a executa.

A implementação em C++ que fizemos é o _motor_ da árvore, mas as _ações_ que ela dispara são, na verdade, gatilhos para o `Ability System`.

### 3.2. O Papel do "Sonhar": O Editor Visual

- **O que é?** O `Sonhar` é a ferramenta de edição visual (um "Main Panel" no Godot) para os sistemas do Zyris.
- **Domínio do Behavior Tree:** Em vez de criar a árvore no código C++ como fizemos no exemplo, você a montaria visualmente no Sonhar, arrastando e soltando nós de `Sequence`, `Selector` e `Tasks` customizadas. Isso permite que designers de IA ajustem a lógica sem precisar recompilar o jogo. O `BehaviorTree` Resource é o arquivo (`.tres`) que o Sonhar salva.
- **Domínio do Ability System:** Da mesma forma, o `Sonhar` fornece editores para criar os `State`, `Skill` e `Effect` Resources. Você pode criar um novo ataque, definir seu custo, dano e efeitos visuais, tudo através de uma interface gráfica, que salva o resultado em um `Resource`.

### 3.3. O Cérebro Híbrido: Onde o Treinamento Entra

O `zyris.md` descreve um "Cérebro Híbrido".

1. **Reflexos (Reinforcement Learning):**

   - O nosso script `training_coach.py` é uma versão simplificada do "Training (Python) backend" do Zyris.
   - Em um projeto real, o "ambiente" seria o próprio jogo Godot, exportando dados de estado. O agente Python aprenderia e, no final, a `q_table` (ou, mais realisticamente, os pesos de uma rede neural) seria salva em um arquivo otimizado: o **`MachiBrain`** resource.
   - No C++ (runtime), um nó especial da Behavior Tree (`RLTask`) carregaria o `MachiBrain` e o usaria para tomar decisões de baixo nível, como o timing exato de uma esquiva ou o melhor ângulo para um ataque. Isso é reservado para IAs complexas, como chefes de fase.

2. **Cognição (Gemini Coach):**
   - Este é o papel exato que demonstramos na função `get_coaching_from_gemini`.
   - Durante o desenvolvimento, os programadores de IA rodam os scripts de treinamento. Os logs gerados são enviados para a API do Gemini.
   - O Gemini atua como um "consultor", analisando a performance e sugerindo melhorias na lógica da IA, nas recompensas do RL, ou até mesmo gerando _assets_ estáticos (como variações de diálogos para NPCs com base em seu perfil de comportamento).
   - **Vantagem de Custo:** Este processo ocorre **offline**, em tempo de desenvolvimento. Não há custo de API durante o gameplay. O Gemini ajuda a criar uma IA melhor, mas não é a IA.

**Conclusão do Fluxo de Trabalho:**

1. **Design da IA (Sonhar):** Um designer cria a lógica estratégica geral em uma **Behavior Tree** visual.
2. **Design das Ações (Sonhar):** O designer ou programador cria as habilidades, ataques e efeitos no editor do **Ability System**.
3. **Implementação (C++):** O `BehaviorTreePlayer` executa a árvore, e os nós de ação invocam o `Ability System`.
4. **Treinamento (Python - Opcional):** Para IAs complexas, um agente de RL é treinado offline contra uma simulação do jogo.
5. **Coaching (Gemini API - Opcional):** Logs do treinamento são enviados ao Gemini para análise e sugestões de otimização.
6. **Deployment:** O modelo treinado (`MachiBrain`) é carregado pela BT em C++.
7. **Resultado Final:** Uma IA inimiga robusta, performática e com comportamento complexo, projetada visualmente, executada em C++ nativo e otimizada com a ajuda de Python e Machine Learning.

```

```
