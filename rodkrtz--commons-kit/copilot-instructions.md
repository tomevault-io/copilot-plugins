## commons-kit

> Biblioteca Kotlin com modelos de domínio compartilhados entre projetos.

# Commons Kit

Biblioteca Kotlin com modelos de domínio compartilhados entre projetos.
Versão atual: `0.0.1` (em desenvolvimento inicial).
Usa `kotlinx.serialization` — sem Spring, sem JPA.

## Conteúdo

```
domain/
  shared/
    contact/    → Email, PhoneNumber (abstrações)
    currency/   → Currency enum, conversão
    privacy/    → enums relacionados a privacidade
    region/     → Region/locale
    tax/        → TaxId, cálculos de imposto
  vehicle/      → modelos de domínio de veículos
```

## Regras de desenvolvimento

- **Explicit API**: visibilidade explícita em toda API pública.
- **kotlinx.serialization**: use `@Serializable` nos data classes compartilhados.
- **Sem dependências de framework**: pura Kotlin, zero Spring/Android.
- Testes com JUnit 5.

## Relação com outros projetos

- `commons-kit-bra` é a versão brasileira (CPF, CNPJ, PhoneBRA, BRL) — depende deste.
- Projetos backend importam via dependência publicada ou composite build.

## Build

```bash
./gradlew build     # compila + testa
./gradlew publish   # publica no GitHub Packages
```

---
> Source: [rodkrtz/commons-kit](https://github.com/rodkrtz/commons-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
