## esp-i2c-scanner

> Aplicado na manutenção do arquivo package.json na raiz do projeto, independente da linguagem, deve haver este arquivo para orquestração de scripts, em conjunto por exemplo com cmake.


## **Build**

* Na **raiz do workspace**, o arquivo `package.json` deve conter todas as **referências e scripts necessários ao processo de build** do sistema.

* Cada **camada ou módulo** do projeto deve possuir seu próprio processo de build, que pode ser executado de forma **independente** ou **recursiva** a partir do workspace principal.

* Ao realizar o build completo do sistema:

  1. Acesse cada pasta correspondente às camadas (por exemplo, `backend/`, `frontend/`, `mobile/`, etc.);
  2. Execute o comando de build definido para aquela camada;
  3. Garanta que todas as dependências estejam atualizadas antes da compilação.

* No arquivo `package.json` da raiz, crie um script padrão denominado:

  ```json
  "build:install"
  ```

  Esse script deve:

  * Executar **recursivamente o `build:install`** de cada camada do projeto;
  * Garantir a **instalação das dependências** e a **compilação sequencial** dos módulos;
  * Finalizar o processo exibindo **logs resumidos** de sucesso ou falha para cada módulo.

* Exemplo de estrutura sugerida para o `package.json` da raiz:

  ```json
  {
    "scripts": {
      "build:install": "npm run build:install -w backend && npm run build:install -w frontend && npm run build:install -w mobile"
    }
  }
  ```

<<<<<<< HEAD
* É recomendado que o **build global** valide a presença de arquivos `.env` e de dependências críticas antes de iniciar a compilação, emitindo avisos ou interrompendo o processo em caso de inconsistências.

=======
* É recomendado que o **build global** valide a presença de arquivos `.env` e de dependências críticas antes de iniciar a compilação, emitindo avisos ou interrompendo o processo em caso de inconsistências.
>>>>>>> main

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArvoreDosSaberes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
