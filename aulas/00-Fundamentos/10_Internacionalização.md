# Internacionalização (i18n) e Localização (l10n)

Se você quer que seu jogo seja jogado pelo mundo, você não pode escrever textos diretamente no código ("Hardcoding").
A Godot possui um dos sistemas de localização mais robustos do mercado, baseado no padrão industrial **Gettext**.

---

## 1. O Problema do Hardcoding

**Errado (Amador):**

```gdscript
label.text = "Game Over"
```

Se você fizer isso, no dia que quiser traduzir para Português, terá que abrir o script e mudar o código. E se quiser Espanhol? E Chinês?

**Certo (Engenheiro):**

```gdscript
label.text = "GAME_OVER_MSG"
```

Você usa uma **CHAVE** (Key). A Godot troca essa chave pelo texto correto dependendo do idioma do jogador.

---

## 2. Formatos: CSV vs PO (Gettext)

A Godot aceita dois formatos principais.

### A. CSV (Simples e Rápido)

Ideal para jogos pequenos ou protótipos. Você usa o Excel/Google Sheets.

1. Crie uma planilha.
2. Primeira linha (Cabeçalho): `keys,en,pt_BR,es`
3. Linhas seguintes:
   - `MENU_START,Start Game,Iniciar Jogo,Empezar`
   - `MSG_HELLO,Hello,Olá,Hola`
4. Salve como `translations.csv` na pasta do projeto.
5. A Godot importa automaticamente.

### B. PO / Gettext (Profissional)

O formato `.po` é o padrão da indústria de software. Ele suporta **Plurais** ("1 coin", "2 coins") e **Contextos**.
Para editar, usamos o software gratuito **Poedit**.

1. Instale o **Poedit**.
2. Crie um arquivo novo.
3. Adicione suas chaves e traduções.
4. Salve como `pt_BR.po`, `en.po`, etc.
5. A Godot entende nativamente.

> **Machi Way:** Use `.po` se seu jogo tiver muito texto ou narrativa. Use CSV se for só UI básica.

---

## 3. Configurando na Godot

Não importa o formato (CSV ou PO), o passo final é o mesmo:

1. Vá em **Project > Project Settings**.
2. Aba **Localization > Translations**.
3. Clique em **Add...** e selecione seus arquivos (`.csv` ou `.po`).

Pronto. A mágica está feita.

---

## 4. Usando no Jogo

### Na UI (Automático)

Em qualquer `Label`, `Button` ou `RichTextLabel`:
Basta escrever a **CHAVE** no campo texto.

- Text: `MENU_START`
- Ao rodar o jogo, aparecerá: "Iniciar Jogo".

### No Script (`tr`)

Use a função `tr()` (translate) para traduzir chaves via código.

```gdscript
func show_welcome(player_name: String):
    # O texto no arquivo de tradução deve ser: "Olá %s, bem-vindo!"
    var message = tr("MSG_WELCOME") % player_name
    print(message)
```

---

## 5. Remap de Assets (Dublagem e Imagens)

E se a placa "PARE" for uma textura? Ou a dublagem?
A Godot permite trocar **Arquivos inteiros** baseados no idioma.

1. Selecione o arquivo (ex: `voice_intro.wav`).
2. Vá na aba **Import** (ao lado de Scene).
3. Procure a seção **Localization**.
4. Selecione o idioma (ex: `pt_BR`) e escolha o arquivo substituto (`voice_intro_pt.wav`).
5. Clique em **Reimport**.

Agora, sempre que o jogo carregar `voice_intro.wav`, ele vai verificar o idioma. Se for PT-BR, ele carrega o outro arquivo silenciosamente.

---

## Resumo

1. Nunca escreva texto final no código. Use **Chaves**.
2. Use **CSV** para coisas simples, **PO (Poedit)** para complexas.
3. Adicione os arquivos em **Project Settings > Localization**.
4. Use `tr("CHAVE")` no GDScript.
