## oficina-realtime

> Painel "Central de Inteligência — Resumo Oficina" da MAAS. Dashboard de TV

# CLAUDE.md — contexto para o Claude Code

## O que é este projeto

Painel "Central de Inteligência — Resumo Oficina" da MAAS. Dashboard de TV
(1920×1080, estética control room em petróleo #183B48) que mostra a operação
da oficina em tempo quase real: o backend consulta o BigQuery a cada 5 min,
calcula os KPIs e empurra o JSON por WebSocket para as telas abertas.

## Arquitetura

```
BigQuery (dataset silver): STJ_Manutencao, TQB_Monitoramento, ST9_CadastroBem,
                           SRA_SRJ_Funcionarios, STF_Status_Manutencao, TQR
                           (TTI_Portaria APOSENTADA — feed parou em 07/04/2026)
   → backend/app/bq.py        (consultas — passo "Fonte")
   → backend/app/kpis.py      (modelagem/regras — as "medidas DAX")
   → backend/app/main.py      (FastAPI: polling + WebSocket /ws + estáticos)
   → frontend/Resumo_Oficina.dc.html  (recebe via WS e re-renderiza)
```

As regras de KPI replicam as medidas DAX do painel de referência **`manutest.vpax`**
(VertiPaq Analyzer). O painel `Painel Maas_Atualizado…vpax` é o antigo e estava
com valores INCORRETOS — não use como gabarito.

Fontes alternáveis por `DATA_SOURCE` no `backend/.env`:
`mock` (demo) · `csv` (lê `backend/dados/*.csv`, validação offline) ·
`bigquery` (produção).

## Regras que NÃO devem ser quebradas

- **Nenhuma credencial em código ou no repositório.** Autenticação só via
  `GOOGLE_APPLICATION_CREDENTIALS` no `.env` (git-ignorado) ou ADC do gcloud.
  O frontend nunca fala com o BigQuery.
- **Contrato do payload**: as chaves do JSON (`kpis`, `tipoServico`, `aging`,
  `detalhes`, `mecanicos`, `veiculos`, `geradoEm`) são compartilhadas entre
  `kpis.py`, `mock.py` e o `renderVals()` do frontend. Mudou de um lado, mude
  dos três. (`veiculos` = {mobilizados, naoMobilizados}; ainda sem bloco no
  frontend novo.)
- **Identidade visual intocada**: paleta petróleo/verde MAAS, tipografia e
  componentes do protótipo vBeta. Novos elementos usam as variáveis CSS
  existentes (`--maas-*`, `--surface-*`), nunca cores hardcoded novas.
- O frontend usa um runtime próprio (`support.js`, não editar — é gerado):
  template com `{{ … }}`, `sc-if`, `sc-for`, e um `class Component extends
  DCLogic` com `renderVals()` dentro do próprio HTML.

## Fatos do schema (regras DAX do manutest.vpax, validadas em 21/07/2026)

- **Ordem ABERTA = `termino='N'` e `situacao≠'C'`; cada O.S. conta** (ver
  `_abertas_os` em `kpis.py`). DESVIA do `qtdRep=0` do manutest: aquele filtro
  escondia O.S. reprovadas (qtdRep>0) ainda abertas — 4 sumiam em 23/07/2026. Uma
  solicitação pode ter mais de uma O.S. aberta (ex.: S.S. 032169 → O.S. 038144 e
  038145) e a operação conta como 2 O.S. (definido em 23/07/2026), então NÃO
  deduplicamos por solicitação. Resultado: 61 O.S. (vs 56 com qtdRep=0).
- **Abertura da O.S. = entrada da S.S.** (`TQB_Monitoramento.DtAbertura`+`Hoaber`,
  data+hora LOCAL), NÃO o `dtOrigem` da O.S. (data em que a O.S. foi recortada, que
  muda em reprovação e vem sem hora). `DataHoraAbertura` guarda hora local rotulada
  como UTC — NÃO converter fuso (atrasa 3h). Ver `_abertura_ss`.
- Serviço: 000001 Corretiva · 000002 Sinistro · 000003 Preventiva ·
  000004 Implementação · 000005 Socorro.
- `codBem` é o veículo; `ST9_CadastroBem.placa` traz a placa (join
  `TQB_Monitoramento.Codbem = ST9_CadastroBem.bem`).
- Junções: `STJ_Manutencao.ordem = TQB_Monitoramento.ordemSTJ`;
  `TQB_Monitoramento.Codbem = ST9_CadastroBem.bem`;
  reserva no limite: `TQB_Monitoramento.Xbemre = ST9_CadastroBem.bem`.
- **Status do bem** (`ST9_CadastroBem.statusBem`): 01 Locado · **02 Reserva** ·
  03 Serviços · 04 Disponível · 05 Negociado · 06 Venda · 07 Vendido ·
  08 Em Adequação · 10 Aguardando Demanda · 12 Distratado.

### Fonte/regra de cada KPI (medidas DAX reproduzidas em `kpis.py`)

- **OS Abertas** = COUNT ordem abertas (`termino='N'` e `situacao≠'C'`, cada O.S.
  conta — ver acima; não é mais `qtdRep=0`).
- **OS Fora do Prazo** = `agora > SLAVencimentoOS` (recalculado AO VIVO a cada ciclo,
  não usa o flag snapshot `SLAUltrapassadoOS`) + O.S. aberta
  **+ serviço ∉ {Implementação}**. Só Implementação não tem obrigação de prazo e
  aparece como "Sem prazo" no detalhamento — ver `SERVICOS_SEM_SLA` em `kpis.py`.
  **Sinistro ENTRA no cálculo** (desde 23/07/2026; antes ficava de fora junto com
  Implementação). Ainda desvia do DAX do manutest, que contava todos os serviços.
- O serviço de uma ordem vem do **STJ** (`NomeServico`/`servico`), não do
  `nmServ` da TQB: em ~3 ordens abertas as fontes divergem e só o STJ tem o
  código do serviço. Usar a mesma fonte na coluna "Tipo" e na regra de SLA
  evita a linha exibir um serviço e ser classificada por outro
  (ver `_servico_da_ordem`).
- **Cláusula** = DISTINCTCOUNT `codBem`: `agora > SLAVencimentoCC` (AO VIVO),
  `xss=''`, `nmServ≠Implementacao`, ST9 `numeroContrato<>''` e `statusBem ∉ {08,02}`, qtdRep=0.
- **Controle de Qualidade** = COUNT ordem, `tipoRet='A'`.
- **Retorno** = COUNT, qtdRep=0 e `xRetorn='1'`.
- **Clientes Esp.** = DISTINCTCOUNT codBem, qtdRep=0 e TQB `Xesper='S'`.
- **S.O.S** = COUNT, qtdRep=0 e `servico='000005'`.
- **S.S. Aguardando** = COUNT TQB onde `StatusOS` NÃO contém "Aberta".
- **Veículos Mob/Não** = DISTINCTCOUNT codBem por TQB `Xcontr` preenchido/vazio.
- **Mão de Obra** = DISTINCTCOUNT `SRA_SRJ_Funcionarios.RA_MAT` por `StatusFinal`
  (Disponível/Trabalhando/Intervalo).
- **Preventivas** = DISTINCTCOUNT `STF_Status_Manutencao.codBem` por `statusManutencao`
  (Atrasado / Período Final / Período Inicial).
- **Reservas no Limite** = medida `Qtd_Res_Limite` (validado =2 em 23/07/2026).
  Frota de reserva é por **contrato+lote**. Estoque = `ST9_CadastroBem` com
  `statusBem='02'` (Reserva) por `numeroContrato`+`numeroLote`; "em uso" = reserva
  (02) que é `Xbemre` de uma O.S. aberta (`TQB_Monitoramento`, `termino='N'`);
  NO LIMITE = grupos com em uso ≥ 1 e estoque − em uso ≤ 0. O reserva continua 02
  enquanto substitui; só baixa por sinistro (o cadastro muda o status).
  **APOSENTOU a `TTI_Portaria`** (portaria): a `raw.TTI` parou em 07/04/2026 e o
  `codVei2` de lá congelou. Ver `_reservas_limite` em `kpis.py`.

## Como rodar e testar

```bash
cd backend
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env          # DATA_SOURCE=csv para validar offline
python -m uvicorn app.main:app --reload --port 8000
```

Smoke test: `GET /healthz` responde `{"ok":true,…}`; `GET /api/resumo` traz o
payload; abrir http://localhost:8000 mostra o painel com o indicador verde
"ao vivo". Não há suíte de testes formal ainda — se criar, use `pytest` em
`backend/tests/` com casos sobre `kpis.build_payload` (linhas sintéticas).

## Pendências conhecidas (contexto para futuras tarefas)

- Todos os KPIs de card já têm fonte automática (não usam mais `.env`); os
  valores `KPI_*_MANUAL`/`MECANICOS_*` no `.env` só servem de fallback offline
  quando a tabela-fonte não está disponível (ex.: modo `csv` sem os CSVs).
- **Tipo de veículo (Pesada/Leve)**: fonte é `TQR.TQR_CATBEM` via
  `ST9_CadastroBem.tecnologia = TQR.TQR_TIPMOD` — ainda não implementado.
- O frontend novo (redesign) NÃO tem os blocos "Veículos mobilizados/não",
  "Localização dos veículos" nem "Tipo de veículo" do painel de TV original;
  o backend já calcula `veiculos` mas falta a UI.
- O painel calcula tudo AO VIVO: OS Fora do Prazo e Cláusula usam
  `agora > SLAVencimento*` (não o flag `SLAUltrapassado*`, que é snapshot da
  ingestão). Por isso pode divergir de propósito de um snapshot do `manutest`.
- O bloco "Veículos" tem drill-down (`detalhes.veiculos`): clica no cabeçalho e
  abre a lista de veículos (placa + nome via `ST9_CadastroBem`, O.S., S.S., serviço).

---
> Source: [ygormaas/oficina-realtime](https://github.com/ygormaas/oficina-realtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
