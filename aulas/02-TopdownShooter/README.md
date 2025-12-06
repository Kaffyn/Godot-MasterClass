# Módulo 02: Arquitetura de Entidades (Topdown Shooter)

> **Foco:** Escala, Herança e Gerenciamento de Memória.

Neste módulo, sairemos dos minigames para um jogo de ação estruturado, focando em como gerenciar muitas entidades simultâneas.

## Ementa

1. **Herança de Inimigos:**

   - Classe Base `Enemy`.
   - Especializações: `MeleeEnemy` (Persegue) e `RangedEnemy` (Atira).

2. **Sistemas de Spawn:**

   - Spawners configuráveis e Timers.
   - Waves (Ondas de inimigos).

3. **Object Pooling:**

   - Por que instanciar e deletar (`queue_free`) é lento?
   - Criando um Pool de Projéteis para performance máxima.

4. **Componentização:**
   - Criando um `WeaponComponent` que pode ser usado pelo Player e pelos Inimigos.
