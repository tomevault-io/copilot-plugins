## treinopp-platform

> // ✅ Correto: Importar do pacote específico

---
description: 
globs: 
alwaysApply: false
---
# Melhores Práticas e Correções para Desenvolvimento

## 1. Importações e Dependências

### 1.1 Pacotes Externos

```typescript
// ✅ Correto: Importar do pacote específico
import { useBoolean } from 'minimal-shared/hooks';

// ❌ Incorreto: Importar do caminho errado ou inexistente
import { useBoolean } from '@hooks/useBoolean';
```

### 1.2 Componentes Internos

```typescript
// ✅ Correto: Usar caminhos absolutos começando com 'src/'
import { ConfirmDialog } from 'src/components/custom-dialog/confirm-dialog';

// ❌ Incorreto: Usar caminhos relativos complexos ou incorretos
import { ConfirmDialog } from '../../../components/confirm-dialog';
```

## 2. Estrutura de Arquivos e Componentes

### 2.1 Organização de Diretórios

```
src/
├── features/
│   └── [feature-name]/
│       ├── hooks/
│       │   ├── index.ts
│       │   ├── use-list-[resource].ts
│       │   └── use-[action]-[resource].ts
│       ├── sections/
│       │   ├── view/
│       │   │   └── [resource]-list-view.tsx
│       │   ├── [resource]-form.tsx
│       │   └── [resource]-table-row.tsx
│       └── types/
│           └── [resource].ts
└── components/
    └── [component-type]/
        └── [component-name].tsx
```

### 2.2 Nomenclatura de Arquivos

- Use kebab-case para arquivos: `detento-list-view.tsx`
- Use PascalCase para componentes: `DetentoListView`
- Use camelCase para hooks: `useListDetento`

## 3. Hooks e Estado

### 3.1 Hooks Personalizados

```typescript
// ✅ Correto: Retornar objeto com valores nomeados
export const useListResource = (params: ListResourceParams = {}) => {
  return {
    resources: data?.resources ?? [],
    isLoading,
    error,
    mutate,
  };
};

// ❌ Incorreto: Retornar array ou valores sem nome
export const useListResource = () => {
  return [data, loading, error];
};
```

### 3.2 Gerenciamento de Estado

```typescript
// ✅ Correto: Usar hooks específicos para cada tipo de estado
const confirmDialog = useBoolean();
const [currentResource, setCurrentResource] = useState<IResource | null>(null);

// ❌ Incorreto: Misturar estados ou usar useState para booleanos simples
const [isDialogOpen, setIsDialogOpen] = useState(false);
```

## 4. Tratamento de Erros e Loading

### 4.1 Operações Assíncronas

```typescript
// ✅ Correto: Usar try/catch com feedback ao usuário
const handleSubmit = async (data) => {
  try {
    await createResource(data);
    toast.success('Recurso criado com sucesso!');
    reset();
  } catch (error) {
    console.error(error);
    toast.error('Erro ao criar recurso');
  }
};

// ❌ Incorreto: Não tratar erros ou não fornecer feedback
const handleSubmit = async (data) => {
  await createResource(data);
  reset();
};
```

### 4.2 Estados de Loading

```typescript
// ✅ Correto: Mostrar feedback de loading
<LoadingButton
  type="submit"
  variant="contained"
  loading={isSubmitting}
>
  {currentResource ? 'Salvar' : 'Criar'}
</LoadingButton>

// ❌ Incorreto: Não mostrar estado de loading
<Button type="submit">
  Salvar
</Button>
```

## 5. Formulários e Validação

### 5.1 Schemas de Validação

```typescript
// ✅ Correto: Usar Zod com mensagens em português
const schema = z.object({
  nome: z.string().min(1, 'Nome é obrigatório'),
  email: z.string().email('Email inválido'),
});

// ❌ Incorreto: Mensagens em inglês ou validação inline
const schema = z.object({
  name: z.string().min(1, 'Name is required'),
});
```

### 5.2 Configuração do React Hook Form

```typescript
// ✅ Correto: Configuração completa com resolver
const methods = useForm<ResourceSchemaType>({
  resolver: zodResolver(schema),
  defaultValues,
  mode: 'onSubmit',
});

// ❌ Incorreto: Configuração incompleta
const methods = useForm();
```

## 6. Tabelas e Listagens

### 6.1 Definição de Colunas

```typescript
// ✅ Correto: Definir larguras e IDs consistentes
const TABLE_HEAD = [
  { id: 'nome', label: 'Nome', width: 200 },
  { id: 'email', label: 'Email', width: 200 },
];

// ❌ Incorreto: Sem larguras ou IDs inconsistentes
const TABLE_HEAD = [
  { id: 'Nome', label: 'Nome' },
  { id: 'EMAIL', label: 'Email' },
];
```

### 6.2 Paginação e Filtros

```typescript
// ✅ Correto: Usar parâmetros de paginação
const { resources } = useListResource({
  page: table.page + 1,
  limit: table.rowsPerPage,
});

// ❌ Incorreto: Não implementar paginação
const { resources } = useListResource();
```

## 7. Componentes Reutilizáveis

### 7.1 Props e Tipos

```typescript
// ✅ Correto: Definir interfaces para props
interface Props {
  open: boolean;
  onClose: VoidFunction;
  currentResource?: IResource;
}

// ❌ Incorreto: Usar any ou não tipar props
interface Props {
  [key: string]: any;
}
```

### 7.2 Composição de Componentes

```typescript
// ✅ Correto: Componentizar partes reutilizáveis
<ResourceForm
  open={createDialog.value}
  onClose={handleCloseEdit}
  currentResource={currentResource}
/>

// ❌ Incorreto: Código repetido ou inline
<Dialog open={open}>
  <DialogContent>
    {/* Código do formulário repetido */}
  </DialogContent>
</Dialog>
```

## 8. Performance

### 8.1 Memoização

```typescript
// ✅ Correto: Usar useCallback para handlers
const handleOpenEdit = useCallback(
  (resource: IResource) => {
    setCurrentResource(resource);
    createDialog.onTrue();
  },
  [createDialog]
);

// ❌ Incorreto: Funções inline sem memoização
const handleOpenEdit = (resource) => {
  setCurrentResource(resource);
  createDialog.onTrue();
};
```

### 8.2 Renderização Condicional

```typescript
// ✅ Correto: Renderizar componentes pesados condicionalmente
{isDialogOpen && <ResourceForm />}

// ❌ Incorreto: Sempre renderizar componentes pesados
<ResourceForm style={{ display: isDialogOpen ? 'block' : 'none' }} />
```

## 9. Internacionalização

### 9.1 Textos e Mensagens

```typescript
// ✅ Correto: Todos os textos em português
const messages = {
  success: 'Recurso criado com sucesso!',
  error: 'Erro ao criar recurso',
};

// ❌ Incorreto: Mistura de idiomas
const messages = {
  success: 'Resource created successfully!',
  error: 'Erro ao criar',
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Joao-AugustoPF) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
