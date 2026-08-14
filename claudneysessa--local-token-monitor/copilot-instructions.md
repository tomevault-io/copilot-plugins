## local-token-monitor

> - Projeto: Local Token Monitor

# AGENTS.md — Local Token Monitor

## Contexto

- Projeto: Local Token Monitor
- Pasta: `E:\portfolio-governance\projects\local-token-monitor`
- Repositório: `claudneysessa/local-token-monitor`
- Demonstração: **não se aplica** — ver "Restrições"
- Idioma principal: inglês (`README.md`), com paridade em `README.pt-BR.md`
- Estado: publicado; leitura, custo e testes validados

## Regras específicas

- Ler `PROJECT_STATE.md`, README e `CHANGELOG.md` antes de alterar.
- Respeitar os padrões da raiz `E:\portfolio-governance`.
- Atualizar os dois idiomas no mesmo ciclo.
- Registrar cada ciclo relevante no `CHANGELOG.md`, com data e hora reais obtidas
  de `Get-Date` — nunca estimadas. Entradas novas no topo.
- Ao mudar preço, multiplicador de cache ou fonte de rótulo, atualizar os testes
  no mesmo commit: são eles que sustentam as afirmações do README.
- **Nunca editar o `hud.html`.** É artefato compilado, arquivo único de ~865 KB com
  as fontes embutidas — não ler, não formatar, não passar linter. Mudança no HUD
  se pede ao autor do design, que regenera o bundle inteiro: substitui-se o
  arquivo e reconstrói a imagem, nunca se aplica diff.
- **Conferir tag com hífen antes de implantar bundle novo.** `<sc-for>` e `<x-dc>`
  estão em todos os builds e são inertes — o JavaScript monta a lista de sessões
  sozinho. Qualquer outra é suspeita: já chegou um bundle com o filtro
  de projeto exportado como `<sc-raw-select>`, sem `customElements.define`, o que
  deixaria o HUD preso em "modo demo" em silêncio. Nesse caso se pede reexportação.

```bash
# nenhuma tag com hifen alem das tres conhecidas
grep -oE "<[a-z]+-[a-z-]+" hud.html | sort -u
```

## Validação

```text
python -m unittest discover -v
python -m compileall -q monitor.py test_monitor.py
python monitor.py            # e abrir http://127.0.0.1:8099
docker compose up -d --build   # e abrir http://127.0.0.1:8099
# / e /hud servem o HUD; /painel serve o painel; /api devolve o snapshot
docker compose down
```

## Documentos obrigatórios

```text
README.md
README.pt-BR.md
CHANGELOG.md
PROJECT_STATE.md
LICENSE
```

## Restrições

```text
Só biblioteca padrão. Uma dependência derruba a proposta de "baixe um arquivo e
rode"; se algo parecer exigir pacote externo, resolver com stdlib ou não fazer.

Só leitura de arquivo local. Nada de proxy, credencial, telemetria ou requisição
de saída. É isso que torna seguro apontar a ferramenta para o próprio trabalho —
e é o que permite calcular custo sem tocar em conta nenhuma.

Sem GitHub Pages e sem demo online, por consequência do desenho: o painel lê o
histórico da máquina de quem executa. Uma versão hospedada não teria dados, e
fabricar dados de exemplo violaria o padrão de mídia. A seção de execução local
cumpre o papel da demonstração e precisa continuar testável por quem clonou.

Valores em dinheiro são equivalente-API, não fatura. Em planos por assinatura a
cota não é denominada em dólares de API. O seletor de moeda começa desligado de
propósito; não inverter esse padrão sem decisão explícita.
```

---
> Source: [claudneysessa/local-token-monitor](https://github.com/claudneysessa/local-token-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
