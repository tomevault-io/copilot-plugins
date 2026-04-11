## tutoriais

> Este guia detalha como configurar um novo projeto Laravel 12 para utilizar o template AdminLTE 4 e Bootstrap 5, gerenciando as dependências frontend com `npm` e compilando os assets com Vite (o padrão do Laravel).

# Configurando Laravel 12 com AdminLTE 4 e Bootstrap 5 via npm/Vite

Este guia detalha como configurar um novo projeto Laravel 12 para utilizar o template AdminLTE 4 e Bootstrap 5, gerenciando as dependências frontend com `npm` e compilando os assets com Vite (o padrão do Laravel).

## Pré-requisitos

Antes de começar, certifique-se de ter o seguinte instalado em seu sistema:

* **PHP:** Versão compatível com Laravel 12 (consulte a [documentação oficial do Laravel](https://laravel.com/docs/12.x/releases#support-policy)).
* **Composer:** Instalado globalmente ([Download Composer](https://getcomposer.org/download/)).
* **Node.js e npm:** Instalados globalmente (versões LTS ou mais recentes são recomendadas) ([Download Node.js](https://nodejs.org/)).

## Passos Detalhados

### 1. Criar Novo Projeto Laravel 12

Abra seu terminal e use o Composer para criar um novo projeto Laravel:

```bash
composer create-project laravel/laravel seu-projeto-adminlte "12.*" --prefer-dist

```

-   Substitua `seu-projeto-adminlte` pelo nome desejado.

Navegue até o diretório do projeto:


```Bash
cd seu-projeto-adminlte

```

### 2. Instalar Dependências Frontend (npm)

Use o `npm` para instalar Bootstrap 5, Popper.js (dependência do Bootstrap), AdminLTE 4 e o compilador Sass:


```Bash
npm install bootstrap @popperjs/core admin-lte@^4 sass --save-dev

```

-   `bootstrap`: Framework CSS/JS.
-   `@popperjs/core`: Necessário para componentes JS do Bootstrap (dropdowns, tooltips, popovers).
-   `admin-lte@^4`: Template AdminLTE versão 4.
-   `sass`: Compilador SCSS que o Vite usará.
-   `--save-dev`: Instala como dependência de desenvolvimento.

### 3. Configurar Vite (`vite.config.js`)

Abra o arquivo `vite.config.js` na raiz do projeto e ajuste a configuração do plugin `laravel` para incluir seus arquivos SCSS e JS principais. Adicionar aliases é recomendado para facilitar os imports.

```javascript

// vite.config.js

import { defineConfig } from 'vite';

import laravel from 'laravel-vite-plugin';

export default defineConfig({

plugins:1 [

laravel({

// Defina aqui os pontos de entrada para compilação

input: [

'resources/scss/app.scss', // Arquivo SCSS principal

'resources/js/app.js', // Arquivo JS principal

],

refresh: true,

}),

],

// (Recomendado) Aliases para imports mais limpos

resolve: {

alias: {

'~bootstrap': 'node_modules/bootstrap',

'~admin-lte': 'node_modules/admin-lte',

}

},

});

```

### 4\. Criar e Configurar Arquivos SCSS

1.  **Renomeie/Crie a pasta:** Se existir `resources/css`, renomeie-a para `resources/scss`. Caso contrário, crie `resources/scss`.
2.  **Renomeie/Crie o arquivo:** Mova/renomeie `resources/css/app.css` para `resources/scss/app.scss` ou crie este último.
3.  **Edite `resources/scss/app.scss`:** Importe o Bootstrap e o AdminLTE.

<!-- end list -->

```scss
// resources/scss/app.scss

// Opcional: Importe partes do Bootstrap se precisar sobrescrever variáveis SCSS
// @import "~bootstrap/scss/functions";
// @import "~bootstrap/scss/variables";
// Defina suas variáveis personalizadas AQUI
// $primary: #meu-azul;

// Importe o SCSS completo do Bootstrap
@import "~bootstrap/scss/bootstrap";

// Importe o SCSS do AdminLTE (use o alias ou o caminho completo)
@import "~admin-lte/build/scss/adminlte";

// Adicione seus estilos personalizados abaixo
.custom-style {
  padding: 1rem;
}

// Se não usou aliases, use caminhos relativos:
// @import "../node_modules/bootstrap/scss/bootstrap";
// @import "../node_modules/admin-lte/build/scss/adminlte";

```

### 5. Configurar Arquivo JavaScript (`resources/js/app.js`)

Edite `resources/js/app.js` para importar o JavaScript necessário:


```JavaScript
// resources/js/app.js

// Importa Popper.js (geralmente necessário para Bootstrap JS)
import '@popperjs/core';

// Importa TODO o JavaScript do Bootstrap
import ** as bootstrap from 'bootstrap';
// OU importe apenas componentes específicos:
// import { Dropdown, Modal } from 'bootstrap';

// Importa o JavaScript do AdminLTE (inicializa o tema)
import 'admin-lte';

// Seu código JS personalizado
console.log('Bootstrap e AdminLTE carregados!');

// Exemplo: Habilitar tooltips do Bootstrap (se usados no HTML)
// const tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'));
// tooltipTriggerList.map(function (tooltipTriggerEl) {
//   return new bootstrap.Tooltip(tooltipTriggerEl);
// });

```

### 6. Incluir Assets no Layout Blade

No seu arquivo de layout principal (ex: `resources/views/layouts/app.blade.php` ou um layout de admin específico), use a diretiva `@vite` para incluir os assets compilados.


```HTML
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{ config('app.name', 'Laravel') }}</title>

    <link rel="preconnect" href="[https://fonts.bunny.net](https://fonts.bunny.net)">
    <link href="[https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap](https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap)" rel="stylesheet" />

    {{-- Inclui os assets processados pelo Vite --}}
    @vite(['resources/scss/app.scss', 'resources/js/app.js'])

</head>
{{-- Adicione as classes do AdminLTE ao body e use a estrutura HTML correta --}}
<body class="hold-transition sidebar-mini layout-fixed">
<div class="wrapper">

    {{-- Navbar (Exemplo: @include('layouts.partials.navbar')) --}}
    {{-- Main Sidebar Container (Exemplo: @include('layouts.partials.sidebar')) --}}

    <div class="content-wrapper">
        {{-- Conteúdo principal aqui (@yield('content')) --}}
        @yield('content')
    </div>
    {{-- Footer (Exemplo: @include('layouts.partials.footer')) --}}

</div>
@stack('scripts') {{-- Para scripts específicos da página --}}
</body>
</html>

```

**Importante:** Você precisará estruturar seu HTML dentro do `body` usando as classes e elementos do AdminLTE 4 (como `.wrapper`, `.main-sidebar`, `.content-wrapper`, etc.). Consulte a [Documentação de Layout do AdminLTE 4](https://www.google.com/search?q=https://adminlte.io/docs/4/html/layout.html) para exemplos.

### 7. Desenvolvimento e Build

-   Para Desenvolvimento:
    
    Execute o servidor de desenvolvimento Vite (compila e observa mudanças com HMR):
    
    
    
    ```Bash
    npm run dev
    
    ```
    
    E em outro terminal, o servidor PHP do Laravel:
    
    
    
    ```Bash
    php artisan serve
    
    ```
    
    Acesse seu projeto via `http://localhost:8000` (ou a porta definida).
    
-   Para Produção:
    
    Quando for fazer o deploy, pare o npm run dev (Ctrl+C) e execute o build:
    
    
    
    ```Bash
    npm run build
    
    ```
    
    Isso criará os assets otimizados e minificados na pasta `public/build`. A diretiva `@vite` no Blade automaticamente usará esses arquivos em ambiente de produção.
    

----------

Com esses passos, seu projeto Laravel 12 estará configurado para usar AdminLTE 4 e Bootstrap 5, com os assets gerenciados eficientemente pelo Vite e npm.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunoaffonso)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brunoaffonso)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
