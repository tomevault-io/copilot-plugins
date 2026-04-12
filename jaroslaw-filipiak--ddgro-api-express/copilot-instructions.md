## ddgro-api-express

> Email and PDF template patterns using Handlebars


# Template Patterns

## Template Directory Structure

Based on [templates/](mdc:templates/) directory:

```
templates/
├── emails/                 # Email templates
│   ├── order.handlebars   # Order confirmation email
│   └── order_ext.handlebars # Extended order template
└── pdf/                   # PDF templates
    ├── template.html      # Main PDF template
    └── template2.html     # Alternative PDF template
```

## Email Template Patterns

Based on [templates/emails/](mdc:templates/emails/):

### Basic Email Template Structure

```handlebars
<!DOCTYPE html>
<html lang="{{lang}}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{subject}}</title>
    <style>
        /* Inline styles for email compatibility */
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            color: #333;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        .header {
            background-color: #f8f9fa;
            padding: 20px;
            text-align: center;
        }
        .content {
            padding: 20px;
        }
        .footer {
            background-color: #f8f9fa;
            padding: 15px;
            text-align: center;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>{{company.name}}</h1>
        </div>

        <div class="content">
            {{> content}}
        </div>

        <div class="footer">
            {{> footer}}
        </div>
    </div>
</body>
</html>
```

### Order Confirmation Template

```handlebars
<!-- Order Header -->
<h2>Potwierdzenie zamówienia #{{order.id}}</h2>
<p>Dzień dobry {{customer.name_surname}},</p>
<p>Dziękujemy za przesłanie kalkulacji. Poniżej znajduje się podsumowanie:</p>

<!-- Customer Information -->
<div class='customer-info'>
  <h3>Dane klienta:</h3>
  <ul>
    <li><strong>Imię i nazwisko:</strong> {{customer.name_surname}}</li>
    <li><strong>Email:</strong> {{customer.email}}</li>
    <li><strong>Telefon:</strong> {{customer.phone}}</li>
    {{#if customer.profession}}
      <li><strong>Branża:</strong> {{customer.profession}}</li>
    {{/if}}
  </ul>
</div>

<!-- Project Details -->
<div class='project-details'>
  <h3>Szczegóły projektu:</h3>
  <ul>
    <li><strong>Typ tarasu:</strong> {{getTypeName project.type}}</li>
    <li><strong>Powierzchnia całkowita:</strong> {{project.total_area}} m²</li>
    <li><strong>Liczba tarasów:</strong> {{project.count}}</li>
    <li><strong>System główny:</strong> {{project.main_system}}</li>
    <li><strong>Typ podparcia:</strong>
      {{getSupportTypeDescription project.support_type}}</li>
  </ul>
</div>

<!-- Product Summary -->
{{#if products}}
  <div class='products'>
    <h3>Produkty:</h3>
    <table style='width: 100%; border-collapse: collapse;'>
      <thead>
        <tr style='background-color: #f8f9fa;'>
          <th style='border: 1px solid #ddd; padding: 8px;'>Produkt</th>
          <th style='border: 1px solid #ddd; padding: 8px;'>Ilość</th>
          <th style='border: 1px solid #ddd; padding: 8px;'>Jednostka</th>
        </tr>
      </thead>
      <tbody>
        {{#each products}}
          <tr>
            <td style='border: 1px solid #ddd; padding: 8px;'>{{this.name}}</td>
            <td
              style='border: 1px solid #ddd; padding: 8px;'
            >{{this.quantity}}</td>
            <td style='border: 1px solid #ddd; padding: 8px;'>{{this.unit}}</td>
          </tr>
        {{/each}}
      </tbody>
    </table>
  </div>
{{/if}}

<!-- Accessories -->
{{#if accessories}}
  <div class='accessories'>
    <h3>Akcesoria:</h3>
    <table style='width: 100%; border-collapse: collapse;'>
      <thead>
        <tr style='background-color: #f8f9fa;'>
          <th style='border: 1px solid #ddd; padding: 8px;'>Akcesorium</th>
          <th style='border: 1px solid #ddd; padding: 8px;'>Ilość</th>
        </tr>
      </thead>
      <tbody>
        {{#each accessories}}
          <tr>
            <td style='border: 1px solid #ddd; padding: 8px;'>{{this.name}}</td>
            <td
              style='border: 1px solid #ddd; padding: 8px;'
            >{{this.quantity}}</td>
          </tr>
        {{/each}}
      </tbody>
    </table>
  </div>
{{/if}}

<!-- Calculations -->
<div class='calculations'>
  <h3>Kalkulacje:</h3>
  <ul>
    <li><strong>Liczba płyt/desek:</strong> {{calculations.slabs_count}}</li>
    <li><strong>Liczba wsporników:</strong> {{calculations.supports_count}}</li>
  </ul>
</div>

<!-- Call to Action -->
<div
  class='cta'
  style='margin: 30px 0; padding: 20px; background-color: #e9f7ff; border-radius: 5px;'
>
  <p><strong>Następne kroki:</strong></p>
  <p>Nasz zespół przeanalizuje Państwa zapotrzebowanie i skontaktuje się w ciągu
    24 godzin z ofertą cenową.</p>
  <p>W przypadku pytań prosimy o kontakt:</p>
  <ul>
    <li>Email: biuro@ddgro.eu</li>
    <li>Telefon: +48 123 456 789</li>
  </ul>
</div>
```

## PDF Template Patterns

Based on [templates/pdf/](mdc:templates/pdf/):

### PDF Template Structure

```html
<!DOCTYPE html>
<html lang="{{lang}}">
  <head>
    <meta charset="UTF-8" />
    <title>Kalkulacja tarasu - {{customer.name_surname}}</title>
    <style>
      @page {
        size: A4;
        margin: 2cm;
      }

      body {
        font-family: 'Arial', sans-serif;
        font-size: 12px;
        line-height: 1.4;
        color: #333;
      }

      .header {
        text-align: center;
        margin-bottom: 30px;
        border-bottom: 2px solid #0066cc;
        padding-bottom: 20px;
      }

      .company-logo {
        max-width: 200px;
        height: auto;
      }

      .page-break {
        page-break-before: always;
      }

      table {
        width: 100%;
        border-collapse: collapse;
        margin: 20px 0;
      }

      th,
      td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
      }

      th {
        background-color: #f2f2f2;
        font-weight: bold;
      }

      .summary-box {
        background-color: #f9f9f9;
        padding: 15px;
        margin: 20px 0;
        border-left: 4px solid #0066cc;
      }

      .calculation-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 20px;
        margin: 20px 0;
      }

      .text-center {
        text-align: center;
      }
      .text-right {
        text-align: right;
      }
      .font-bold {
        font-weight: bold;
      }
    </style>
  </head>
  <body>
    <!-- Header -->
    <div class="header">
      <img
        src="data:image/png;base64,{{company.logo_base64}}"
        alt="{{company.name}}"
        class="company-logo"
      />
      <h1>Kalkulacja tarasu</h1>
      <p>Data: {{formatDate created_at}}</p>
    </div>

    <!-- Customer Information -->
    <section>
      <h2>Dane klienta</h2>
      <table>
        <tr>
          <td class="font-bold">Imię i nazwisko:</td>
          <td>{{customer.name_surname}}</td>
        </tr>
        <tr>
          <td class="font-bold">Email:</td>
          <td>{{customer.email}}</td>
        </tr>
        <tr>
          <td class="font-bold">Telefon:</td>
          <td>{{customer.phone}}</td>
        </tr>
        {{#if customer.profession}}
        <tr>
          <td class="font-bold">Branża:</td>
          <td>{{customer.profession}}</td>
        </tr>
        {{/if}}
      </table>
    </section>

    <!-- Project Specifications -->
    <section>
      <h2>Specyfikacja projektu</h2>
      <div class="calculation-grid">
        <div>
          <h3>Podstawowe parametry</h3>
          <ul>
            <li><strong>Typ tarasu:</strong> {{getTypeName project.type}}</li>
            <li><strong>Powierzchnia:</strong> {{project.total_area}} m²</li>
            <li><strong>Liczba tarasów:</strong> {{project.count}}</li>
            <li><strong>System:</strong> {{project.main_system}}</li>
          </ul>
        </div>
        <div>
          <h3>Wymiary</h3>
          <ul>
            {{#if project.slab_width}}
            <li><strong>Szerokość płyty:</strong> {{project.slab_width}} mm</li>
            {{/if}} {{#if project.slab_height}}
            <li><strong>Długość płyty:</strong> {{project.slab_height}} mm</li>
            {{/if}} {{#if project.slab_thickness}}
            <li>
              <strong>Grubość płyty:</strong> {{project.slab_thickness}} mm
            </li>
            {{/if}}
          </ul>
        </div>
      </div>
    </section>

    <!-- Products Table -->
    {{#if products}}
    <section>
      <h2>Lista produktów</h2>
      <table>
        <thead>
          <tr>
            <th>Lp.</th>
            <th>Nazwa produktu</th>
            <th>Kod</th>
            <th>Ilość</th>
            <th>Jednostka</th>
            <th>Opis</th>
          </tr>
        </thead>
        <tbody>
          {{#each products}}
          <tr>
            <td class="text-center">{{@index}}</td>
            <td>{{this.name}}</td>
            <td>{{this.code}}</td>
            <td class="text-right">{{this.quantity}}</td>
            <td class="text-center">{{this.unit}}</td>
            <td>{{this.description}}</td>
          </tr>
          {{/each}}
        </tbody>
      </table>
    </section>
    {{/if}}

    <!-- Summary -->
    <div class="summary-box">
      <h3>Podsumowanie kalkulacji</h3>
      <div class="calculation-grid">
        <div>
          <p>
            <strong>Liczba płyt/desek:</strong> {{calculations.slabs_count}}
          </p>
          <p>
            <strong>Liczba wsporników:</strong> {{calculations.supports_count}}
          </p>
        </div>
        <div>
          <p><strong>Data utworzenia:</strong> {{formatDate created_at}}</p>
          <p><strong>Język:</strong> {{lang}}</p>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <footer
      style="position: fixed; bottom: 0; width: 100%; text-align: center; font-size: 10px; color: #666;"
    >
      <p>{{company.name}} | {{company.address}} | {{company.contact}}</p>
    </footer>
  </body>
</html>
```

## Handlebars Helper Registration

### Template Helper Setup

```javascript
const handlebars = require('handlebars');

// Date formatting helper
handlebars.registerHelper('formatDate', function (date, format = 'DD.MM.YYYY') {
  if (!date) return '';

  const d = new Date(date);
  return d.toLocaleDateString('pl-PL', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
  });
});

// Currency formatting helper
handlebars.registerHelper(
  'formatCurrency',
  function (amount, currency = 'PLN') {
    if (!amount) return '0,00';

    return new Intl.NumberFormat('pl-PL', {
      style: 'currency',
      currency: currency,
    }).format(amount);
  },
);

// Conditional helpers
handlebars.registerHelper('ifEquals', function (arg1, arg2, options) {
  return arg1 == arg2 ? options.fn(this) : options.inverse(this);
});

handlebars.registerHelper('ifGreater', function (arg1, arg2, options) {
  return arg1 > arg2 ? options.fn(this) : options.inverse(this);
});

// Math helpers
handlebars.registerHelper('add', function (a, b) {
  return parseFloat(a) + parseFloat(b);
});

handlebars.registerHelper('multiply', function (a, b) {
  return parseFloat(a) * parseFloat(b);
});

// Array helpers
handlebars.registerHelper('length', function (array) {
  return array ? array.length : 0;
});

handlebars.registerHelper('join', function (array, separator = ', ') {
  return array ? array.join(separator) : '';
});

// Index helper for loops
handlebars.registerHelper('index', function (value) {
  return parseInt(value) + 1;
});
```

## Template Compilation and Usage

### Email Template Service

```javascript
const fs = require('fs');
const path = require('path');
const handlebars = require('handlebars');

class TemplateService {
  constructor() {
    this.templatesPath = path.resolve('./templates');
    this.compiledTemplates = new Map();
  }

  async compileTemplate(templatePath) {
    if (this.compiledTemplates.has(templatePath)) {
      return this.compiledTemplates.get(templatePath);
    }

    const fullPath = path.join(this.templatesPath, templatePath);
    const templateContent = await fs.promises.readFile(fullPath, 'utf8');
    const compiled = handlebars.compile(templateContent);

    this.compiledTemplates.set(templatePath, compiled);
    return compiled;
  }

  async renderEmail(templateName, context) {
    const templatePath = `emails/${templateName}.handlebars`;
    const template = await this.compileTemplate(templatePath);
    return template(context);
  }

  async renderPDF(templateName, context) {
    const templatePath = `pdf/${templateName}.html`;
    const template = await this.compileTemplate(templatePath);
    return template(context);
  }

  // Clear compiled templates cache
  clearCache() {
    this.compiledTemplates.clear();
  }
}

module.exports = new TemplateService();
```

### Template Context Preparation

```javascript
const prepareEmailContext = (application, additionalData = {}) => {
  return {
    // Customer data
    customer: {
      name_surname: application.name_surname,
      email: application.email,
      phone: application.phone,
      profession: application.proffestion,
    },

    // Project data
    project: {
      type: application.type,
      total_area: application.total_area,
      count: application.count,
      main_system: application.main_system,
      support_type: application.support_type,
      slab_width: application.slab_width,
      slab_height: application.slab_height,
      slab_thickness: application.slab_thickness,
    },

    // Calculations
    calculations: {
      slabs_count: application.slabs_count,
      supports_count: application.supports_count,
    },

    // Products and accessories
    products: application.products || [],
    accessories: application.accessories || [],

    // Company information
    company: {
      name: process.env.COMPANY_NAME || 'DDGRO',
      address: process.env.COMPANY_ADDRESS,
      contact: process.env.COMPANY_CONTACT,
      logo_base64: additionalData.logo_base64,
    },

    // Meta information
    created_at: application.created_at || new Date(),
    lang: application.lang || 'pl',

    // Additional context
    ...additionalData,
  };
};
```

## Template Partials

### Reusable Template Components

```handlebars
<!-- Partial: customer-info.hbs -->
<div class="customer-section">
    <h3>Dane klienta</h3>
    <div class="customer-details">
        <p><strong>Imię i nazwisko:</strong> {{customer.name_surname}}</p>
        <p><strong>Email:</strong> {{customer.email}}</p>
        <p><strong>Telefon:</strong> {{customer.phone}}</p>
        {{#if customer.profession}}
        <p><strong>Branża:</strong> {{customer.profession}}</p>
        {{/if}}
    </div>
</div>

<!-- Partial: product-table.hbs -->
<table class="product-table">
    <thead>
        <tr>
            <th>Lp.</th>
            <th>Nazwa produktu</th>
            <th>Ilość</th>
            <th>Jednostka</th>
        </tr>
    </thead>
    <tbody>
        {{#each products}}
        <tr>
            <td>{{index @index}}</td>
            <td>{{this.name}}</td>
            <td>{{this.quantity}}</td>
            <td>{{this.unit}}</td>
        </tr>
        {{/each}}
    </tbody>
</table>

<!-- Register and use partials -->
{{> customer-info}}
{{> product-table}}
```

### Partial Registration

```javascript
const registerPartials = async () => {
  const partialsDir = path.join(__dirname, 'templates/partials');
  const partialFiles = await fs.promises.readdir(partialsDir);

  for (const file of partialFiles) {
    if (file.endsWith('.hbs')) {
      const partialName = path.basename(file, '.hbs');
      const partialContent = await fs.promises.readFile(
        path.join(partialsDir, file),
        'utf8',
      );
      handlebars.registerPartial(partialName, partialContent);
    }
  }
};
```

## Template Testing

### Template Rendering Tests

```javascript
const TemplateService = require('../services/TemplateService');

describe('Template Service', () => {
  const mockContext = {
    customer: {
      name_surname: 'Jan Kowalski',
      email: 'jan@example.com',
      phone: '+48123456789',
    },
    project: {
      type: 'slab',
      total_area: 50,
      count: 1,
    },
  };

  it('should render email template', async () => {
    const html = await TemplateService.renderEmail('order', mockContext);

    expect(html).toContain('Jan Kowalski');
    expect(html).toContain('jan@example.com');
    expect(html).toContain('50');
  });

  it('should render PDF template', async () => {
    const html = await TemplateService.renderPDF('template', mockContext);

    expect(html).toContain('<!DOCTYPE html>');
    expect(html).toContain('Jan Kowalski');
  });
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaroslaw-filipiak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jaroslaw-filipiak)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
