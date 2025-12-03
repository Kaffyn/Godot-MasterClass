# Visão de Raio-X: Debugging e Profiling

Um engenheiro não adivinha; ele mede.
A Godot oferece ferramentas profissionais para dissecar seu jogo enquanto ele roda.

---

## 1. O Debugger (Painel Inferior)

Quando o jogo crasha ou você usa `print()`, tudo aparece aqui.

- **Stack Trace:** Mostra a linha exata do erro e quem chamou quem. Clique na pilha para navegar no código.
- **Breakpoints (F9):** Pausa o jogo naquela linha. Permite inspecionar o valor das variáveis naquele exato momento. Essencial para entender "por que essa variável está null?".

---

## 2. Remote Scene Tree

Enquanto o jogo roda, olhe para a aba **Scene** no Editor.
Acima dela, clique em **Remote**.

Agora você está vendo a SceneTree **ao vivo**.

- Você pode ver inimigos nascendo e morrendo.
- Você pode clicar em um nó e editar suas variáveis **em tempo real** (mudar a velocidade do player, a cor da luz).
- Ótimo para testar balanceamento sem reiniciar o jogo.

---

## 3. Debug Visual (Menu Debug)

No menu superior **Debug**, ative opções visuais:

- **Visible Collision Shapes:** Mostra as hitboxes. Fundamental para ver se seu ataque está acertando.
- **Visible Navigation:** Mostra a malha de navegação da IA.

---

## 4. Profiler e Monitors

Seu jogo está lento? Não adivinhe. Vá na aba **Debugger > Monitors**.

- **FPS:** Quadros por segundo.
- **Process Memory:** Quanta RAM o jogo está usando.
- **Objects:** Quantos nós existem. Se esse número só sobe e nunca desce, você tem um **Memory Leak** (esqueceu de dar `queue_free`).

**Profiler:** Mostra exatamente qual função está demorando mais para rodar.
Se o gráfico diz que `_physics_process` do `Enemy.gd` está levando 20ms, você achou o culpado do lag.

> **Machi Way:** Otimização prematura é a raiz de todo mal. Faça funcionar primeiro, faça ficar bonito depois, e só otimize se o Profiler disser que precisa.
