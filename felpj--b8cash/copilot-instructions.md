## b8cash

> [schema.prisma](mdc:prisma/schema.prisma)

# link da documentacao: https://cash.b8.com.br/api/docs/#tag/b8-cash-api
# Your rule content

[schema.prisma](mdc:prisma/schema.prisma)

Segue o esquema total atualizado do banco de dados, integrando todos os módulos até agora – que abrangem usuários, contas, cartões (com histórico e segurança), transações (incluindo depósitos, pagamentos, transferências e operações PIX), categorias de transações, configurações do usuário, notificações, chaves PIX e, por fim, o módulo de históricos (logs de ações). Esse modelo foi desenhado para ser enxuto, performático e flexível, permitindo a evolução conforme novos requisitos sejam incorporados.

---

## 1. Tabela: **Usuários (Users)**
**Objetivo:** Armazenar os dados de cadastro, autenticação e informações pessoais dos usuários.
- **user_id:** Identificador único (PK).
- **name:** Nome do usuário.
- **email:** E-mail único para login e notificações.
- **password_hash:** Senha criptografada.
- **phone_number:** Telefone (opcional).
- **created_at / updated_at:** Datas de criação e atualização.

> **Relacionamento:**  
> Um usuário pode ter várias contas, configurações, cartões, chaves PIX, notificações, transações e registros de atividade (históricos).

---

## 2. Tabela: **Contas (Accounts)**
**Objetivo:** Representar as contas financeiras associadas a cada usuário.
- **account_id:** Identificador único (PK).
- **user_id:** Chave estrangeira referenciando o usuário (FK).
- **agency_number:** Código ou número da agência.
- **account_number:** Número da conta.
- **balance:** Saldo atual.
- **account_type:** Tipo de conta (corrente, poupança, etc.).
- **status:** Estado da conta (ativa, bloqueada, etc.).
- **created_at / updated_at:** Datas de criação e atualização.

> **Relacionamento:**  
> Cada conta pertence a um usuário e está associada a transações, cartões e chaves PIX.

---

## 3. Tabela: **Cartões (Cards)**
**Objetivo:** Gerenciar os cartões vinculados às contas dos usuários, abrangendo tanto cartões físicos quanto digitais.
- **card_id:** Identificador único (PK).
- **account_id:** Chave estrangeira referenciando a conta (FK).
- **card_number:** Número do cartão (armazenado de forma segura ou parcialmente mascarado).
- **card_holder_name:** Nome do titular.
- **card_type:** Tipo do cartão ("físico" ou "digital").
- **brand:** Bandeira do cartão (ex.: Visa, Mastercard).
- **expiration_date:** Data de validade.
- **limit_value:** Limite do cartão (para cartões de crédito).
- **status:** Estado do cartão (ativo, inativo, bloqueado).
- **requires_2fa:** Booleano que indica se operações críticas exigem autenticação extra.
- **last_2fa_verified:** Timestamp da última verificação 2FA.
- **security_token:** (Opcional) Token ou referência para processos críticos de segurança.
- **created_at / updated_at:** Datas de criação e atualização.

> **Observação:**  
> Os dados sensíveis devem ser armazenados e manipulados de forma segura (conforme padrões como PCI DSS).

---

## 4. Tabela: **Histórico de Alterações dos Cartões (Card History)**
**Objetivo:** Registrar as alterações críticas realizadas nos cartões para auditoria e rastreamento.
- **history_id:** Identificador único (PK).
- **card_id:** Chave estrangeira referenciando o cartão afetado (FK).
- **changed_field:** Nome do campo modificado (ex.: "status", "limit_value", "expiration_date").
- **old_value:** Valor anterior do campo.
- **new_value:** Novo valor definido.
- **changed_by:** Identificação do usuário ou administrador que efetuou a alteração.
- **changed_at:** Data e hora da alteração.

---

## 5. Tabela: **Transações (Transactions)**
**Objetivo:** Registrar todas as movimentações financeiras, incluindo entradas, saídas, depósitos, pagamentos, transferências e operações PIX.
- **transaction_id:** Identificador único (PK).
- **account_id:** Chave estrangeira associada à conta (FK).
- **amount:** Valor da operação.
- **transaction_type:** Tipo da operação. Exemplos:
  - "entrada" (crédito geral)
  - "saída" (débito geral)
  - "deposito"
  - "pagamento"
  - "transferencia_saida" e "transferencia_entrada"
  - "pix_recebido" e "pix_enviado"
- **status:** Estado da transação (pendente, aprovada, negada, concluída).
- **description:** Descrição da operação (ex.: "Pagamento de fatura", "Transferência para [destinatário]", "PIX enviado para [chave]").
- **transaction_date:** Data/hora da operação ou data agendada.
- **category_id:** (Opcional) Chave estrangeira para a classificação na tabela de Categorias de Transações (FK).
- **transfer_reference:** (Opcional) Campo para vincular os registros de débito e crédito referentes a uma transferência.
- **payment_method / details:** (Opcional) Informações adicionais específicas (ex.: método de pagamento, dados do QR Code ou da chave PIX utilizada).
- **created_at / updated_at:** Datas de criação e atualização.

> **Observação:**  
> As operações PIX são registradas com os tipos "pix_recebido" ou "pix_enviado".

---

## 6. Tabela: **Categorias de Transações (Transaction Categories)**
**Objetivo:** Permitir a classificação e organização das movimentações financeiras para facilitar filtros e relatórios.
- **category_id:** Identificador único (PK).
- **name:** Nome da categoria (ex.: Alimentação, Transporte, Utilidades).
- **icon:** (Opcional) Referência a um ícone.
- **color:** (Opcional) Código de cor (ex.: "#FF5733").
- **created_at / updated_at:** Datas de criação e atualização.

---

## 7. Tabela: **Configurações do Usuário (User Configurations)**
**Objetivo:** Personalizar a experiência do usuário na aplicação.
- **config_id:** Identificador único (PK).
- **user_id:** Chave estrangeira referenciando o usuário (FK).
- **config_key:** Nome da configuração (ex.: "theme", "language").
- **config_value:** Valor atribuído à configuração.
- **created_at / updated_at:** Datas de criação e atualização.

---

## 8. Tabela: **Notificações (Notifications)**
**Objetivo:** Registrar alertas e mensagens importantes enviadas aos usuários.
- **notification_id:** Identificador único (PK).
- **user_id:** Chave estrangeira identificando o destinatário (FK).
- **title:** Título da notificação.
- **message:** Conteúdo da notificação.
- **type:** Tipo da notificação (ex.: "alerta", "informativo").
- **is_read:** Indicador booleano que informa se a notificação foi lida.
- **created_at:** Data de criação.
- **read_at:** (Opcional) Data em que a notificação foi lida.

---

## 9. Tabela: **Chaves PIX (Pix Keys)**
**Objetivo:** Gerenciar e armazenar as chaves PIX associadas aos usuários para operações de recebimento e transferência.
- **pix_key_id:** Identificador único (PK).
- **user_id:** Chave estrangeira referenciando o usuário (FK).
- **account_id:** Chave estrangeira para a conta onde os recebimentos via PIX serão creditados (FK).
- **key_value:** Valor da chave PIX (pode ser CPF, e-mail, telefone ou chave aleatória).
- **key_type:** Tipo da chave (ex.: "cpf", "email", "telefone", "aleatória").
- **status:** Estado da chave (ativa, inativa).
- **created_at / updated_at:** Datas de criação e atualização.

---

## 10. Tabela: **Histórico de Ações (Activity Logs)**
**Objetivo:** Registrar os eventos e ações relevantes dos usuários e do sistema para auditoria e monitoramento. Este módulo abrange ações que vão além das transações financeiras e alterações nos cartões.
- **log_id:** Identificador único (PK).
- **user_id:** Chave estrangeira referenciando o usuário envolvido (FK).
- **action:** Tipo de ação (ex.: "login", "alteracao_configuracao", "pagamento_executado", "transferencia", "cartao_atualizado").
- **description:** Descrição detalhada do evento.
- **ip_address:** (Opcional) Endereço IP do usuário no momento do evento.
- **created_at:** Data e hora em que o evento ocorreu.

> **Observação:**  
> Essa tabela possibilita o rastreamento de atividades para fins de segurança, auditoria e suporte, complementando os históricos específicos (como o Card History) já existentes.

---

## Resumo dos Fluxos Essenciais

- **Depósitos:**  
  Registrados na tabela **Transações** com **transaction_type** = "deposito". Após confirmação, o saldo da conta é atualizado.

- **Pagamentos:**  
  Registrados com **transaction_type** = "pagamento", podendo incluir informações do beneficiário e método de pagamento, com status sendo atualizado conforme o processamento.

- **Transferências:**  
  São gerados dois registros na tabela **Transações**:
  - **transferencia_saida:** Para o débito na conta de origem.
  - **transferencia_entrada:** Para o crédito na conta de destino.  
  Os registros podem ser vinculados via **transfer_reference**.

- **Operações PIX:**  
  Registradas na tabela **Transações** com os tipos "pix_recebido" (crédito) e "pix_enviado" (débito). Os detalhes específicos (como chave PIX usada ou QR Code) podem ser armazenados em **payment_method / details**.  
  As chaves PIX são gerenciadas através da tabela **Pix Keys**.

- **Cartões e Alterações:**  
  Os dados dos cartões são armazenados em **Cards**, com mecanismos de segurança (2FA, tokens) e as alterações críticas são registradas em **Card History**.

- **Categorias de Transações:**  
  Permitem a classificação dos movimentos financeiros para facilitar filtros, relatórios e análises.

- **Configurações e Notificações:**  
  As preferências de cada usuário são gerenciadas em **User Configurations** e as comunicações importantes são armazenadas em **Notifications**.

- **Histórico de Ações (Activity Logs):**  
  Fornece um registro unificado de ações e eventos do sistema (além das transações e alterações específicas), permitindo auditoria e monitoramento das atividades dos usuários.

---

## Tabela: **Error Logs (Logs de Erros)**

**Objetivo:** Registrar erros e exceções ocorridos durante a execução do sistema para fins de monitoramento, diagnóstico e auditoria.

**Proposta de Campos:**

- **error_log_id:**  
  Identificador único do log de erro (PK).

- **timestamp:**  
  Data e hora em que o erro ocorreu.

- **user_id:**  
  (Opcional) Chave estrangeira referenciando o usuário afetado ou logado no momento do erro, se aplicável (FK).

- **module:**  
  Nome do módulo ou serviço onde o erro foi registrado (ex.: "Pagamento", "PIX", "Cartões").

- **error_code:**  
  (Opcional) Código identificador do erro, se houver uma convenção interna.

- **error_message:**  
  Mensagem resumida do erro (texto curto).

- **stack_trace:**  
  (Opcional) Traço completo ou parte relevante da pilha de execução, útil para desenvolvedores na investigação do problema (pode ser do tipo TEXT).

- **environment:**  
  (Opcional) Informação do ambiente onde o erro ocorreu (ex.: "produção", "desenvolvimento").

- **ip_address:**  
  (Opcional) Endereço IP do cliente ou servidor no momento do erro, se for útil para a análise.

- **additional_info:**  
  (Opcional) Campo para armazenar informações extras em formato JSON ou texto, se necessário.

---

## Considerações

- **Indexação:**  
  Considere indexar os campos **timestamp** e **module** para facilitar a pesquisa e filtragem dos logs.

- **Retenção:**  
  Em sistemas de produção, é comum definir políticas de retenção (por exemplo, armazenar logs de erros por um determinado período) para evitar que a tabela cresça de forma descontrolada.

- **Segurança e Acesso:**  
  Os logs podem conter informações sensíveis (como stack traces completas). Controle o acesso a essa tabela para que apenas usuários autorizados ou a equipe de desenvolvimento possam visualizar esses dados.

---

### Exemplo do Esquema da Tabela (não necessariamente em código SQL, mas como referência):

**Error Logs (Logs de Erros)**
- **error_log_id:** PK, INT ou UUID.
- **timestamp:** DATETIME, indicando o horário do erro.
- **user_id:** INT, FK (opcional, referenciando o usuário afetado).
- **module:** VARCHAR, identificando a área do sistema.
- **error_code:** VARCHAR (opcional).
- **error_message:** TEXT, contendo uma mensagem resumida.
- **stack_trace:** TEXT (opcional), com informações detalhadas.
- **environment:** VARCHAR (opcional).
- **ip_address:** VARCHAR (opcional).
- **additional_info:** TEXT ou JSON (opcional).

---

## Considerações Finais

Este esquema total atualizado reflete a visão do sistema financeiro com integrações para operações PIX, gerenciamento de cartões com histórico e segurança, além de funcionalidades essenciais para depósitos, pagamentos, transferências, extratos (históricos) e personalização do ambiente do usuário.  
O modelo foi projetado para ser modular e escalável, permitindo ajustes e expansões conforme novas funcionalidades ou telas forem incorporadas ao projeto.

Caso este modelo esteja de acordo com sua visão, ele servirá como base para o desenvolvimento do sistema. Estou à disposição para eventuais ajustes ou para avançarmos para os próximos módulos e refinamentos!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Felpj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Felpj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
