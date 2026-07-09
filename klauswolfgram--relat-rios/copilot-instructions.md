## relat-rios

> - Arquivos `.tlpp`, `.prw`, `.ch` e demais fontes Protheus/TOTVS devem ser mantidos em Windows-1252.

# Diretrizes Do Projeto

## Encoding

- Arquivos `.tlpp`, `.prw`, `.ch` e demais fontes Protheus/TOTVS devem ser mantidos em Windows-1252.
- Arquivos TypeScript, HTML, SCSS, JSON, Markdown e demais fontes do Angular devem ser mantidos em UTF-8.
- Ao editar fontes TLPP, preservar o encoding original do arquivo.
- Nao converter arquivos TLPP para UTF-8 sem autorizacao explicita.
- Ao adicionar textos acentuados em arquivos Angular, usar UTF-8 normalmente.
- Ao adicionar textos acentuados em arquivos TLPP, garantir que o arquivo continue salvo em Windows-1252.

## Orientacoes Gerais

- Nao editar arquivos TLPP quando a solicitacao for restrita ao aplicativo Angular.
- Quando uma mudanca envolver a fila de integracao, manter alinhados model, servico da fila, payload enviado ao ERP e tela da fila.
- Preservar alinhamentos existentes em declaracoes, atribuicoes e blocos tabulados. Quando adicionar novas linhas em um bloco ja alinhado, ajustar os espacos para manter colunas e operadores no mesmo padrao visual do entorno.
- Usar caracteres de espaco para indentacao, tabulacao e alinhamento; nao usar caracteres de tabulacao.

## Orientacoes Para TOTVS/Protheus

- Em fontes Protheus/TOTVS, quando uma user function for criada, usar a forma alternativa `Function U_NOME()` em vez de `User Function Nome()`. A palavra-chave deve ser escrita como `Function` e o nome da funcao deve estar totalmente em caixa alta, incluindo o prefixo `U_`.
- Ao criar ou renomear uma `Function U_NOME()` chamada por menu, manter o `ACTION` do menu, a documentacao Protheus.doc e o nome da funcao exatamente alinhados.
- Em fontes Protheus/TOTVS, `Static Function` deve usar camel case no nome da funcao, iniciando com letra minuscula e usando letra maiuscula no inicio de cada termo seguinte.
- Em fontes Protheus/TOTVS, variaveis, metodos e atributos de classes devem seguir o mesmo padrao camel case usado para nomes de `Static Function`. Chamadas de metodos tambem devem iniciar com letra minuscula, por exemplo `oBrowse:disableReport()` em vez de `oBrowse:DisableReport()`.
- Em fontes Protheus/TOTVS, no instanciamento de classes, nomes iniciados por `T` devem usar o primeiro caractere minusculo, como `tDialog():new()`. Classes iniciadas por `FW` ou `MS` devem usar o prefixo em minusculo, como `fwLayer():new()`, `fwBrowse():new()` e `msDialog():new()`.
- Em fontes Protheus/TOTVS, chamadas de funcoes seguem a mesma convencao de caixa dos metodos/classes: funcoes iniciadas por `FW` ou `MS` devem usar prefixo minusculo, como `fwGetDialogSize()`, e funcoes iniciadas por `T` devem iniciar com `t` minusculo quando aplicavel.
- Em arquivos TLPP, as variaveis e atributos devem ser declarados usando o tipo de dados esperado por eles, sendo as opcoes disponiveis: Character, Date, Numeric, Logical, Array, Object e Json. O que nao for possivel destacar como uma dessas possibilidades deve ser declarado como Variant.
- Em fontes Protheus/TOTVS, toda variavel atribuida ou usada dentro de uma funcao deve estar declarada no bloco inicial de declaracoes da funcao com o tipo adequado; nao introduzir variaveis implicitas durante a implementacao.
- Em fontes Protheus/TOTVS, declaracoes de variaveis devem preservar o padrao de alinhamento em colunas adotado no arquivo: `Local`, nome da variavel, `as` e tipo alinhados verticalmente. Separar grupos semanticos de declaracoes com uma linha em branco e manter o mesmo alinhamento nas atribuicoes subsequentes, alinhando operadores como `:=` e `+=` com o entorno.
- Em fontes Protheus/TOTVS, declaracoes de variaveis de escopo local (`Local`) devem ocorrer antes de qualquer outro comando dentro da funcao, inclusive antes de declaracoes `Private`, atribuicoes, `Default` ou qualquer chamada de rotina.
- Em acoes chamadas pelo menu de uma rotina/browse Protheus, considerar que o registro corrente visivel ja esta posicionado e nao deletado; evitar validacoes defensivas de `eof()`/`deleted()` nesse ponto. Reservar essas validacoes para loops, consultas diretas e rotinas sem interacao de usuario.
- Em rotinas Protheus/TOTVS que exibem regua de progressao, preferir `fwMsgRun(Nil,{|| rotina()},'Titulo','Mensagem')` em vez de `msAguarde(...)`.
- Em mensagens de interacao com usuario em Protheus/TOTVS, preferir as mensagens atuais `fwAlertInfo`, `fwAlertWarning`, `fwAlertError`, `fwAlertSuccess` e `fwAlertYesNo` em vez de `msgInfo`, `msgAlert`, `msgStop` e `msgYesNo`. Os parametros seguem o mesmo padrao de mensagem e titulo.
- Em rotinas Protheus/TOTVS, especialmente acoes chamadas por menu/browse, evitar `dbSelectArea()` para acessar tabelas auxiliares, pois isso altera a tabela principal da thread. Priorizar chamadas qualificadas por alias, como `Z09->(dbSetOrder(1))`, `Z09->(dbSeek(...))` e `(cAlias)->(dbCloseArea())`, preservando a area principal posicionada pela rotina. Quando uma operacao exigir a troca de area, como `COPY TO`, salvar a area atual com `getArea()` antes do `dbSelectArea()` e restaurar com `RestArea()` logo apos a operacao.
- Em fontes Protheus/TOTVS, chamadas `dbUseArea()` devem usar explicitamente o driver `"TOPCONN"`, no padrao `dbUseArea(.T.,"TOPCONN",cAlias,cAlias,.T.,.F.)`.
- Em fontes Protheus/TOTVS, nao posicionar diretamente no dicionario SX3 para recuperar metadados de campos. A TOTVS nao recomenda essa pratica. Usar `fwSx3Util():getAllFields('ALIAS',.F.)` para obter a lista de campos e `getSx3Cache(cCampo,'X3_ATRIBUTO')` para recuperar atributos como `X3_TITULO`, `X3_TIPO`, `X3_TAMANHO`, `X3_DECIMAL` e `X3_CBOX`. Quando precisar de tamanho/decimal de campo, priorizar `getSx3Cache()` em vez de posicionar ou consultar a SX3 diretamente.
- Em rotinas Protheus/TOTVS que usam `msExecAuto`, inicializar sempre a variavel `lMsErroAuto` com `.F.` antes da chamada. Essa variavel so tem seu conteudo alterado quando ocorre erro; se for inicializada com `.T.`, uma execucao sem erro pode ser interpretada incorretamente como falha.

## Orientacoes Para Angular

- Em templates HTML, quando o elemento nao tiver conteudo entre abertura e fechamento e a sintaxe permitir, priorizar a forma autocontida. Exemplo: usar `<po-divider />` em vez de `<po-divider></po-divider>`.
- Em templates Angular, priorizar o control flow novo com `@if` e `@for` em vez de `*ngIf` e `*ngFor`, quando a versao do Angular e o contexto do template permitirem.
- Ao criar novos artefatos Angular, seguir o padrao de nomes do projeto e nao usar sufixos antigos como `.component`, `.service`, `.module`, `.pipe` ou `.directive`.
- Quando um trecho de codigo puder ser escrito em uma unica linha sem prejudicar a legibilidade, priorizar essa forma. Exemplos: declaracoes simples de `const` e callbacks curtos como `finalize: () => this.loadingService.isHidden.set(true)`.
- Em classes TypeScript, metodos que nao sejam o `constructor` nem hooks de ciclo de vida do Angular devem ser declarados como arrow functions.
- Em componentes Angular, atributos e metodos acessados pelo template devem declarar explicitamente o modificador `public`, exceto hooks de ciclo de vida do Angular.
- Componentes Angular devem ser standalone e declarar dependencias diretamente em `imports`.
- Usar `inject()` para obter servicos nas classes em vez de injecao por parametros no `constructor`, mantendo o `constructor` apenas para inicializacao simples.
- Priorizar PO-UI em formularios e telas operacionais, mantendo o padrao visual existente do projeto.
- Formularios devem usar Reactive Forms com `FormBuilder`, seguindo os nomes dos campos dos models e do payload da fila.
- Rotas devem seguir o padrao ja usado em `app.routes.ts`, com `loadComponent` e arquivos sem sufixos tradicionais.
- Em telas filhas de projeto, obter o id do projeto por `ActivatedRoute` usando o parametro `projeto`.
- Ao abrir telas a partir da fila de integracao, respeitar o parametro de query `fila` para carregar e alterar o registro existente.
- Em componentes com inscricoes RxJS, usar `Subscription` e liberar em `ngOnDestroy`.
- Configuracoes de toolbar da pagina devem ser feitas pelo `ToolbarService`, como `showGoBackButton` e `title`.
- Quando criar novas mensagens e textos em arquivos Angular, usar acentos corretos em UTF-8.

---
> Source: [klauswolfgram/Relat-rios](https://github.com/klauswolfgram/Relat-rios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
