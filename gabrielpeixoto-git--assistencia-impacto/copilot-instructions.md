## codereviewmd

> Aplicar quando o usuário pedir revisão, análise ou feedback de código Python


# Checklist de Code Review Python

Ao revisar código, verifique nesta ordem:

<corretude>
- A lógica implementa corretamente o requisito?
- Há edge cases não tratados?
- O tratamento de erros é adequado?
</corretude>

<qualidade>
- Type hints presentes e corretos?
- Docstrings presentes nas funções públicas?
- Código legível sem precisar de comentários explicativos?
- Funções com responsabilidade única?
</qualidade>

<performance>
- Há loops desnecessários ou operações O(n²) evitáveis?
- Recursos sendo abertos sem serem fechados?
- Queries N+1 em código que acessa banco de dados?
</performance>

<seguranca>
- Inputs externos validados?
- Nenhum secret/credencial no código?
- SQL construído com parâmetros, não concatenação?
</seguranca>

Apresente os problemas agrupados por severidade: 🔴 Crítico | 🟡 Melhoria | 🟢 Sugestão

---
> Source: [gabrielpeixoto-git/assistencia-impacto](https://github.com/gabrielpeixoto-git/assistencia-impacto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
