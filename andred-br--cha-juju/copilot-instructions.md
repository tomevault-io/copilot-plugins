## cha-juju

> Contexto para quem (ou qual assistente) for mexer neste repositório depois.

# CLAUDE.md

Contexto para quem (ou qual assistente) for mexer neste repositório depois.

## O que é

Site do chá de bebê da Julieta. Funciona como site de casamento: o convidado
escolhe um presente da lista e recebe um **Pix copia e cola** já com o valor
preenchido, para colar no app do banco. Depois de copiar, pode deixar um
recado — que vai para uma planilha privada, nunca para o site.

Feito para a Ana (mãe). A conta que recebe é do André (pai), no C6.
Vai haver mais de um chá, em cidades diferentes: **o site não tem data nem
local**, de propósito.

## Decisões de arquitetura

- **Site 100% estático.** HTML, CSS e JS puros, sem framework, sem build, sem
  dependência instalada. Hospedagem gratuita para sempre, nada para manter no ar.
- **O Pix é gerado no navegador do convidado.** `assets/pix.js` monta o BR Code
  (EMV®QRCPS do Banco Central) e calcula o CRC16-CCITT-FALSE.
- **O formato foi calibrado contra um Pix real do C6.** A primeira versão foi
  recusada pelo banco; o gabarito revelou duas diferenças: faltava o campo
  `01 = 11` (QR reutilizável) e a cidade cadastrada é SÃO PAULO, não Campinas.
  O teste `tests/testar_pix.js` compara campo a campo com esse código real —
  **se mexer no gerador, rode esse teste antes de qualquer coisa.**
- **O identificador da transação é `***`.** Um txid inventado é o único campo
  que pode fazer um banco recusar o código, e não é necessário: como cada
  presente tem um **valor único**, o valor já identifica o presente no extrato.
- **Cada presente aceita um `codigo` fixo** no catálogo. Se preenchido, o site
  usa esse código em vez de montar o dele — escotilha de emergência caso algum
  banco implique com o código gerado.
- **Recados vão para o Google Sheets** via Apps Script (`apps-script/Codigo.gs`).
  O envio usa `fetch` com `mode: 'no-cors'` e `Content-Type: text/plain`, que
  evita o preflight CORS que o Apps Script não responde. Como a resposta é
  opaca, **o site não sabe se deu certo** — por isso o Apps Script manda um
  e-mail de confirmação opcional.
- **Dez presentes.** A lista começou com vinte e foi enxugada: lista longa
  cansa quem escolhe. As pinturas dos dez que saíram continuam em `arte/`.
- **Presentes podem se repetir.** Sem controle de estoque, decisão da Ana. É o
  que mantém o site sem estado e sem backend.
- **Nada de localStorage nem cookie.**

## Estrutura

```
index.html               markup da página inteira, inclusive o modal
assets/styles.css        paleta e layout
assets/pix.js            gerador do BR Code (também roda no Node, nos testes)
assets/app.js            CONFIG, CATALOGO, montagem da página, modal, recados
assets/ilustracao.jpg    a aquarela do save the date
assets/bichos/*.png      bichos recortados da aquarela (fundo transparente)
assets/presentes/*.png   os 10 ícones de presente, pintados por código
arte/icones.py           20 pinturas (as 10 em uso + 10 de reserva)
arte/base.py             primitivas de apoio (corpo, sombra, linha, arred)
apps-script/Codigo.gs    web app que grava os recados na planilha
tests/                   três suítes, descritas abaixo
```

**Ponto de entrada para mudança de conteúdo: `CONFIG` e `CATALOGO`, no topo do
`assets/app.js`.**

## Identidade visual

Vem inteira da aquarela do save the date. Nada foi baixado da internet.

**Os bichos** (`assets/bichos/`) são recortes da própria ilustração. Os que
estão sobre papel branco viram recorte limpo (beija-flor e joaninha); os que estão sobre folhagem viram medalhão de borda esfumada
(tucano, maritaca, sagui). Uso: maritaca e sagui encabeçam
as duas seções; joaninha é o ornamento das divisas; tucano aparece no modal
depois de copiar; beija-flor fecha a página.

**Os ícones dos presentes** foram pintados por código com a skill `paint`
(`/mnt/skills/examples/paint`), em aquarela, na mesma paleta. Não são fotos:
não havia como baixar imagens, e banco de imagem traria problema de licença.
Se um dia quiser fotos de verdade, é só trocar os arquivos em
`assets/presentes/` mantendo os nomes.

Cores amostradas da ilustração:

| Token | Hex | De onde veio |
|---|---|---|
| `--papel` | `#FFFFFF` | fundo da aquarela |
| `--creme` | `#FBF4E6` | papel da ilustração |
| `--halo` | `#F7E6C9` | a aguada pêssego atrás do moisés |
| `--granada` | `#852E27` | as flores vinho |
| `--musgo` | `#425833` | as samambaias |
| `--casca` | `#7A5A42` | os troncos |

Tipos: **Gilda Display** (títulos, valores) e **Alegreya Sans** (texto), com
reservas do sistema. Cantos assimétricos (`--curva: 2px 14px 2px 14px`) para
lembrar borda de papel molhado.

Elemento assinatura: a **florada** — ao copiar o código, uma aguada pêssego se
espalha pela caixa, como tinta em papel úmido (`@keyframes florada`). É o único
efeito exuberante; o resto é deliberadamente quieto. Respeita
`prefers-reduced-motion`.

## Repintar um ícone

```bash
cd arte && CANVAS_WORKERS=0 python3 icones.py    # renderiza em arte/saida/
```

Depois, exportar para o site (recorte na caixa do desenho, quadrado, 300px,
paleta de 110 cores — fica em ~7 KB cada):

```python
from PIL import Image; import numpy as np, os
for f in sorted(os.listdir('saida')):
    arr = np.array(Image.open(f'saida/{f}').convert('RGB')).astype(float)
    alpha = np.clip((252 - arr.min(axis=2)) / 22.0, 0, 1)
    ys, xs = np.where(alpha > 0.06)
    lado = max(ys.max()-ys.min(), xs.max()-xs.min())
    cy, cx = (ys.min()+ys.max())//2, (xs.min()+xs.max())//2; m = int(lado*0.56)
    img = Image.fromarray(np.dstack([arr, alpha*255]).astype(np.uint8), 'RGBA') \
               .crop((max(0,cx-m), max(0,cy-m), min(420,cx+m), min(420,cy+m)))
    quad = Image.new('RGBA', (max(img.size),)*2, (0,0,0,0))
    quad.paste(img, ((quad.width-img.width)//2, (quad.height-img.height)//2))
    quad.resize((300,300), Image.LANCZOS).quantize(colors=110, method=Image.FASTOCTREE) \
        .save(f'../assets/presentes/{f}', optimize=True)
```

## Testes

```bash
node tests/testar_pix.js        # CRC, campos do BR Code, catálogo (valores
                                # únicos e múltiplos de 10, imagem existente),
                                # comparação com o Pix real do C6 e com uma
                                # implementação independente em Python
python3 tests/testar_pagina.py  # ids que o JS procura existem no HTML,
                                # classes têm estilo, arquivos existem
node tests/testar_navegador.js  # Chromium headless em viewport de celular:
                                # abre um presente, confere o código na tela,
                                # copia, valida o formulário, fecha com Esc.
                                # Capturas em tests/capturas/
```

`testar_navegador.js` aceita `CHROME=` e `PUPPETEER=` por variável de ambiente.

## Pendências conhecidas

- `CONFIG.recadosEndpoint` está vazio: o Apps Script ainda não foi implantado
  (a Ana estava no celular). Até lá o formulário avisa em vez de fingir.
- **O Pix ainda não foi testado num banco de verdade nesta versão.** O formato
  bate campo a campo com o código real do C6, mas isso é conferência, não
  prova. Testar antes de divulgar o link.
- A Ana tinha mais imagens do save the date (com as fontes e os dizeres); só
  chegou a ilustração. Se as outras aparecerem, vale reavaliar a tipografia.
- Não há QR code, só o copia e cola — o público está no celular, onde o QR
  seria inútil.

## Coisas a não fazer

- Não introduza framework, bundler ou `npm install` no site.
- Não coloque os recados no site público. A Ana pediu que só ela veja.
- Não repita valores entre presentes: é o valor que identifica o presente.
- Não invente txid no Pix sem testar num banco. Foi o que quebrou a v1.
- Não invente "presente já escolhido" sem backend: sem estado compartilhado,
  a marcação mentiria para o convidado.

---
> Source: [andred-br/Cha-juju](https://github.com/andred-br/Cha-juju) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
