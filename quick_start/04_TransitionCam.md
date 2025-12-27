# Aula 4: TransitionCam - Câmeras Virtuais e Âncoras

## 🎯 Objetivo

Criar um sistema de câmeras virtuais inspirado no Cinemachine, onde Resources definem o comportamento.

---

## 🟢 [INTRODUÇÃO]

- "Sua câmera principal é burra. Ela só renderiza."
- "A inteligência vai morar nas 'Câmeras Virtuais' (VirtualCams)."

## 🟢 [O CONCEITO]

- "VirtualCam não é uma Camera3D/2D real. É um ponto no espaço."
- "Ela contém um Resource que diz: 'Siga o player com suavidade X' ou 'Fique parada'."

## 🟢 [PASSO A PASSO]

1. **CamConfig (Resource):**
   - `@export var damping: float`.
   - `@export var offset: Vector2`.
2. **VirtualCam (Node):**
   - Possui o `CamConfig`.
   - Ao ser ativada, avisa o Server.
3. **CameraServer:**
   - O Árbitro. Ele vê qual VirtualCam tem prioridade e move a Camera Real para lá usando `lerp` ou `Tweens`.

## 🟢 [DICA DO ARQUITETO]

- "Transições de câmera não são apenas visuais, são de contexto."
- "Ao trocar de VirtualCam, você pode mudar o humor do jogo sem trocar uma única linha na câmera principal."

---
