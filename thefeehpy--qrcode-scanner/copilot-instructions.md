## qrcode-scanner

> - Aplicação web estática em HTML, CSS e JavaScript puro; não há `package.json`, build ou testes automatizados.

# Instruções para agentes

## Contexto do projeto

- Aplicação web estática em HTML, CSS e JavaScript puro; não há `package.json`, build ou testes automatizados.
- [README.md](README.md) documenta a execução e o comportamento esperado.
- [index.html](index.html) define os IDs e a estrutura consumidos por [app.js](app.js); preserve esses contratos ao atualizar a interface.
- [app.js](app.js) separa responsabilidades em `URLSafety`, `ImagePipeline`, `QRDetector`, `UIController`, `FileScanner`, `ScreenScanner` e `QRScannerApp`.
- `jsQR.js` fica na raiz do projeto. Não introduza referências a `vendor/jsQR.js` sem também corrigir a documentação e o carregamento real.

## Como validar atualizações

1. Sirva a pasta pela raiz com `python3 -m http.server 8080`.
2. Abra `http://localhost:8080` no Chrome ou Edge.
3. Faça o teste manual com `test-qr-exemplo.png`; confirme a leitura, a exibição do conteúdo e a abertura segura de URLs.
4. Para alterações em captura de tela ou PDF, valide também permissões de `getDisplayMedia` e o carregamento do `pdf.js` via CDN.

## Regras de implementação

- Mantenha o processamento local no navegador; não envie imagens, frames ou conteúdo decodificado para servidores.
- Preserve a validação de URLs em `URLSafety`: somente `http:` e `https:` podem ser abertos, sempre com `noopener,noreferrer`.
- Exiba conteúdo lido com APIs seguras como `textContent`; não use `innerHTML` para dados do QR Code.
- Evite duplicar lógica de detecção: reutilize `QRDetector` e `ImagePipeline` para imagens, PDFs e frames de tela.
- Considere concorrência, cancelamento, permissões e consumo de CPU ao alterar os fluxos de leitura.
- Mantenha o estilo existente no CSS: variáveis em `:root`, classes com convenção BEM e layout responsivo.
- Como não há suíte automatizada, registre no resumo da alteração quais fluxos manuais foram verificados e quais limitações de ambiente permaneceram.

---
> Source: [Thefeehpy/QRcode_scanner](https://github.com/Thefeehpy/QRcode_scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
