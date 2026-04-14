## my-site

> - Siga as convenções do [Airbnb JavaScript Style Guide] adaptadas para TypeScript.

# Regras Gerais
- Siga as convenções do [Airbnb JavaScript Style Guide] adaptadas para TypeScript.
- Use componentes funcionais e hooks ao invés de componentes de classe.
- Utilize o ESLint e o Prettier para garantir a consistência do código.

# Estrutura do Projeto
- Organize os componentes em pastas conforme sua funcionalidade.
- Utilize a extensão `.tsx` para arquivos que contêm JSX e `.ts` para arquivos que não contêm.
- Nomeie arquivos e pastas utilizando o padrão `kebab-case`. A não ser que seja um componente, nesse caso utilize o padrão `PascalCase` para o nome do componente e sua pasta.
- Para as pastas de componentes, use a estrutura `ComponentName/ComponentName.tsx` para o componente principal e `ComponentName/styles.ts` para os estilos e `ComponentName/types.ts` para os tipos.

# Uso do TypeScript
- Utilize `interface` ao invés de `type` para definir tipos de objetos.
- Evite o uso do tipo `any`; prefira tipos mais específicos ou genéricos.

# Sintaxe e Formatação
- Utilize ponto e vírgula no final de cada instrução.
- Utilize aspas simples para strings, exceto quando a string contiver aspas simples.
- Mantenha a indentação de 2 espaços para melhor legibilidade.

# Estilização
- Utilize bibliotecas como `styled-components` para estilização de componentes.
- Prefira estilos baseados em temas para facilitar a manutenção e consistência.
- Evite o uso de estilos inline; centralize os estilos em arquivos dedicados.

# Boas Práticas
- Evite a repetição de código; utilize componentes reutilizáveis sempre que possível.
- Mantenha as funções e componentes pequenos e com responsabilidade única.
- Realize revisões de código periódicas para garantir a qualidade e consistência.

# Performance
- Utilize o `React.memo` para componentes que não precisam ser renderizados novamente.
- Evite operações custosas dentro do render; utilize hooks como `useMemo` e `useCallback` quando necessário.
- Implemente carregamento assíncrono para componentes pesados ou não essenciais.

# CURRENT FILE STRUCTURE
- sempre que fizer sentido, cheque a estrutura do projeto com o comando:
```
tree -L 6 -a -I "node_modules|dist|build|public|.git|.DS_Store"
```

# IMPORTANT
- Use 'export default function ComponentName() { return <div>Component</div>; }' para criar componentes.
- Trabalhe como um Senior Developer.
- Não pare de trabalhar até que o problema seja resolvido por completo.
- Comece sempre escrevendo no mínimo 3 paragrafos sobre o que você vai fazer no Chat para que você possa refletir sobre o que você vai fazer, nao pule etapas e nem tire conclusões precocemente. Após isso, comece a de fato resolver o problema.
- Nome de variáveis e funções em inglês, assim como os nomes dos componentes, componentes de pastas, comentários, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Guilherme-N-Sa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
