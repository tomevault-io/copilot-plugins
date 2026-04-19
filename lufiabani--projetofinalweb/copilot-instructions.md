## projetofinalweb

> - Siga APENAS os padrões ensinados pelo professor Matheus Cataneo

# .cursorrules — Desenvolvimento de Sistemas Web
# Professor: Matheus Cataneo
# IMPORTANTE: Siga EXATAMENTE os padrões ensinados pelo professor.
# NÃO use padrões alternativos, mesmo que "melhores". Simplicidade e fidelidade ao modelo.

## REGRA GERAL
- Siga APENAS os padrões ensinados pelo professor Matheus Cataneo
- NÃO adicione técnicas, bibliotecas ou padrões não ensinados
- NÃO use Repository Pattern, DTOs, AutoMapper, MediatR, FluentValidation
- NÃO use TypeScript no frontend — apenas JavaScript puro (.jsx, .js)
- Mantenha o código simples, comentado e fiel ao modelo do professor
- Comentários devem ser em português, explicando o "por quê" como o professor faz

---

## BACKEND — .NET 8 Web API + PostgreSQL

### Stack obrigatória
- .NET 8 Web API
- Entity Framework Core 8.0.0
- Npgsql.EntityFrameworkCore.PostgreSQL 8.0.0
- Microsoft.EntityFrameworkCore.Design 8.0.0
- Swashbuckle.AspNetCore.SwaggerGen 8.0.0
- Swashbuckle.AspNetCore.SwaggerUI 8.0.0

### Criação do projeto
```
dotnet new webapi -n NomeProjeto.Api --no-openapi
dotnet add package Microsoft.EntityFrameworkCore --version 8.0.0
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0.0
dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.0
```
Instalar Swagger manualmente via csproj (SwaggerGen + SwaggerUI versão 8.0.0).
Deletar WeatherForecast.cs e WeatherForecastController.cs ao criar o projeto.

### Estrutura de pastas obrigatória
```
NomeProjeto.Api/
├── Controllers/
├── Data/
│   └── AppDbContext.cs
├── Models/
├── Migrations/       ← gerada automaticamente
├── appsettings.json
├── Program.cs
└── NomeProjeto.Api.csproj
```

### Padrão de Models
- Chave primária: public int Id { get; set; }
- Campo obrigatório: public required string Nome { get; set; }
- Campo opcional: public string? Descricao { get; set; }
- Valor monetário: public decimal Preco { get; set; }
- Data de criação: public DateTime DataCriacao { get; set; } = DateTime.UtcNow;
- SEMPRE usar DateTime.UtcNow, NUNCA DateTime.Now
- Chave estrangeira obrigatória: public int CategoriaId { get; set; }
- Chave estrangeira opcional: public int? CategoriaId { get; set; }
- Propriedade de navegação (lado "muitos"): public Categoria? Categoria { get; set; }
- Propriedade de navegação (lado "um", 1-para-N): public ICollection<Produto> Produtos { get; set; } = new List<Produto>();
- Propriedade de navegação (1-para-1): public DetalheProduto? DetalheProduto { get; set; }
- NÃO usar Data Annotations ([Required], [MaxLength], etc.) — usar Fluent API no DbContext

### Padrão do AppDbContext
```csharp
// Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using NomeProjeto.Api.Models;

namespace NomeProjeto.Api.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<Produto> Produtos { get; set; }
    public DbSet<Categoria> Categorias { get; set; }
    public DbSet<DetalheProduto> DetalhesProduto { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Relacionamento 1-para-N: Produto → Categoria
        modelBuilder.Entity<Produto>()
            .HasOne(p => p.Categoria)
            .WithMany(c => c.Produtos)
            .HasForeignKey(p => p.CategoriaId)
            .OnDelete(DeleteBehavior.Restrict);

        // Relacionamento 1-para-1: DetalheProduto → Produto
        modelBuilder.Entity<DetalheProduto>()
            .HasOne(d => d.Produto)
            .WithOne(p => p.DetalheProduto)
            .HasForeignKey<DetalheProduto>(d => d.ProdutoId)
            .OnDelete(DeleteBehavior.Cascade);

        // OBRIGATÓRIO para 1-para-1: índice UNIQUE na FK
        modelBuilder.Entity<DetalheProduto>()
            .HasIndex(d => d.ProdutoId)
            .IsUnique();
    }
}
```

### Regras do DbContext
- Relacionamento 1-para-N: usar HasOne/WithMany + OnDelete(DeleteBehavior.Restrict)
- Relacionamento 1-para-1: usar HasOne/WithOne + HasForeignKey<T> + OnDelete(DeleteBehavior.Cascade) + .HasIndex().IsUnique()
- Restrict no 1-para-N: protege contra deleção acidental de registros pai com filhos
- Cascade no 1-para-1: detalhe sem entidade principal não faz sentido, deleta junto
- NUNCA usar EntityState.Modified no PUT

### Padrão do Program.cs
```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using NomeProjeto.Api.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Evita erro de ciclo de serialização em relacionamentos
        options.JsonSerializerOptions.ReferenceHandler =
            System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
    });

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddCors(options =>
{
    options.AddPolicy("PermitirTudo", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseCors("PermitirTudo");
app.MapControllers();
app.Run();
```

### Padrão de Controllers — CRUD básico
```csharp
// Controllers/ProdutosController.cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using NomeProjeto.Api.Data;
using NomeProjeto.Api.Models;

namespace NomeProjeto.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    private readonly AppDbContext _context;

    public ProdutosController(AppDbContext context)
    {
        _context = context;
    }

    // GET /api/produtos
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Produto>>> GetProdutos()
    {
        var produtos = await _context.Produtos
            .Include(p => p.Categoria)
            .ToListAsync();
        return Ok(produtos);
    }

    // GET /api/produtos/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Produto>> GetProduto(int id)
    {
        var produto = await _context.Produtos
            .Include(p => p.Categoria)
            .Include(p => p.DetalheProduto)
            .FirstOrDefaultAsync(p => p.Id == id);

        if (produto == null)
            return NotFound(new { mensagem = $"Produto com ID {id} não encontrado." });

        return Ok(produto);
    }

    // POST /api/produtos
    [HttpPost]
    public async Task<ActionResult<Produto>> PostProduto(Produto produto)
    {
        produto.DataCriacao = DateTime.UtcNow;
        _context.Produtos.Add(produto);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetProduto), new { id = produto.Id }, produto);
    }

    // PUT /api/produtos/5
    [HttpPut("{id}")]
    public async Task<IActionResult> PutProduto(int id, Produto produto)
    {
        if (id != produto.Id)
            return BadRequest(new { mensagem = "O ID da URL não corresponde ao ID do produto." });

        var produtoExistente = await _context.Produtos.FindAsync(id);

        if (produtoExistente == null)
            return NotFound(new { mensagem = $"Produto com ID {id} não encontrado." });

        // Atualizar campo a campo — NUNCA usar EntityState.Modified
        produtoExistente.Nome = produto.Nome;
        produtoExistente.Descricao = produto.Descricao;
        produtoExistente.Preco = produto.Preco;
        produtoExistente.Quantidade = produto.Quantidade;
        produtoExistente.CategoriaId = produto.CategoriaId;

        await _context.SaveChangesAsync();
        return NoContent();
    }

    // DELETE /api/produtos/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduto(int id)
    {
        var produto = await _context.Produtos.FindAsync(id);

        if (produto == null)
            return NotFound(new { mensagem = $"Produto com ID {id} não encontrado." });

        _context.Produtos.Remove(produto);
        await _context.SaveChangesAsync();
        return NoContent();
    }
}
```

### Regras dos Controllers
- GET lista: retorna Ok(lista) — HTTP 200
- GET por ID: retorna Ok(objeto) ou NotFound(new { mensagem = "..." }) — HTTP 200 ou 404
- POST: retorna CreatedAtAction(nameof(GetX), new { id = x.Id }, x) — HTTP 201
- PUT: verifica id != objeto.Id → 400; busca existente → 404; atualiza campo a campo → 204
- DELETE simples: busca → 404 se não existe; remove → 204
- DELETE com Restrict (1-para-N pai): usar try/catch (DbUpdateException) → retorna 400 com mensagem amigável
- Mensagens de erro SEMPRE em português no formato: new { mensagem = "..." }
- NUNCA usar FindAsync com .Include() — usar FirstOrDefaultAsync com .Include()
- FindAsync só para busca simples por PK sem relacionamentos (PUT, DELETE)

### Padrão do Controller com Restrict (ex: CategoriasController DELETE)
```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteCategoria(int id)
{
    var categoria = await _context.Categorias.FindAsync(id);

    if (categoria == null)
        return NotFound(new { mensagem = $"Categoria com ID {id} não encontrada." });

    try
    {
        _context.Categorias.Remove(categoria);
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateException)
    {
        return BadRequest(new
        {
            mensagem = "Não é possível deletar esta categoria porque ela possui produtos associados. " +
                "Remova ou reatribua os produtos antes de deletar a categoria."
        });
    }

    return NoContent();
}
```

### Padrão do Controller 1-para-1 (DetalhesProdutoController)
- GET usa rota semântica: [HttpGet("produto/{produtoId}")] — busca pelo ID do produto, não do detalhe
- POST valida: (1) produto existe? → 400; (2) produto já tem detalhe? → 409 Conflict
- PUT e DELETE usam o ID do detalhe (PK), não o ProdutoId
- NÃO atualizar ProdutoId no PUT — um detalhe não pode "migrar" para outro produto

```csharp
// POST — validações obrigatórias para 1-para-1
[HttpPost]
public async Task<ActionResult<DetalheProduto>> PostDetalhe(DetalheProduto detalhe)
{
    var produto = await _context.Produtos.FindAsync(detalhe.ProdutoId);
    if (produto == null)
        return BadRequest(new { mensagem = $"Produto com ID {detalhe.ProdutoId} não encontrado." });

    var detalheExistente = await _context.DetalhesProduto
        .FirstOrDefaultAsync(d => d.ProdutoId == detalhe.ProdutoId);

    if (detalheExistente != null)
        return Conflict(new
        {
            mensagem = $"O produto de ID {detalhe.ProdutoId} já possui um detalhe cadastrado. " +
                "Use PUT para atualizar o detalhe existente."
        });

    _context.DetalhesProduto.Add(detalhe);
    await _context.SaveChangesAsync();

    return CreatedAtAction(
        nameof(GetDetalhePorProduto),
        new { produtoId = detalhe.ProdutoId },
        detalhe
    );
}
```

### Migrations
- Nomes SEMPRE descritivos: CriacaoInicial, AdicionarCategoriaERelacionamento, AdicionarDetalheProduto
- NUNCA usar nomes genéricos como Migration1, Update, Fix
- Comandos:
```
dotnet ef migrations add NomeDescritivo
dotnet ef database update
```

### appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=nomedb;Username=postgres;Password=SUA_SENHA"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

---

## FRONTEND — React + Vite (JavaScript puro, SEM TypeScript)

### Stack obrigatória
- React + Vite (template react, NÃO react-ts)
- Axios (chamadas HTTP)
- Tailwind CSS v4 (tailwindcss + @tailwindcss/postcss + postcss)
- Lucide React (ícones)
- React Router DOM (navegação)
- React Toastify (notificações)

### Instalação
```
npm create vite@latest meucrud-frontend -- --template react
cd meucrud-frontend
npm install
npm install axios
npm install tailwindcss @tailwindcss/postcss postcss lucide-react react-router-dom
npm install react-toastify
```

### Configuração do PostCSS (postcss.config.js na raiz)
```javascript
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

### Configuração do CSS principal (src/index.css)
```css
@import "tailwindcss";

@keyframes fade-in {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}

@keyframes slide-in {
  from { opacity: 0; transform: translateX(100%); }
  to { opacity: 1; transform: translateX(0); }
}

.animate-fade-in { animation: fade-in 0.2s ease-out; }
.animate-slide-in { animation: slide-in 0.3s ease-out; }
```

### Estrutura de pastas obrigatória
```
meucrud-frontend/
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Layout.jsx
│   │   │   └── Sidebar.jsx
│   │   ├── produtos/
│   │   │   ├── ProdutosPage.jsx
│   │   │   ├── ProdutoTable.jsx
│   │   │   ├── ProdutoFormModal.jsx
│   │   │   ├── ProdutoDeleteDialog.jsx
│   │   │   └── DetalheModal.jsx        ← para 1-para-1
│   │   ├── categorias/
│   │   │   ├── CategoriasPage.jsx
│   │   │   ├── CategoriaTable.jsx
│   │   │   └── CategoriaFormModal.jsx
│   │   └── ui/
│   │       ├── Modal.jsx
│   │       └── ConfirmDialog.jsx
│   ├── services/
│   │   ├── produtoService.js
│   │   ├── categoriaService.js
│   │   └── detalheProdutoService.js
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── postcss.config.js
└── package.json
```

### Padrão de Services (Axios)
```javascript
// src/services/categoriaService.js
import axios from 'axios';

// IMPORTANTE: Substitua a porta pela porta da SUA API
const API_URL = 'http://localhost:5217/api/categorias';

const api = axios.create({
  baseURL: API_URL,
  headers: { 'Content-Type': 'application/json' },
});

export const getCategorias = async () => {
  const response = await api.get('/');
  return response.data;
};

export const getCategoria = async (id) => {
  const response = await api.get(`/${id}`);
  return response.data;
};

export const criarCategoria = async (categoria) => {
  const response = await api.post('/', categoria);
  return response.data;
};

export const atualizarCategoria = async (id, categoria) => {
  const response = await api.put(`/${id}`, categoria);
  return response.data;
};

export const deletarCategoria = async (id) => {
  const response = await api.delete(`/${id}`);
  return response.data;
};
```

### Padrão do Service 1-para-1 (detalheProdutoService.js)
```javascript
// src/services/detalheProdutoService.js
import axios from 'axios';

const API_URL = 'http://localhost:5217/api/detalhesproduto';

const api = axios.create({
  baseURL: API_URL,
  headers: { 'Content-Type': 'application/json' },
});

// GET usa produtoId (não o id do detalhe)
export const getDetalhePorProduto = async (produtoId) => {
  const response = await api.get(`/produto/${produtoId}`);
  return response.data;
};

export const criarDetalhe = async (detalhe) => {
  const response = await api.post('/', detalhe);
  return response.data;
};

// PUT e DELETE usam o id do DETALHE
export const atualizarDetalhe = async (id, detalhe) => {
  const response = await api.put(`/${id}`, detalhe);
  return response.data;
};

export const deletarDetalhe = async (id) => {
  const response = await api.delete(`/${id}`);
  return response.data;
};
```

### Padrão de carregamento de dados com Promise.all
```javascript
// Carregar múltiplas entidades em paralelo — SEMPRE usar Promise.all
const carregarDados = async () => {
  try {
    const [produtos, categorias] = await Promise.all([
      getProdutos(),
      getCategorias(),
    ]);
    setProdutos(produtos);
    setCategorias(categorias);
  } catch (error) {
    toast.error('Erro ao carregar os dados.');
    console.error('Erro ao carregar:', error);
  }
};

useEffect(() => {
  carregarDados();
}, []);
```

### Padrão de atualização de estado local (SEM re-fetch)
Após POST/PUT/DELETE, atualizar o estado local diretamente. NÃO chamar carregarDados() novamente.

```javascript
// CRIAR — POST retorna o objeto criado com ID gerado pelo banco
const novoItem = await criarCategoria(categoria);
setCategorias(prev => [...prev, novoItem]);

// EDITAR — PUT retorna 204 (sem body), usar os dados locais do formulário
await atualizarCategoria(categoria.id, categoria);
setCategorias(prev =>
  prev.map(cat => cat.id === categoria.id ? categoria : cat)
);

// DELETAR — DELETE retorna 204 (sem body), usar filter
await deletarCategoria(id);
setCategorias(prev => prev.filter(cat => cat.id !== id));
```

### Regras de atualização de estado
- NUNCA mutar o state diretamente (ex: categorias.push(...))
- SEMPRE criar novo array: spread [...prev, novo], map, filter
- Usar forma de callback prev => ... para garantir valor mais atualizado
- carregarDados() só no useEffect inicial e no botão de refresh manual

### Tratamento de 404 no relacionamento 1-para-1
```javascript
// 404 = produto não tem detalhe ainda — NÃO é erro real
try {
  const detalhe = await getDetalhePorProduto(produtoId);
  setDetalhe(detalhe);
  setDetalheExiste(true);
} catch (error) {
  if (error.response?.status === 404) {
    // Situação normal: produto ainda não tem detalhe
    setDetalhe(null);
    setDetalheExiste(false);
  } else {
    // Erro real (servidor fora do ar, etc.)
    toast.error('Erro ao carregar detalhes.');
  }
}
```

### Tratamento de erro específico da API
```javascript
// Capturar mensagem de erro do backend (ex: Restrict no DELETE)
try {
  await deletarCategoria(id);
} catch (error) {
  const mensagem = error.response?.data?.mensagem || 'Erro ao deletar.';
  toast.error(mensagem);
}
```

### Notificações — React Toastify
```javascript
// Em main.jsx ou App.jsx — adicionar o container
import { ToastContainer } from 'react-toastify';
import 'react-toastify/dist/ReactToastify.css';

// No JSX:
// <ToastContainer position="top-right" autoClose={3000} />

// Nos componentes:
import { toast } from 'react-toastify';
toast.success('Categoria cadastrada com sucesso!');
toast.error('Erro ao salvar.');
```

### Padrão de Page Component (orquestrador)
```javascript
// src/components/categorias/CategoriasPage.jsx
import { useState, useEffect } from 'react';
import { toast } from 'react-toastify';
import { getCategorias, criarCategoria, atualizarCategoria, deletarCategoria } from '../../services/categoriaService';
import CategoriaTable from './CategoriaTable';
import CategoriaFormModal from './CategoriaFormModal';

function CategoriasPage() {
  const [categorias, setCategorias] = useState([]);
  const [isFormModalOpen, setIsFormModalOpen] = useState(false);
  const [categoriaEditando, setCategoriaEditando] = useState(null);
  const [isDeleteDialogOpen, setIsDeleteDialogOpen] = useState(false);
  const [categoriaDeletando, setCategoriaDeletando] = useState(null);

  const carregarDados = async () => {
    try {
      const data = await getCategorias();
      setCategorias(data);
    } catch (error) {
      toast.error('Erro ao carregar as categorias.');
      console.error('Erro ao carregar:', error);
    }
  };

  useEffect(() => {
    carregarDados();
  }, []);

  const handleNovo = () => {
    setCategoriaEditando(null);
    setIsFormModalOpen(true);
  };

  const handleEditar = (categoria) => {
    setCategoriaEditando(categoria);
    setIsFormModalOpen(true);
  };

  const handleDeletarClick = (categoria) => {
    setCategoriaDeletando(categoria);
    setIsDeleteDialogOpen(true);
  };

  const handleSalvar = async (categoria) => {
    try {
      if (categoriaEditando) {
        await atualizarCategoria(categoria.id, categoria);
        setCategorias(prev =>
          prev.map(cat => cat.id === categoria.id ? categoria : cat)
        );
        toast.success(`Categoria "${categoria.nome}" atualizada com sucesso!`);
      } else {
        const novaCategoria = await criarCategoria(categoria);
        setCategorias(prev => [...prev, novaCategoria]);
        toast.success(`Categoria "${categoria.nome}" cadastrada com sucesso!`);
      }
      setIsFormModalOpen(false);
      setCategoriaEditando(null);
    } catch (error) {
      toast.error('Erro ao salvar a categoria.');
      console.error('Erro ao salvar:', error);
    }
  };

  const handleDeletar = async () => {
    try {
      await deletarCategoria(categoriaDeletando.id);
      setCategorias(prev =>
        prev.filter(cat => cat.id !== categoriaDeletando.id)
      );
      toast.success(`Categoria "${categoriaDeletando.nome}" removida com sucesso!`);
      setIsDeleteDialogOpen(false);
      setCategoriaDeletando(null);
    } catch (error) {
      const mensagem = error.response?.data?.mensagem || 'Erro ao deletar a categoria.';
      toast.error(mensagem);
      setIsDeleteDialogOpen(false);
      console.error('Erro ao deletar:', error);
    }
  };

  // ... JSX com Tailwind CSS
}

export default CategoriasPage;
```

### Regras de componentes
- Page (ex: CategoriasPage) = orquestrador: gerencia state, chama services, passa props
- Table (ex: CategoriaTable) = apresentação: recebe dados e callbacks via props
- FormModal (ex: CategoriaFormModal) = formulário em modal: recebe item para editar (ou null para criar)
- NÃO usar bibliotecas de formulário (React Hook Form, Formik, etc.) — useState simples
- NÃO usar React Query, SWR, Redux, Zustand — useState + useEffect direto
- Estilização APENAS com Tailwind CSS utility classes
- Ícones APENAS com Lucide React

### Padrão de seletor dinâmico (relacionamento 1-para-N)
```javascript
// No FormModal de Produto — select de categorias
<select
  value={categoriaId}
  onChange={(e) => setCategoriaId(Number(e.target.value))}
  className="w-full border border-gray-300 rounded-lg px-3 py-2"
>
  <option value="">-- Selecione uma categoria --</option>
  {categorias.map((cat) => (
    <option key={cat.id} value={cat.id}>{cat.nome}</option>
  ))}
</select>
```

### Padrão do Modal de Detalhes (1-para-1) — estados
O modal deve gerenciar 3 estados:
1. Carregando → spinner
2. Sem detalhe (404) → formulário vazio + banner amarelo "ainda não possui detalhes" + botão "Cadastrar"
3. Com detalhe (200) → formulário preenchido + banner verde "já possui detalhes" + botão "Atualizar" + botão "Remover"

Salvar executa POST se não existe, PUT se já existe.

---

## PROIBIÇÕES ABSOLUTAS

### Backend
- NÃO usar Repository Pattern
- NÃO usar DTOs (Data Transfer Objects)
- NÃO usar AutoMapper
- NÃO usar MediatR
- NÃO usar FluentValidation
- NÃO usar Data Annotations nos Models ([Required], [MaxLength], etc.)
- NÃO usar DateTime.Now — SEMPRE DateTime.UtcNow
- NÃO usar FindAsync com .Include() — usar FirstOrDefaultAsync
- NÃO usar EntityState.Modified no PUT
- NÃO usar nomes genéricos em Migrations (Migration1, Update, Fix)
- NÃO usar _context.Entry(obj).State = EntityState.Modified

### Frontend
- NÃO usar TypeScript (.ts, .tsx)
- NÃO usar React Query
- NÃO usar SWR
- NÃO usar Redux
- NÃO usar Zustand
- NÃO usar bibliotecas de formulário (React Hook Form, Formik)
- NÃO mutar state diretamente (push, splice, atribuição direta)
- NÃO fazer re-fetch (chamar carregarDados()) após POST/PUT/DELETE — atualizar estado local
- NÃO usar CSS modules, styled-components ou CSS inline — APENAS Tailwind
- NÃO usar outras bibliotecas de ícones — APENAS Lucide React
- NÃO usar outras bibliotecas de toast — APENAS React Toastify

---

## CONVENÇÕES DE NOMENCLATURA

- Backend: PascalCase para classes, métodos e propriedades (C# padrão)
- Backend: nomes em português (Produto, Categoria, DetalheProduto)
- Frontend: camelCase para variáveis e funções, PascalCase para componentes
- Frontend: nomes de arquivos em PascalCase para componentes (.jsx)
- Frontend: nomes de arquivos em camelCase para services (.js)
- Mensagens de erro e sucesso SEMPRE em português
- Comentários SEMPRE em português, explicando o "por quê"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lufiabani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
