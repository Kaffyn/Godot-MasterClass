# Machi Class: GDExtension & Performance Nativa

> **Instrutor:** Machi
> **Objetivo:** Entender a arquitetura nativa da Godot. Saber como integrar código C++ ou Rust para plugins e sistemas de alta performance.

---

## 1. O que é GDExtension?

GDExtension **não é uma linguagem**. É a **API de interoperabilidade** da Godot.

Ela funciona como uma ponte que permite conectar bibliotecas compiladas (escritas em C++, Rust, Go, Swift) diretamente ao motor do jogo. O resultado final é um arquivo binário (`.dll` no Windows, `.so` no Linux) que a Godot carrega ao iniciar.

**A Mágica:** As classes criadas via GDExtension aparecem no editor como nós nativos. Para quem usa, não há diferença visual entre um `Sprite2D` (nativo da engine) e um `MyCustomSprite` (feito por você em C++).

---

## 2. Para que serve?

O uso vai muito além de apenas plugins. Existem 3 motivos técnicos para usar GDExtension:

1. **Performance Bruta (Number Crunching):**
   GDScript é interpretado. C++ é compilado. Para loops massivos (ex: calcular a rota de 5.000 zumbis ou gerar um terreno de voxels frame a frame), o GDScript pode gargalar. O GDExtension roda "no metal".

2. **Integração de Bibliotecas (SDKs):**
   Se você precisa usar a **Steamworks SDK**, integração com **Discord**, ou um driver proprietário que só existe em C/C++, você precisa do GDExtension para fazer o "binding" (a cola) entre essa biblioteca e a Godot.

3. **Segurança (Obfuscação):**
   É trivial ler o código fonte de um jogo feito apenas em GDScript (mesmo exportado). Código compilado em C++ é binário, tornando a engenharia reversa muito mais difícil.

---

## 3. As Linguagens Suportadas

A GDExtension é agnóstica, mas duas linguagens dominam o ecossistema:

### C++ (Godot-CPP)

A linguagem "oficial".

- **Prós:** É a linguagem em que a Godot foi escrita. Documentação oficial robusta.
- **Contras:** Sintaxe complexa, gerenciamento de memória manual (risco de vazamento de memória e crashes).

### Rust (Godot-Rust / gdext)

A favorita da comunidade moderna.

- **Prós:** **Memory Safety** (o compilador impede que você cometa erros que crasham o jogo), gerenciador de pacotes (`cargo`) superior.
- **Contras:** Tempos de compilação maiores.

---

## 4. Workflow: A Arquitetura Híbrida

Em um projeto profissional, raramente se usa 100% C++ ou 100% GDScript. Usamos uma abordagem híbrida:

| Camada       | Linguagem      | Responsabilidade                                            |
| :----------- | :------------- | :---------------------------------------------------------- |
| **Core**     | **C++ / Rust** | Algoritmos pesados (Pathfinding, Geração de Malha, Física). |
| **Gameplay** | **GDScript**   | Movimentação, Lógica de Jogo, UI, Interação.                |

**Exemplo Prático:**
Você cria um nó `VoxelGenerator` em C++ que calcula milhões de vértices por segundo.
No GDScript, você apenas instancia esse nó e define: `$VoxelGenerator.seed = 1234`.

---

## 5. Exemplo de Implementação (C++)

Para expor uma classe C++ ao Godot, registramos seus métodos na `ClassDB`.

**Header (`.h`):**

```cpp
class MeuNoRapido : public Sprite2D {
    GDCLASS(MeuNoRapido, Sprite2D)

protected:
    static void _bind_methods(); // O registro acontece aqui

public:
    void processamento_pesado();
};
```

**Source (`.cpp`):**

```cpp
void MeuNoRapido::_bind_methods() {
    // Diz para a Godot: "Existe uma função chamada 'processar' neste nó"
    ClassDB::bind_method(D_METHOD("processar"), &MeuNoRapido::processamento_pesado);
}

void MeuNoRapido::processamento_pesado() {
    // Código C++ de alta performance
}
```
