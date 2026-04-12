## socratesonline

> - O SISTEMA USA POSTGRES, utilizar somente esse banco pra qualquer alteração


- O SISTEMA USA POSTGRES, utilizar somente esse banco pra qualquer alteração
- Sempre respeitar o design já utilizado para as demais páginas e manter um padrão, inclusive de paleta de cores
- Sempre que criar uma nova ação dentro do sistema, criar chamada para gravar um log na tabela log_sistema;
- Pensar sempre que tudo deve funcionar responsivamente, para desktop e mobile
- Ao aplicar filtro de datas, usar o filtro de datas padrão para todas as páginas
- Sempre que houver uma mudança em uma tabela de banco de dados, revisar os forms, os models, os templates e os demais locais onde a tabela é acessada, e tbm fazer a migração do database se necessário.
- Nas criações de tabelas sempre deverá haver um campo unico de id na tabela (verificar os moldes já criados e seguir o mesmo padrão);
- Sempre que criar uma nova entidade ou elaborar uma exclusão, analisar as dependências - proteção de dependências: PROTEGER registros que estão sendo usados (não permitir exclusão) / PERMITIR exclusão em cascata apenas de pai para filho;
- Sempre pensar em código reutilizável ao escrever qualquer lógica, evitando código repetido (DRY);
- Não reverter funcionalidades que já foram aprovadas pelo usuário sem perguntar antes.
- Todos os relatórios devem ter opção de exportar para PDF e excel;
- Todos os cadastros devem seguir o mesmo padrão, com listagemdos dados, filtro para busca livre, filtro avançado com todos os campos e ordenação por campos, além de exportação para pdf e excel;
- sempre executar comandos de powershell etc separadamente 
- sempre que criar uma funcionalidade, verificar se existem padrões já implementados no sistema e repetir (ex: botões de excluir, editar, etc)
- todos os ajustes que forem feitos no sistema, considerar a versão produção (VPS Oracle Cloud) e aplicar as instruções se for necessário
- nunca realizar subida pro git automática
- SEMPRE que criar uma nova rota, incluir na tela de permissões como opção, e na de exibição também.

### Regras para criar e aplicar migrações (Alembic/Flask-Migrate) neste projeto

- Sempre gerar migrações via CLI; editar arquivos manualmente só em último caso.
- O valor de `down_revision` deve ser o ID da revisão anterior (conteúdo da variável `revision` no arquivo), NUNCA o nome do arquivo.
- Antes de criar uma migração, garanta que o banco está no último head.

#### 1) Pré-requisitos
- Ativar venv e setar app:
```powershell
cd "C:\Users\comer\Desktop\Python - Projects\socrates_online"
.\venv\Scripts\Activate.ps1
$env:FLASK_APP="app.py"
```

#### 2) Conferir estado de migrações
- Ver head atual:
```powershell
flask db heads
```
- Ver revisão aplicada no banco:
```powershell
flask db current
```

#### 3) Fluxo padrão (autogeração)
- Alterar modelos em `models.py`.
- Gerar migração:
```powershell
flask db migrate -m "Descrição da mudança"
```
- Aplicar:
```powershell
flask db upgrade
```

#### 4) Se precisar criar/ajustar migração manualmente
- Use o head atual como `down_revision`. Exemplo de cabeçalho correto:
```python
# migrations/versions/xxxx_add_nova_coluna.py
revision = 'xxxx_add_nova_coluna'
down_revision = 'adicionar_campo_media_km_litro_veiculo'  # ID (revision) da migração head atual
```
- Em SQLite, alterações de coluna devem usar batch_alter:
```python
from alembic import op
import sqlalchemy as sa

def upgrade():
    with op.batch_alter_table('colaborador') as batch_op:
        batch_op.add_column(sa.Column('telefone', sa.String(), nullable=True))
        batch_op.add_column(sa.Column('email', sa.String(), nullable=True))

def downgrade():
    with op.batch_alter_table('colaborador') as batch_op:
        batch_op.drop_column('email')
        batch_op.drop_column('telefone')
```
- Aplicar:
```powershell
flask db upgrade
```

#### 5) Se houver múltiplos heads (divergência)
- Detectar:
```powershell
flask db heads
```
- Criar migração de merge (une ramificações):
```powershell
flask db merge -m "merge heads"
```
- Verifique que o novo merge virou `head`, então crie sua migração normal apontando `down_revision` para esse merge.
- Aplicar:
```powershell
flask db upgrade
```

#### 6) Verificações e correções rápidas
- “no such column …”: migração não aplicada. Rodar:
```powershell
flask db upgrade
flask db current
```
- Se `flask db` falhar por dependência:
```powershell
pip install -r requirements.txt
```

#### 7) Convenções deste projeto
- Manter consistência nos IDs de revisão; podem ser legíveis (ex.: `adicionar_campo_xyz`), mas devem ser únicos.
- Nunca referenciar nomes de arquivos em `down_revision`; sempre o conteúdo de `revision` da migração anterior.
- Em SQLite, preferir `op.batch_alter_table` para adicionar/alterar colunas.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mariacarolk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mariacarolk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
