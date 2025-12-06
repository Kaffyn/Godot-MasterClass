# Machi Class: Internacionalização (i18n) e Localização

> **Instrutor:** Machi
> **Objetivo:** Preparar o jogo para múltiplos idiomas desde o Dia 1. Proibido escrever texto "Hardcoded" na UI.

---

## 1. O Problema do "Hardcoding"

Se você escreve no seu botão: `Button.text = "Jogar"`.
No dia que quiser lançar em inglês, você terá que caçar esse script e mudar para:
`Button.text = "Jogar" if lang == "pt" else "Play"`.
Isso é insustentável.

---

## 2. A Solução: Chaves de Tradução (Keys)

Em vez de escrever o texto final, escrevemos um **Identificador Único**.

- `MENU_PLAY`
- `MENU_OPTIONS`
- `GAME_YOU_DIED`
- `ITEM_SWORD_NAME`
- `ITEM_SWORD_DESC`

A Godot (e o padrão da indústria) usa um sistema de **Chave -> Valor**.

---

## 3. Configurando na Godot (CSV ou PO)

A Godot suporta nativamente arquivos CSV e Gettext (PO). Para projetos pequenos/médios, CSV é o mais rápido.

### O Formato CSV Mágico

1. Crie um arquivo `translations.csv` na pasta `i18n/`.
2. A primeira linha (cabeçalho) deve ser: `keys,en,pt_BR,es`.
3. Preencha:

```csv
keys,en,pt_BR,es
MENU_PLAY,Play,Jogar,Jugar
MENU_OPTIONS,Options,Opções,Opciones
GAME_OVER,Game Over,Fim de Jogo,Fin del Juego
```

4. Salve. A Godot vai importar automaticamente e gerar arquivos `.translation` para cada coluna.
5. Vá em **Project Settings -> Localization -> Translations** e adicione os arquivos gerados.

### O Uso Automático

Agora, em qualquer `Label` ou `Button`, se você escrever `MENU_PLAY` no campo Text, a Godot automaticamente substitui por "Jogar" (ou a língua do sistema) quando o jogo rodar.

---

## 4. O Uso via Código (`tr()`)

Se precisar definir texto via script:

```gdscript
# Errado
label.text = "Bem-vindo, " + player_name

# Certo
label.text = tr("MSG_WELCOME") % player_name
```

(Assumindo que no CSV tem: `MSG_WELCOME,Welcome %s,Bem-vindo %s`).

O método `tr()` busca a chave na tabela atual.

---

## 5. Formatação Complexa (Plurais e Gênero)

Para casos avançados ("1 coin", "2 coins"), o CSV é limitado.
Recomendamos usar ferramentas como **Poedit** (formato `.po`).
A Godot tem suporte nativo a `.po`.

O fluxo é:

1. Extrair strings do jogo.
2. Mandar o `.po` para o tradutor.
3. Ele devolve o `.po` preenchido.
4. Godot lê.

---

## 6. Remap de Assets (Dublagem e Imagens)

E se a placa "PARE" no jogo for uma textura?
E se o áudio da narração precisar mudar de português para inglês?

A Godot tem o sistema de **Remaps**.

1. Selecione o arquivo `voice_intro.wav`.
2. Na aba Import, vá em **Localization**.
3. Selecione o idioma (ex: `en`) e aponte para o arquivo substituto `voice_intro_en.wav`.

Quando o jogo estiver em inglês, qualquer `load("res://voice_intro.wav")` vai carregar silenciosamente a versão em inglês. Mágica.
